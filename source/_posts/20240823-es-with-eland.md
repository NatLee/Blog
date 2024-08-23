---
title: 在 Docker 中使用 Elasticsearch 和 Eland：實用指南
categories:
tags:
  - Docker
  - Elasticsearch
  - Eland
  - Python
  - Data Analysis
abbrlink: 25207cf0
date: 2024-08-23 00:00:00
---

## 前言

本指南提供了在 Docker 環境中使用 Elasticsearch 和 Eland 的實用方法，為資料分析和處理大型資料集提供了強大的支援。

---

<!--more-->

## 內容

我們先從[es-in-docker-with-eland](https://github.com/NatLee/es-in-docker-with-eland)將專案clone下來：

```bash
git clone https://github.com/NatLee/es-in-docker-with-eland.git
```

### 步驟 1：設定環境

檢查 `.env` 檔案。這個檔案包含了 Elasticsearch 的設定參數，例如 ES 訪問的密碼等。根據需要調整這些設定。

### 步驟 2：建立 ES 資料儲存資料夾

將 Elasticsearch 資料掛載到外部。運行以下指令來建立資料夾並設定存取權限：

```bash
mkdir ./es-data && chmod 777 ./es-data
```

這一步驟很重要，因為如果資料夾權限不足，ES 會直接拋出權限錯誤。

### 步驟 3：啟動 Elasticsearch

使用 Docker Compose 來啟動 Elasticsearch Stack：

```bash
docker-compose up -d
```

這個命令會使用 Docker 在後台啟動 Elasticsearch 服務。

### 步驟 4：複製憑證

為了安全連接到 Elasticsearch，我們需要使用 SSL 憑證。運行以下腳本將憑證從 Docker 容器複製到本地主機：

```bash
bash dev-cp-cert.sh
```

### 步驟 5：設定 Python 環境

使用 Conda 為 Eland 的測試建立一個虛擬的 Python 環境：

```bash
conda create -n es-test python=3.11
conda activate es-test
pip install -r requirements.txt
```

這將建立一個名為 `es-test` 的 Conda 環境，並安裝所有必要的套件。

### 使用 Eland

在 `notebooks` 資料夾中，有兩個 Jupyter notebook 檔案：

1. `eland-test.ipynb`：展示了使用 Eland 進行基本的 Elasticsearch 操作。
2. `eland-over-10000-test.ipynb`：展示了如何使用 Eland 處理超過 10,000 筆資料的情況。

### 深入探索：處理大量資料

讓我們深入探討 `eland-over-10000-test.ipynb` 這個處理大量資料的實際範例。以下是一些關鍵點的程式碼範例：

#### 1. 連接到 Elasticsearch

```python
from elasticsearch import Elasticsearch
from eland import DataFrame

es = Elasticsearch(
    hosts=["https://localhost:9200"],
    basic_auth=("elastic", "pass.123"), # 密碼存在 .env 檔案中
    ca_certs="../http_ca.crt", # 本機憑證路徑 
    verify_certs=True
)

print(es.info())
```

成功的話會出現 ES 的名稱跟版本：

```
Connected to cluster named 'docker-cluster' (version: 8.15.0)
```

#### 2. 資料索引

接下來，我們將大量資料丟到 Elasticsearch 中：

> 索引的概念跟SQL中的Table很像

```python
import pandas as pd
from eland import pandas_to_eland

index_name = "random_data_index"

if not es.indices.exists(index=index_name):
    es.indices.create(index=index_name)

# 生成隨機資料的函數
def generate_random_data(n):
    """
    這個是負責產生隨機資料的函數
    """
    data = []
    for i in range(n):
        random_string = ''.join(random.choices(string.ascii_letters + string.digits, k=10))
        data.append({
            "id": i,
            "random_string": random_string,
            "random_number": random.randint(1, 1000)
        })
    return data

num_records = 20000
random_data = generate_random_data(num_records)

df = pd.DataFrame(random_data)

ed = pandas_to_eland(
    pd_df=df,
    es_client=es,
    es_dest_index=index_name,
    es_if_exists="replace",
    es_refresh=True
)
```

直接使用 Eland 就能簡單暴力的將資料插進 ES！

#### 3. 執行查詢

現在我們可以使用 Eland 執行各種查詢操作：

```python
# 從ES中讀取資料
# 底下的 ed_df 類別是 <class 'eland.dataframe.DataFrame'>
ed_df = eland.DataFrame(es_client=es, es_index_pattern=index_name)

# 基本查詢：搜尋欄位 random_number 中大於 500 的資料
filtered_df = ed_df[ed_df['random_number'] > 500]

# 複雜條件查詢：搜尋介於 300 到 700 的資料量
complex_query = ed_df[(ed_df['random_number'] > 300) & (ed_df['random_number'] < 700)]
```

看到這邊會發現，我們使用 Eland 後，免除了寫 ES 查詢指令原本的複雜敘述。

例如原本我們如果要查詢大於等於990的資料量，就得寫一堆括號：

```python
# 使用ES原生查詢大於等於 990 的資料量
original_query = ed_df.es_query(
    {
        "query": {
            "range": {
                "random_number": {
                    "gte": 990
                }
            }
        }
    }
)
```

而且返回的還不會是 DataFrame 類型的物件！

如果我們是用 Eland 的話，也可以將 Eland 的 DataFrame 直接轉成 Pandas 的 DataFrame，並且只要用 to_pandas() 即可！

```python
# 文字搜尋：將 ed_df 轉為 DataFrame 並且搜尋包含 `a` 的內容
char_query = ed_df['random_string'].to_pandas()
char_query = char_query[char_query.str.contains('a')]
```

更詳細的部分可以再參照[eland-over-10000-test.ipynb](https://github.com/NatLee/es-in-docker-with-eland/blob/main/notebooks/eland-over-10000-test.ipynb)

#### 4. 資料分析

最後，我們可以用 describe() 去看整體資料的分布：

```python
statics = ed_df['random_number'].describe()
print("\n隨機數的統計資訊:")
print(statics)
```

```
隨機數的統計資訊:
count    20000.000000
mean       498.331150
std        288.301338
min          1.000000
25%        250.165305
50%        495.636658
75%        747.183718
max       1000.000000
Name: random_number, dtype: float64
```

看到這邊，有些人可能會有疑問。因為 ES 的限制是最多一次取出 10000 筆資料，為什麼 Eland 可以超過上限？

這邊有一篇官方論壇的QA有提到 ES 限制的事 [max-limit-for-number-of-search-results](https://discuss.elastic.co/t/max-limit-for-number-of-search-results/329544)

實際上，Eland 的 code 裡面相關程式碼 [etl.py#L233](https://github.com/elastic/eland/blob/main/eland/etl.py#L233) 顯示它會一萬筆一萬筆循序把資料全部讀取出來。

也就是說，它只要呼叫一次函數就會使用多次讀取達到超過 10000 筆上限的限制！


### 結語

通過這個範例，你獲得了一個功能強大的 Elasticsearch （以及這邊沒講到的Kibana）和相見恨晚的 Eland 環境，現在可以拋開繁瑣的進行資料分析之旅了。

無論你是想要構建一個大數據搜索引擎、分析大量LOG資料，還是進行更複雜的資料處理，這個環境都能提供所需的工具和靈活性。

祝各位資料分析愉快！

---

## 參考資料

- [Elasticsearch 官方文件](https://www.elastic.co/guide/en/elasticsearch)
- [Eland 官方文件](https://eland.readthedocs.io/)
- [Eland GitHub Repo](https://github.com/elastic/eland)
---

這篇文章同步發表於 Medium ，歡迎留言討論！

[Medium 文章連結](https://medium.com/@natlee_/%E5%9C%A8-docker-%E4%B8%AD%E4%BD%BF%E7%94%A8-elasticsearch-%E5%92%8C-eland-%E5%AF%A6%E7%94%A8%E6%8C%87%E5%8D%97-ef5fedb133d8)