# hugo搭建

# brew

https://brew.sh/

## brew Linux安装文档

https://docs.brew.sh/Homebrew-on-Linux

## 前期准备

*debian*或者Ubuntu

```shell
sudo apt-get install build-essential procps curl file git
```

## 开始安装

```shell
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

## 环境变量配置

```shell
test -d ~/.linuxbrew && eval $(~/.linuxbrew/bin/brew shellenv)
test -d /home/linuxbrew/.linuxbrew && eval $(/home/linuxbrew/.linuxbrew/bin/brew shellenv)
test -r ~/.bash_profile && echo "eval \$($(brew --prefix)/bin/brew shellenv)" >>~/.bash_profile
echo "eval \$($(brew --prefix)/bin/brew shellenv)" >>~/.profile
```

## 安装第一个包工具

```shell
brew install hello
```

```shell
hello
```

```shell
which hello
```

## brew框架安装目录查询

```shell
which brew
```

## 更新hello包

```shell
brew upgrade hello
```

## 用brew工具生成静态网站

```shell
brew install hugo
```

## 可用包一览

https://formulae.brew.sh/



# hugo

## 官网

https://gohugo.io

## 安装

首先需要再Linux（Ubuntu）上安装brew包管理工具

```shell
brew install hugo
```

```shell
hugo help
```

```shell
 hugo env #环境版本号
```

## 使用步骤

### 1.建立一个网站

```shell
hugo new site myweb
```

```shell
cd myweb
```

```shell
vim config.toml
```

```
baseURL = "http://xxx.example.org/"
languageCode = "zh"
title = "小全技术站"
```

#### 其他可设置内容

```shell
hugo config
```

### 2.加入一个主题(https://themes.gohugo.io/)

```shell
cd themes #如果没有主题文件夹mkdir themes
```

#### 按照主题文档来

```shell
git clone https://github.com/google/docsy.git
cp /home/lxq/myweb/themes/docsy/config.toml /home/lxq/myweb
```

### 3.网站加入内容

```shell
vim /home/lxq/myweb/archetypes/default.md
```

`This is my demo
中文测试
`

### 4.生成三个页面

```shell
cd /myweb
```

```shell
hugo new posts/page1.md
```

```shell
hugo new posts/page2.md
```

```shell
hugo new posts/page3.md
```

#### 建立静态网站

```shell
hugo
```

```shell
ls -l public/
```

或在配置文件中设置

`theme = "book"`

#### 使用hugo http服务

```shell
hugo server -D --bind 192.168.63.130 --baseURL http://192.168.63.130/

hugo server --disableFastRender  -e production -D --bind 192.168.75.128 --baseURL http://192.168.75.128/
```

# hugo不错模板

https://themes.gohugo.io/github-style/

![](/img/hugobucuo.png)
# 将个人博客部署到远端服务器(可以使用github部署到github仓库)

`在github创建一个远端仓库`

![img](/img/2.png)


## 在myblog目录下执行

```shell
hugo --theme=m10c --baseUrl="https://liuxiaoquan.github.io/" --buildDrafts #执行其中一条命令
hugo --theme=m10c --baseUrl="https://freerun.xyz/" --buildDrafts #执行其中一条命令

hugo --theme=LoveIt --baseUrl="https://freerun.xyz/" --buildDrafts
```

## 接下来把public文件推送到github上：

- ### 切换到public文件夹下，代开命令行窗口，依次键入

  ```shell
  git init #将此public文件夹变成git本地仓库
  ```

  ```shell
  git add .  #将public文件夹下的所有文件放入缓存流中等待提交（注意后面的点）
  ```

  ```shell
  git commit -m "Hugo第一次提交" #这样就把缓存内容放进发送头，仍为待发送状态
  ```

  ```shell
  git remote add origin https://github.com/liuxiaoquan/liuxiaoquan.github.io.git #绑定了.git配置文件夹对应的远端服务器的发布了已经
  ```

  ```shell
  git push -u origin master #推送到githubu
  ```

  ```shell
  git push origin master -f #强制覆盖远程仓库的内容
  ```
  
- ### 添加CNAME文件

  ```shell
  lxq@ubuntu:~/loveltblog/public$ ll CNAME 
  -rw-rw-r-- 1 lxq lxq 12 May 12 21:33 CNAME
  
  lxq@ubuntu:~/loveltblog/public$ cat CNAME 
  freerun.xyz
  ```

  


