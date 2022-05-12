# 如何给不同theme的Hugo添加评论系统gitalk


[gitalk 官方demo环境](https://gitalk.github.io/)           [gitalk github地址](https://github.com/gitalk/gitalk)

## 运行效果

![img](/img/plxg.png)

```
#评论是调用github的api来完成的，不依赖第三方
https://api.github.com/repos/yourrepo/issues?
```



## 1. 创建 Github Application

如果没有application，就先创建 [Github Application](https://github.com/settings/applications/new)，这里是[官方中文帮助文档](https://docs.github.com/cn/free-pro-team@latest/developers/apps/creating-an-oauth-app)

![img](/img/qiming.png)

`Client ID，Client secrets在后面config.toml中配置时，需要使用。`

![img](/img/clientid.png)

## 方式一,直接引用Hugo的主题

如果您是直接下载使用的主题，只需要四大步骤

1. 为主题添加 `gitalk.html` 模板

   路径地址 |m10c/layouts/partials/gitalk.htm

   ```html
   {{ if .Site.Params.enableGitalk }}
   <div id="gitalk-container"></div>
   <link rel="stylesheet" href="https://unpkg.com/gitalk/dist/gitalk.css">
   <script src="https://unpkg.com/gitalk/dist/gitalk.min.js"></script>
   <script>
     const gitalk = new Gitalk({
       clientID: '{{ .Site.Params.Gitalk.clientID }}',
       clientSecret: '{{ .Site.Params.Gitalk.clientSecret }}',
       repo: '{{ .Site.Params.Gitalk.repo }}',
       owner: '{{ .Site.Params.Gitalk.owner }}',
       admin: ['{{ .Site.Params.Gitalk.owner }}'],
       id: location.pathname, // Ensure uniqueness and length less than 50
       distractionFreeMode: false // Facebook-like distraction free mode
     });
     (function() {
       if (["localhost", "127.0.0.1"].indexOf(window.location.hostname) != -1) {
         document.getElementById('gitalk-container').innerHTML = 'Gitalk comments not available by default when the website is previewed locally.';
         return;
       }
       gitalk.render('gitalk-container');
     })();
   </script>
   {{ end }}
   ```

2. 在 `single.html` 插入 `{{ partial "gitalk.html" . }}`

   路径地址 |m10c/layouts/_default/single.html

   ```javascript
   {{ partial "gitalk.html" . }}
   ```

   ![img](/img/tianjia.png)

3. config.toml

```toml
[params]
  enableGitalk = true #必须添加才生效
[params.gitalk] 
    clientID = "Client ID" # 您刚才创建Github Application 的 Client ID
    clientSecret = "Client Secret" # 您刚才创建Github Application 的 Client Secret
    repo = "xxxx.github.io" # 您的博客的github地址Repository name，例如：xxxx.github.io
    owner = "GitHub ID" # 您的GitHub ID
    admin= "GitHub ID" # 您的GitHub ID
    id= "location.pathname" # 文章页面的链接地址就是ID
    labels= "gitalk" # Github issue labels. If you used to use Gitment, you can change it
    perPage= 15 # Pagination size, with maximum 100.
    pagerDirection= "last" # Comment sorting direction, available values are 'last' and 'first'.
    createIssueManually= true # 设置为true，如果是管理员登录，会自动创建issue，如果是false，需要管理员手动添加第一个评论(issue)
    distractionFreeMode= false # Enable hot key (cmd|ctrl + enter) submit comment.
```

4. 配置 OK 后，进行编译网站 并且 push 到 你的 GitHub 上，如果 createIssueManually 设置是 true . 首次登录后 Gitalk 就能正常使用，否则需要登录后，发起一个首论，才能正常使用。



## 方式二,通过submodule引用Hugo的主题

如果你的themes是使用submodule来管理的，就是另一种方式来修改theme了。

Git Submodule 允许一个git仓库，作为另一个git仓库的子目录，并且保持父项目和子项目相互独立。

```shell
#其它操作都是一样，只是需要你先在theme仓库中，把上面的gitalk.html和single.html都做了，提交代码到你自己的theme仓库

#需要在themes下
cd /your地址/xxx.github.io.source/themes
#查看日志
git log
#然后再进入你的themes实例进行更新
git submodule update
#后续就是和本地编辑的theme修改一样的
```



## 补充：bolg评论功能选择

- Disqus（国外的，需要 VPN，kexueshangwang）

- Hypercomments（不支持 Markdown）

- valine

  ```
  Valine 诞生于2017年8月7日，是一款基于LeanCloud的快速、简洁且高效的无后端评论系统。 理论上支持但不限于静态博客，目前已有Hexo、Jekyll、Typecho、Hugo、Ghost 等博客程序在使用Valine。
  
  特性
  快速
  安全
  Emoji 😉
  无后端实现
  MarkDown 全语法支持
  轻量易用
  文章阅读量统计 v1.2.0+
  ```

Github大礼包：[gitment](https://link.zhihu.com/?target=https%3A//github.com/iissnan/hexo-theme-next/issues/1604)， [gitalk](https://link.zhihu.com/?target=https%3A//github.com/iissnan/hexo-theme-next/pull/2037)**（推荐），**[gitter](https://link.zhihu.com/?target=https%3A//www.vincentqin.tech/2016/08/09/build-a-website-using-hexo/%23增加Gitter)**（推荐）;** 三个都支持**Markdown**

```
原则上来说比较靠谱的是gitment（依托于github issue，能够自己管理，而且被墙的概率小），不过兼容性不太好（需要chrome内核才行）。

Gitalk 是一个基于 GitHub Issue 和 Preact 开发的评论插件。

特性

使用 GitHub 登录
支持多语言 [en, zh-CN, zh-TW, es-ES, fr, ru, de, pl, ko]
支持个人或组织
无干扰模式（设置 distractionFreeMode 为 true 开启）
快捷键提交评论 （cmd|ctrl + enter）
```



## 碰到的问题

### 404 Error: Not Found

![img](/img/notfound.png)

`解决办法：未能正确找到仓库 repo，检查一下你的仓库是否配置正确。`

```toml
#params.gitalk
repo = "rep名字" # 你的github托管仓库，点击settings,就能看到Repository name
#或者参考界面，按F12,查看issues的get请求地址
https://api.github.com/repos/owner名字/repo名称/issues?labels=Gitalk,...
```



### Error: Validation Failed.

是因为 GitHub 对 Issue 的 label （文章页面的链接地址）存在限制，不能超过 50 个字符，否则会导致加载 Gitalk 插件失败。

1. Toml解决办法一：截取字符串 for toml

   路径地址 |m10c/layouts/partials/gitalk.htm

```html
// 截取字符串 
var title = location.pathname.substr(0, 50); 
var gitalk = new Gitalk({     
clientID: 'xxxx',     
clientSecret: 'xxxx',     
repo: 'xxxx',     
owner: 'xxxx',     
admin: ['xxxx'],     
id: title,     
distractionFreeMode: false }) 
```

![image-20210429154149870](/img/image-20210429154149870.png)

2. Html解决办法：中文标题被转码长度变长

```html
var gitalk = new Gitalk({
    clientID: '{{ theme.gitalk.ClientID }}',
    clientSecret: '{{ theme.gitalk.ClientSecret }}',
    repo: '{{ theme.gitalk.repo }}',
    owner: '{{ theme.gitalk.githubID }}',
    admin: ['{{ theme.gitalk.adminUser }}'],
    id: decodeURI(location.pathname),
    distractionFreeMode: '{{ theme.gitalk.distractionFreeMode }}'
})
gitalk.render('gitalk-container')
```


