# Blog

![](https://api.travis-ci.com/NatLee/Blog.svg)

Just a blog.

## Usage

Go to the folder of this repo.

### MacOS

```bash
brew install node
brew install yarn
yarn global add hexo-cli
yarn install
yarn -m hexo build && yarn -m hexo server
```

### Windows

```powershell
winget install -e --id OpenJS.NodeJS
```

Close terminal and reopen it.

```
npm --global install yarn
yarn global add hexo-cli
yarn install
yarn -m hexo build && yarn -m hexo server
```

### Ubuntu

```
sudo apt update
curl -fsSL https://deb.nodesource.com/setup_16.x | sudo -E bash -
sudo apt-get install -y nodejs
sudo npm install yarn -y
yarn global add hexo-cli
yarn -m hexo build && yarn -m hexo server
```