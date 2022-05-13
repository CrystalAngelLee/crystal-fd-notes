Npm-yarn 包管理工具：yrm

```
1. 增加源 yrm add <源名称> 源地址 yrm add testResource  https://testResource.com
2. 删除源 yrm del <registry> yrm del testResource
```



node版本管理工具：nvm

```
1、nvm安装
brew install nvm
2、编辑配置文件，vim ~/.bash_profile，文件中写入如下内容：
export NVM_DIR=~/.nvm
source $(brew --prefix nvm)/nvm.sh
export NVM_NODEJS_ORG_MIRROR=http://npm.taobao.org/mirrors/node
export NVM_IOJS_ORG_MIRROR=http://npm.taobao.org/mirrors/iojs
3、重启配置文件 source ~/.bash_profile
4、安装node指定版本
nvm ls-remote // 查看所有的node可用版本
nvm list // 查看已安装node版本
nvm install 版本号 // 下载指定node版本，如nvm install v11.14.0
nvm use 版本号 // 使用指定版本
nvm alias default // 设置默认版本，每次启动终端都使用该版本
```

