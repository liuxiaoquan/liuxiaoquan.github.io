# GitHub搜索技巧整理

# GitHub搜索技巧整理

github作为全球最大的开源软件项目托管平台，相信很多程序员都在使用，不仅仅是因为它可以免费的作为我们公有或者私有的代码仓库，更因为github上面有大量的开源学习项目或资源，秉着开源自由的理念，吸引了大量的个人或者企业开发者。

那么面对如此海量的代码仓库，如何才能在众多的资源中搜索出更优秀，更符合自己需求的项目呢？

比如我想搜索一个springboot项目，你是否就直接输入springboot关键字直接搜索，但是搜索出了118,085个结果，当然了，你还可以做一些简单的排序，比如通过stars、forks的数量。

![](/img/github1111.png)

搜索中如果你发现github网页加载很慢，或者图片打不开，请打开hosts文件(C:\Windows\System32\drivers\etc)，加上以下内容：

```
192.30.253.113     github.com
151.101.113.194    github.global.ssl.fastly.net
151.101.184.133    assets-cdn.github.com
151.101.184.133    raw.githubusercontent.com
151.101.184.133    gist.githubusercontent.com
151.101.184.133    cloud.githubusercontent.com
151.101.184.133    camo.githubusercontent.com
151.101.184.133    avatars0.githubusercontent.com
151.101.184.133    avatars1.githubusercontent.com
151.101.184.133    avatars2.githubusercontent.com
151.101.184.133    avatars3.githubusercontent.com
151.101.184.133    avatars4.githubusercontent.com
151.101.184.133    avatars5.githubusercontent.com
151.101.184.133    avatars6.githubusercontent.com
151.101.184.133    avatars7.githubusercontent.com
151.101.184.133    avatars8.githubusercontent.com
```

但这样搜索出来的结果真的精确吗？接下来，我们来演示一下几个我们常用的github搜索技巧，让搜索出来的结果更加精确、符合要求！

首先我们来看一张思维导图：

![](/img/github12222.png)

上面的搜索技巧，我分为了2类，一类常用和更多，常用的部分应该是我们日常使用频率最高的，需要我们记住。

#### 1、in

关键字 in 是用来限定搜索的范围，可以指定是在名称、描述、readme文档中搜索关键字

- in:name：指定搜索范围是仓库名称
- in:description：指定搜索范围是摘要中
- in:readme：指定搜索范围是readme文档中

比如，指定项目仓库名称springboot、mybatis、demo三个关键字，那么搜索如下：

```
in:name springboot mybatis demo
```

结果如下：

![](/img/github1333.png)

这样搜索出来的项目就是一个简单的demo整合项目，而不是综合项目。 你还可以这样搜：

```
in:description springboot mybatis 整合
```

![](/img/github1444.png)

#### 2、stars 、forks

通常我们判断一个项目好不好，可以通过项目的stars和fork数量来判断，当然了，这也不是绝对的，github中还隐藏这很多不为人所知的优秀项目，等着你挖掘哈。

方式如下：

- **stars:>** ：筛选stars数量大于某个值的仓库
- **stars:start..end** ：筛选stars数量在start和end区间的仓库
- **fork:>**
- **fork:start..end**

所以，通过stars 、forks关键字，我们可以通过stars 、forks数量来过滤一部分。比如，我要筛选搜索结果中，stars数量大于50的项目。那么如下：

```
in:name springboot mybatis demo stars:>50
复制代码
```

筛选之后的结果只有2个符合要求：

![](/img/github1555.png)

#### 3、language

这个简单，指定项目的编写语言，如java、python、php等。比如我们搜索`单点登录`，如果我们直接搜索`in:description 单点登录`，那么出现的结果会包含各种语言的实现项目，但是如果你加上了java语言的限定条件之后，搜索出来的结果就只有java的。

```
in:description 单点登录 language:java
```

![](/img/github1666.png)

#### 4、created、pushed

创建日期、更新日期。项目久不维护了，或者项目已经创建很久了，那么项目的技术有时候就已经过时了，比如以前Springboot的1.5版本的创建项目就不是很适合现在了，现在我们学习的话直接上手2.0版本以上的比较好，所以找新项目，还得跟紧技术的迭代速度。

```
in:description 单点登录 language:java pushed:>2019-12-01
```

![](/img/github1777.png)

通常来说，stars数量多，维护频繁的项目都是比较优秀的开源项目。

#### 其他

还可以根据协议`license:`；或者项目作者`user:`；或者仓库的大小`size:>=`；被关注人数`followers:`，只不过大家就用得比较少。

#### 高级搜索

除了使用这种特定的限定词来筛选之外，其实github还给我们提供了一种筛选的搜索链接。

```
https://github.com/search/advanced
复制代码
```

其实就是界面化的搜索条件筛选框，想不起搜索关键词或者单词的时候可以收藏这个高级搜索界面哈。

![](/img/github1888.png)

下面是常用搜索总结

![](/img/github搜索技巧.png)


