---
title: 以Django爲後端開發集成式管理的Docker開發環境
categories: dev
tags:
  - docker
  - django
  - python
  - traefik
abbrlink: c83a1208
date: 2024-01-04 00:00:00
---

![demo](https://github.com/NatLee/dev-dock-manager/raw/main/doc/workflow.gif)

## 前言

---

從使用Docker快速建立GUI圖形化的深度學習開發環境這篇文章之後，我們自己土砲幹了一個[Portainer](https://www.portainer.io/)，可以用來管理Docker，但是還是沒辦法滿足我們大量建立並管理開發環境的需求。

<!--more-->

## 內容

---

> 這篇文章完成的專案在[Dev Dock Manager](https://github.com/NatLee/dev-dock-manager)


在上一篇文章[以 Django 建立 Docker GUI 控制界面](https://natlee.github.io/Blog/posts/41a1fec1/)中，我們使用Django建立了一個Docker GUI控制界面

但是還是沒有辦法滿足我們的需求，因爲我們需要一個可以管理多個開發環境的界面

所以上一次，我們只完成了第一階段的目標

這次，我們把完整的流程圖畫出來：

![workflow](https://i.imgur.com/5zvekw0.png)

從圖上可以看到第二階段我們需要：

- GUI Docker Image
- Development Container

`GUI Docker Image`的部分，我們已經在之前的文章有介紹了

詳情可參考[使用 Docker 快速建立 GUI 圖形化的深度學習開發環境](https://natlee.github.io/Blog/posts/c7016718/)，其專案的位置在[Dev Dock](https://github.com/NatLee/dev-dock)

所以，我們要完成的最後一個部分是『開發容器』的管理

### Stage II: Container Management

這個階段，我們分爲兩個部分，其中一個是我們要新增的後端API

另外一個則是爲了要能夠對container使用網頁進行遠端操作，我們需要找到proxy的工具並且將其整合進來

#### Proxy工具 Traefik

在這個專案中，我使用了[traefik](https://traefik.io/)來做為管理各個container proxy的工具

> 原始碼可以參照[docker-compose.yml](https://github.com/NatLee/dev-dock-manager/blob/main/docker-compose.yml)

```yml
version: "3.3"

x-common-networks: &common-networks
  # 爲了安全（可能也沒多安全XD）
  # 我們得把開發用的container都綁在同一個docker network上
  networks:
    - d-gui-network

x-common-extra-hosts: &extra-hosts
  # 使用 host.docker.internal 來連接到 host 的 gateway
  # 這樣我們就可以在 container 中使用 host 的網路
  # （剛剛才說爲了安全，現在這個又是不安全的操作XDDD）
  extra_hosts:
    - "host.docker.internal:host-gateway"

services:
  backend:
    <<: [*common-networks, *extra-hosts]
    # ...<省略一些code>...
    environment:
      # 這邊把專案的 network 寫進環境變數
      - DOCKER_NETWORK=d-gui-network
    # ...<省略一些code>...
    labels:
      # 開啓 traefik
      - "traefik.enable=true"
      # ====== 這邊去定義 traefik 對這個 container 的路由 ======
      # Define router for /
      # 這邊是定義訪問 traefik / 的路由
      - "traefik.http.routers.backend-root.rule=Path(`/`)"
      - "traefik.http.routers.backend-root.service=backend-service"
      # Define router for /login
      - "traefik.http.routers.backend-login.rule=PathPrefix(`/login`)"
      - "traefik.http.routers.backend-login.service=backend-service"
      # Define router for /dashboard
      - "traefik.http.routers.backend-dashboard.rule=PathPrefix(`/dashboard`)"
      - "traefik.http.routers.backend-dashboard.service=backend-service"
      # Define router for /api
      - "traefik.http.routers.backend-api.rule=PathPrefix(`/api`)"
      - "traefik.http.routers.backend-api.service=backend-service"
      # Define router for websocket
      - "traefik.http.routers.backend-websocket.rule=PathPrefix(`/ws`)"
      - "traefik.http.routers.backend-websocket.service=backend-service"
      # Define the service
      # Django啓動的後端在 container 內部的 port 是 8000
      - "traefik.http.services.backend-service.loadbalancer.server.port=8000"
      # ========================================================

  # ...<省略一些code>...

  traefik:
    <<: [*common-networks]
    image: traefik:v3.0
    container_name: d-gui-proxy
    command:
      # 這邊是 traefik 啓動時的設定
      - --providers.docker
      - --providers.docker.exposedByDefault=false
      - --entrypoints.web.address=:80 # web entrypoint
      - --api.dashboard=true
      - --api.insecure
      - --serverstransport.insecureskipverify=true
    ports:
      - "8000:80" # Traefik 的 web entrypoint
      - "8080:8080" # Traefik Dashboard
    volumes:
      # 這邊是讓 traefik 可以讀取 docker.sock
      # 這樣 traefik 才能夠知道有哪些 container 需要被 proxy
      - /var/run/docker.sock:/var/run/docker.sock

networks:
  d-gui-network:
    external: true
```

我們在`docker-compose.yml`把設定都定義完成之後，我們就可以使用`docker-compose up -d`來啓動這個服務

另外，補充一下上篇文章沒有提到的部分

因爲我們有使用 task queue ，所以需要額外啓動 Django-RQ

根據Docker官方建議做法，我是使用 supervisor 來同時管理的兩個服務（Django、Django-RQ）

> 詳情可見[supervisord.conf](https://github.com/NatLee/dev-dock-manager/blob/main/supervisord.conf)

關於Django後端在supervisord上的設定如下：

```conf
[program:django]
command=python manage.py runserver 0.0.0.0:8000 # 啓動 Django 的方式
stdout_logfile=/dev/stdout # 把 stdout 導到容器內的 /dev/stdout
stdout_logfile_maxbytes=0 # 不限制 stdout 的大小
stderr_logfile=/dev/stderr # 把 stderr 導到容器內的 /dev/stderr
stderr_logfile_maxbytes=0 # 不限制 stderr 的大小
autostart=true # 自動啓動
autorestart=true # 自動重啓
```

#### 新增的額外後端API

- 更新建立Container的API

在上篇文章中，我們建立container的API不需要再去做額外的設定

但因爲我們要建立開發用的container，所以我們需要做帳號、密碼以及環境的設定

原本的`run_image_task`程式碼改成下面這樣：

> 原始碼可以參照[task.py](https://github.com/NatLee/dev-dock-manager/blob/main/src/xterm/task.py)

```python
@job
def run_image_task(
  image_name, # 使用的image名稱
  ports, # 開啓的port
  volumes, # 掛載的volume
  environment, # 環境變數
  name, # 設定的container名稱
  privileged=False, # 是否使用privileged
  nvdocker=False # 是否使用NVIDIA GPU
):
    client = docker.from_env()
    device_requests = []
    network = client.networks.get(DOCKER_NETWORK)

    if nvdocker:
        # Define device requests for NVIDIA GPUs
        device_requests += [
            docker.types.DeviceRequest(
                count=-1,  # -1 specifies all available GPUs
                capabilities=[['gpu']],  # This is the equivalent of `--gpus all`
                driver='nvidia'
            )
        ]

    # 這邊要新增 traefik 的 label
    # 讓 traefik 可以抓到並將這個 container 的 port
    traefik_labels = {
        # 對這個 container 啓用 traefik
        "traefik.enable": "true",
        # 標記 novnc 路由位置
        f"traefik.http.routers.d-gui-{name}.rule": f"PathPrefix(`/novnc/{name}/`)",
        # 標記這個 container 的 novnc 的port
        f"traefik.http.services.d-gui-{name}.loadbalancer.server.port": "6901",
        # 設定一個 middlewares 做stripprefix 來把 novnc 的路由前綴去掉
        f"traefik.http.middlewares.d-gui-{name}-strip-prefix.stripprefix.prefixes": f"/novnc/{name}/",
        # 把路由前綴去掉的 middleware 加到這個 container 上
        f"traefik.http.routers.d-gui-{name}.middlewares": f'd-gui-{name}-strip-prefix',
        # 把這個 container 標記到同一個 treafik 的 network 上
        "traefik.docker.network": DOCKER_NETWORK,
    }

    # 使用 client 的 instance 建立 container
    container = client.containers.run(
        image_name,
        stdin_open=True,
        detach=True,
        tty=True,
        ports=ports,
        volumes=volumes,
        environment=environment,
        name=name,
        privileged=privileged,
        device_requests=device_requests,
        network=network.id,  # Attach the container to the network
        labels=traefik_labels
    )

    # ...<省略一些code>...
    
    return msg
```

其中有一個地方寫到`DOCKER_NETWORK`，這個是爲了讓所有的container都可以在同一個網路上，這樣我們才能夠使用traefik來管理

```python
DOCKER_NETWORK = os.environ.get("DOCKER_NETWORK", "d-gui-network")
```

- 偵測是否支援NVIDIA GPU

偵測機器上面是否能使用GPU這件事，我們可以使用`nvidia-smi`

但是我們要怎麼在 Django 後端的 container 內部去偵測呢？答案是做不到！

不僅僅是因爲我們的 container 內沒有 NVIDIA GPU 的 driver，而且我們安裝也會很麻煩

可是這樣就沒辦法偵測了嗎？當然不是！

我們可以使用`nvidia/cuda`這個 image 來偵測

如果使用`nvidia/cuda`這個 image 啓動 container 沒有問題，那就代表這個機器是支援 NVIDIA GPU 的

爲了確保使用者先pull下來這個 image，我們可以在`docker-compose.yml`中定義一個`nvidia-cuda`的 container

> 定義在[nvidia-cuda](https://github.com/NatLee/dev-dock-manager/blob/main/docker-compose.yml#L52)

```yaml
nvidia-cuda:
  # for testing with GPU support
  image: nvidia/cuda:11.0.3-base-ubuntu20.04
  container_name: d-gui-cuda
  entrypoint: ["echo", "CUDA image ready"]
```

程式內部的偵測方式就可以使用Docker SDK來做了

> 原始碼可以參照[can_use_nvidia_docker.py](https://github.com/NatLee/dev-dock-manager/blob/main/src/xterm/utils/can_use_nvidia_docker.py)

```python
import docker
from docker.errors import APIError, ContainerError

def can_use_nvidia_docker() -> bool:
    client = docker.from_env()
    test_image = 'nvidia/cuda:11.0.3-base-ubuntu20.04' # NVIDIA CUDA image
    test_command = 'nvidia-smi' # 測試用的指令

    try:
        # Run a container that utilizes GPU
        container = client.containers.run(
            test_image,
            command=test_command,
            runtime='nvidia',  # Specify the NVIDIA runtime
            detach=True,
            auto_remove=True,  # Remove container after execution
        )

        # Fetch logs to check if nvidia-smi command was successful
        #logs = container.logs()
        #print(logs.decode('utf-8'))  # Optional: Print output for verification

        # If the command was successful, NVIDIA Docker is available
        return True
    except (APIError, ContainerError) as e:
        # If there was an error, log it and return False
        print(f"Error checking NVIDIA Docker availability: {e}")
        return False
    finally:
        # Cleanup: Stop container if it's still running
        try:
            container.stop()
        except Exception:
            pass  # If container doesn't exist or is already removed, ignore
```

也就是說，我們直接Run一個`nvidia/cuda`的container，然後執行`nvidia-smi`指令

如果成功就代表這個機器是支援NVIDIA GPU的，反之則否

對於前端來說，如果機器本身不支援NVIDIA GPU，我們就可以提示使用者

![detect-gpu](https://i.imgur.com/QdfaRng.png)

- 偵測現在能使用哪些Port

雖然我們的VNC、noVNC可以靠proxy來解決訪問的問題

但是SSH就不行了，因爲SSH是直接連接到container的port

所以我們需要偵測現在能使用哪些port，然後把這些port顯示給使用者選擇或是自動選擇

也就是說，爲了達成這個功能，我們需要做到以下三件事：

1. 判斷某個port是否能使用
2. 偵測現在有哪些port被container使用（！？）
3. 偵測現在有哪些port能使用

---

第一件事，我們可以使用`socket`來做：

> 原始碼可以參照[check_port_in_use.py](https://github.com/NatLee/dev-dock-manager/blob/main/src/xterm/utils/check_port_in_use.py)

```python
def check_port_in_use(port) -> bool:
    # check if port is in use in the host
    with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:
        # If connect_ex returns 0, the port is in use
        return s.connect_ex(('host.docker.internal', port)) == 0
```

上面要注意的是，我們使用`host.docker.internal`來連接到 host 的 gateway

因爲port必須對應到主裝置，所以才需要連接到 host

---

第二件事比較弔詭

因爲理論上來說，我們判斷port有沒有被使用，就可以知道哪些port可以使用了吧？

但是當container是stop的狀態下，port會被釋出，所以我們無法知道哪些port是被container使用的

所以我們需要做的是，使用Docker SDK去判斷哪些port是被container使用的（包含stop的container）

> 原始碼可以參照[is_port_used_by_container.py](https://github.com/NatLee/dev-dock-manager/blob/main/src/xterm/utils/is_port_used_by_container.py)

```python
def is_port_used_by_container(port:int) -> bool:
    client = docker.from_env()
    for container in client.containers.list(all=True):
        port_bindings:dict = client.api.inspect_container(container.id)['HostConfig'].get('PortBindings', {}) # 關鍵是這邊直接inspect去看container設定的port
        ports = parse_ports(port_bindings)
        if str(port) in ports.values():
            return True
    return False
```

---

最後一件事！

我們完成前面兩件事之後，就可以從最小的port開始往上找，找到沒被使用的port

這邊要注意的是，找到所有的port非常耗時（畢竟是使用socket去測試）

所以我們要定義數量，找到足夠的port就可以停止了

> 原始碼可以參照[find_multiple_free_ports.py](https://github.com/NatLee/dev-dock-manager/blob/main/src/xterm/utils/find_multiple_free_ports.py)

```python
def find_multiple_free_ports(count):
    # find multiple free ports in the host

    base_port = 1024  # Start scanning from port 1024
    free_ports = []

    while len(free_ports) < count and base_port < 65535:
        if not check_port_in_use(base_port) and not is_port_used_by_container(base_port):
            # find port is not in use in the host
            free_ports.append(base_port)
        base_port += 1

    if len(free_ports) < count:
        raise Exception("Not enough free ports available.")
    
    return free_ports
```

再寫一個API包起來，就可以在前端使用了

從網站就可以看到每次建立container會先去問現在有哪些port可以使用

![check-ports](https://i.imgur.com/FyJ0znu.png)


## 結語

---

這次的專案，我們最後完成了一個多個開發環境的網頁管理系統

內容包含可以操作的GUI容器、可網頁控制的docker界面以及整個管理系統

也就是一開始的目標：

**『以Django爲後端開發集成式管理的Docker開發環境』**

如同一開始這篇文章的cover所示，我們可以在這個界面上面建立獨立的開發環境

甚至可以使用網頁進行遠端GUI操作以及web terminal

還可以綁定NVIDIA GPU，讓開發環境可以使用GPU進行各種深度學習AI相關的開發及測試

---

說實在的，其實一開始沒想過要做完整套管理系統

但後來要測試的專案跟環境越來越多又越來越亂，也沒有一個好的管理方式

而且待過的公司都是在同一台機器上面開發（大家甚至都是用root帳號，動不動就破壞環境）

還是我沒有去過有好開發方式的公司？

總之，我設想一個系統能夠做到管理不同虛擬環境的事情，於是就催生了這個專案

雖然這個專案的功能還有很多可以改進的地方，但畢竟時間不多，而且也只是side project

之前能重用的部分我也盡量重用了XDD

---

最後，我們來看一下這個專案的部分畫面

- 登入畫面
  ![login-page](https://i.imgur.com/dSBhK00.png)

- 首頁看到的container list
  ![container-list](https://i.imgur.com/grxAR2O.png)

- NoVNC進入container
  ![demo](https://i.imgur.com/F2AHauZ.png)

- Web Terminal
  ![web-terminal](https://i.imgur.com/O0lLm0R.png)


歡迎大家提出建議或是發PR，我都會處理的 :D

---

這篇文章同步發表於 Medium ，歡迎留言討論！

[Medium 文章連結](https://medium.com/@natlee_/%E4%BB%A5django%E7%88%B2%E5%BE%8C%E7%AB%AF%E9%96%8B%E7%99%BC%E9%9B%86%E6%88%90%E5%BC%8F%E7%AE%A1%E7%90%86%E7%9A%84docker%E9%96%8B%E7%99%BC%E7%92%B0%E5%A2%83-fe790fcfb2db)


