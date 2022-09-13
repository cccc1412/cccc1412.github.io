## 部署
* 安装nvm
nvm是node版本管理工具，来源：https://github.com/nvm-sh/nvm
```shell
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.38.0/install.sh | zsh
source ~/.zshrc
nvm install 13.14.0
```

* 先从github从hexo分支把源克隆下来
`git clone git@github.com:cccc1412/cccc1412.github.io.git`


* 在blog文件夹下
npm install
npm install -g hexo-cli

然后hexo g 即可完成迁移

## 环境问题
若出现`The "mode" argument must be integer. Received an instance of Object`, 检查node版本是否过高，目前确定版本13.14.0可用


