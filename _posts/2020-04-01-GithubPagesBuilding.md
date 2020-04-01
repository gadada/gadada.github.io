---
title: "git小站搭建记录"
tags:
  - Step by step
categories:
  - Beginning
---
考研复试通知还没出来，闲来无事想尝试一下搭建自己的博客，用来记录漫长的学习之路，[Github Pages](https://github.io)平台的看起来比较干净，可自定义程度较高，那么就开始了。

Github Pages支持[Jekyll](https://jekyllrb.com/)作为后台将用户基于Markdown的博文等转换为静态页面。[Jekyll Themes](http://jekyllthemes.org/)上列出了许多Jekyll主题模板，并且基本上都可以直接下载使用。[Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/)比较干净并且功能齐全，那么就入选本小站的模板了。

另外一个比较干净的面向简历的Jekyll模板[Online CV](http://jekyllthemes.org/themes/online-cv/)。这个模板按简历的布局将页面展示出来。

综上，本小站主要是基于[Minimal Mistakes](https://mmistakes.github.io/minimal-mistakes/)以及[Online CV](http://jekyllthemes.org/themes/online-cv/)这两个Jekyll模板进行整合。

 > **由于git对国内的网络不太友好，本站所有内容会尽量减少大体积图片**


{% include toc %}

## 搭建基础框架
Minimal Mistakes的主页就是基于Github Pages运行的，因此，想要搭建起一个可用的Blog系统，只要从它的github repo拷贝或者fork一份即可，这大概是最懒的办法。另外，Minimal Mistakes的[Quick-Start Guide](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)给了更详细的过程。

### Creat一个Github Page的Repo
[Github Pages的主页](https://pages.github.com/)给出了详细的创建过程。直接参考它即可。

 1. Create a repository：登录自己的帐号，新建**gadada.github.io**
 2. Set it public: 将**gadada.github.io**库设置为Public
 3. Clone it to local
> **public = free，clone need git bash or github desktop**


### Copy files from Minimal Mistakes
 1. 下载[Minimal Mistakes](https://github.com/mmistakes/minimal-mistakes/archive/master.zip)代码包
 2. 解压到clone的库目录中
 3. 根据Minimal Mistake的[Quick Start](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/)的说明，删除多余的文件：`rm -r .editorconfig .gitattributes .github docs test CHANGELOG.md minimal-mistakes-jekyll.gemspec README.md screenshot-layouts.png screenshot.png`
 4. push到github中:
 ``` yaml
  git add --all
  git commit -m "Firs Push"
  git push -u origin master
```

## 配置Minimal Mistakes
在上面的步骤完成后，在浏览器里打开username.github.io就能看到：
![Page after first commit]({{site.url}}{{site.baseurl}}/images/posts/first-commit.jpg)

### 博客基础信息配置
Jekyll的配置文件为`_config.yml`，该文件可以对大部分的系统信息进行配置。
这个过程中，我主要对以下参数进行了设置，需要将`head.jpg`放到`/images`目录中：

``` yaml
locale                   : "zh_CN"
title                    : "Ga's Homepage"
title_separator          : "<"
name                     : "Gadada"
description              : "Hello World"
repository               : gadada/gadada.github.io

# Site Author
author:
  name             : "Gadada"
  avatar           : /images/head.jpg
  bio              : "Computer Science"
  location         : "Beijing, China"
  email            : ga_dada@qq.com
  github           : gadada

timezone: Asia/Shanghai
```

### 配置favicon
网上有许多将图片转换为favicon的[网站](http://www.favicon-generator.org/)，然后将图片转换成了favicon.ico，并将favicon.ico放置在`/images`，同时在`_include/head/custom.html`文件中添加一行：

``` html
<link rel="shortcut icon" href="{{site.baseurl}}/images/favicon.ico">
```
在完成这一步并commit后，便可以看到浏览器tab栏中的favicon变成了自己的图片了。


### 配置导航栏
Jekyll的导航栏数据主要在`_data/navigation.yml`中指定。
本小站中主要分为三部分，记录一些日常的想法(Blogs)，文章归档(Archives)，未来可能研究的一些内容(Research)，平时的一些学习记录(Notes)，，以及我的个人说明(About)。
如上所述，将`_data/navigation.yml`做了如下修改。

``` yaml
# main links
main:
  - title: "Blogs"
    url: /
  - title: "Archives"
    url: /archives/
  - title: "Research"
    url: /research/
  - title: "Notes"
    url: /notes/
  - title: "About"
    url: /cv/
```

其中，`title`是在导航栏中显示的条目名称，而`url`则是导航内容对应的地址。
但如果直接使用的话，github会告诉你说：

 > **404** File not found

产生这个问题的原因是，github pages不知道`/research/`或者`/projects/`或者`/cv/`等这些地址对应的访问内容放在哪里。
因此，需要对这些地址进行相应的配置。

#### 设置配置文件
主要是需要修改配置文件项，并建立相应的页面文件，并在页面文件中指定相应的`permalink`。

 - 修改`_config.yml`文件

``` yaml
  # Collections
    collections:
    blogs:
      output: true
      permalink: /:collection/:path/
    research:
      output: true
      permalink: /:collection/:path/
    notes:
      output: true
      permalink: /:collection/:path/

  # Defaults
    defaults:
  # _posts
  - scope:
      path: ""
      type: posts
    values:
      layout: single
      author_profile: true
      read_time: true
      comments: true
      share: true
      related: true
  # _research
  - scope:
      path: ""
      type: research
    values:
      layout: splash
      author_profile: false
      read_time: false
      comments: false
      share: true
      related: true
  # _notes
  - scope:
      path: ""
      type: notes
    values:
      layout: single
      author_profile: true
      read_time: true
      comments: true
      share: true
      related: true

```

#### 添加所需的文件
创建`_pages`目录，并在该目录下创建相应的[文件](https://github.com/gadada/gadada.github.io/tree/master/_pages)：

 - `_pages/category-archive.html`
 - `_pages/tag-archive.`
 - `_pages/notes.html`
 - `_pages/research.html`
 - `_pages/archives.html`

文件内容中的`layout`指这个页面应该用`_layout`目录中的那个页面布局，`permalink`指得是给这个页面赋予一个URL地址，该地址和`_data/navigation.yml`中的各`url`字段对应。需要注意的是，这里的permalink是源，也就是说，navigation中的url是由这些文件中的`permalink`字段决定的。


## 添加并配置Online CV
接下来将简历模板[Online CV](http://jekyllthemes.org/themes/online-cv/)整合到Minimal Mistake这个模板中。

### 将Online CV融合进Minimal Mistakes

 1. 下载Online CV的代码库。
 2. 将代码库中的`index.html`放入到github pages代码库，但是由于根目录下已经有一个同名的存在，因此把Online CV的`index.html`文件搬过来的时候将它重命名为了`cv.html`。 [^cv-navigation]
 3. 将`_layouts/default.html`文件拷贝为github pages代码库的`_layouts/cv.html`。这里相当于在Minimal Mistakes中新建立了一个简历模板布局。同时需要修改`cv.html`文件中对布局的引用，将`layout`字段的`default`改为`cv`。
 4. 将`_includes`目录拷贝为github pages代码库的`_includes/cv`目录。
 5. 由于将Online CV的文件合并到了不同的目录中，需要对Online CV代码中的include路径进行相应调整。通过`grep include _config.yml _layouts _includes -r`，便可以知道哪些文件需要进行目录调整。[^include-dir-adjust]
 6. 将Online CV的assets目录中的各个子目录分别移动倒相应的目录中
 7. 修改目录变动引起的所有文件变更。
 8. 整合`_config.yml`内容
 9. 将Online CV中的所有对配置文件的变量引用合并倒Minimal Mistakes中的对相应字段的引用。

[^cv-navigation]: `_data/navigation.yml`中的About项设定的`url`字段就是由这里将`cv.html`放置在根目录下决定的，如果将它的名字命名为`resume.html`的话，那么`_data/navigation.yml`中的相应url也需要改为`resume`。
[^include-dir-adjust]: 主要调整了`_layouts/cv.html`和`_includes/cv/sidebar.html`两个文件中的include内容。

### Online CV页面微调整

#### 添加页面导航按钮
第一步是在Online CV的页面中添加一个`Home`链接按钮，以便在CV页能够方便的跳转到博客主页中。

Online CV的页面布局是在[`_layout/cv.html`](https://github.com/huajianmao/huajianmao.github.io/blob/master/_layouts/cv.html#L12)文件中定义的，所以想要在CV页添加按钮，主要是在该页面中添加相应代码。

**Home按钮**
``` html
        <div class="back-to-home">
          <a href="/">Home</a>
        </div>
```

Online CV使用的是单独的[css文件](https://github.com/gadada/gadada.github.io/blob/master/assets/css/cv/styles.css)，

``` css
.back-to-home {
  position: absolute;
  margin-left: -65px;
  padding: 10px;
  width: 65px;
  font-size: 18px;

  background: #42A8C0;
  left: 0;
  color: #fff;
  font-weight: 800;
}

.back-to-home a {
  color: #fff;
}
```

此外，Online CV使用了[Responsive css](https://en.wikipedia.org/wiki/Responsive_web_design)，如果直接使用上面定义的`back-to-home`样式，那么在小屏幕的时候，Online CV的主体样式变化后，`Home`按钮依然会在左边，并且会被屏幕挡住。基于这个原因，还需要在`styles.css`中添加相应的[Responsive CSS](https://github.com/gadada/gadada.github.io/blob/master/assets/css/cv/styles.css#L313)，其中`max-width: 1100px`是根据`Online CV主体页面的宽度`+`按钮宽度的两倍`计算得到。

``` css
@media (max-width: 1100px) { /* 960px + 2 * (50 + 2*10)px*/
  .back-to-home {
    padding: 5px;
    width: 50px;
    font-size: 14px;
    z-index: 9999;
    margin-left: 0px;
  }
}
```

严格来说，样式不应该在css中直接修改，而应该在scss中修改，然后在重新build生成。

#### 添加PDF版本下载按钮
另外，还需要添加一个PDF版本下载按钮，处理方式和`Home`按钮基本类似。
分别在[`_layout/cv.html`](https://github.com/gadada/gadada.github.io/blob/master/_layouts/cv.html#L15)中添加相应html代码，
以及在`styles.css`中添加相应[css代码](https://github.com/gadada/gadada.github.io/blob/master/assets/css/cv/styles.css#L66)即可。

``` html
<!-- _layouts/cv.html -->
<div class="get-pdf">
  <a href="{{site.url}}{{site.baseurl}}/assets/docs/cv/test.pdf">PDF</a>
</div>
```

``` css
.get-pdf {
  position: absolute;
  margin-left: -65px;
  padding: 10px;
  width: 65px;
  font-size: 18px;

  background: #FF9900;
  left: 0;
  top: 50px;
  color: #fff;
  font-weight: 800;
}

.get-pdf a {
  color: #fff;
}

@media (max-width: 1100px) { /* 960px + 2 * (50 + 2*10)px*/
  /* ... */
  .get-pdf {
    padding: 5px;
    width: 50px;
    font-size: 14px;
    z-index: 9999;
    margin-left: 0px;
    top: 35px;
  }
  /* ... */
}
```

## 配置评论模块
Minimal Mistakes中提供了许多种[评论模块](https://mmistakes.github.io/minimal-mistakes/docs/configuration/#comments)的支持。
其中，我个人更喜欢[Staticman](https://staticman.net/)，
因为它可以保持评论和博客本身是放在一个站点上的，那么迁移等都会非常方便。
Minimal Mistakes里给出了详细的[配置Staticman的方法](https://mmistakes.github.io/minimal-mistakes/docs/configuration/#static-based-comments-via-staticman)，基本参考他的就能够将评论系统配置完毕。

### 评论模块原理
Staticman的原理其实还是比较简单的。

 1. 博客系统上线后，博客系统在每篇文章的某个地方设计一个供用户输入的comment form。
 2. 用户输入相应字段后，当点击提交的时候，博客会将用户的评论内容等post到staticman提供的一个API接口。在调用该接口时，用户的请求中会带上博客的github库的名字，分支等信息。
 3. staticman在通过接口接收到用户的评论后，通过新建一个comment文件，并将该文件push到博客所在的git库所指定的branch的相应目录中。
 4. Github Pages在接收到文件变动（即新的comment文件生成）后，它会自动重新build整个博客系统，并自动上线。
 5. 之后，其他用户就可以在页面中看到用户所做的评论内容。

除了以上的这种方式外，用户还可开启`moderation`功能。
如果开启此功能，那么staticman不是直接将comment文件直接push到用户的库当中，
而是请求一次Pull Reques的方式，等待博客主的merge。
在merge之后，才能使comment合并到gh branch中。

### Staticman配置
基于以上的原理，我们需要对Staticman进行的配置如下：

#### 开启评论，并指定staticman为provider

配置内容如下，详见[`_config.yml`](https://github.com/huajianmao/huajianmao.github.io/blob/master/_config.yml#L20)文件

``` yaml
comments:
  provider               : "staticman"

  # ...

  staticman:
  entry                  : "https://api.staticman.net/v1/entry"
  allowedFields          : ['name', 'email', 'url', 'message']
  branch                 : "master"
  commitMessage          : "New comment."
  filename               : comment-{@timestamp}
  format                 : "yml"
  moderation             : false
  path                   : "_data/comments/{options.slug}" # "/_data/comments/{options.slug}" (default)
  requiredFields         : ['name', 'email', 'message']
  transforms:
    email                : "md5"
  generatedFields:
    date:
      type               : "date"
      options:
        format           : "iso8601" # "iso8601" (default), "timestamp-seconds", "timestamp-milliseconds"
```

配置文件中，需要注意的是`path`这个值，它是以代码库的根目录为基础路径的。所有用户发表的每一条评论，都将会被生成一个文件放在这个目录中。

#### 添加staticmanapp为collaborator
如上文原理中的介绍，staticman会将用户提交的comments转换成一个文件，并且push到blog所在的库中。
因此，Github Pages对应的blog库需要给staticman相应的push代码的权限。
因此，需要在[`Settings` -> `Collaborators`](https://github.com/gadada/gadada.github.io/settings/collaboration)中添加`staticmanapp`为Collaborator。

#### 让staticman接受你的collaborator邀请
在向staticman发送邀请后，staticman需要接受该邀请，才能开始往blog库中push代码。
staticman提供了一个api来让用户自己去操作接受邀请。
该API是`https://api.staticman.net/v1/connect/{your GitHub username}/{your repository name}`。
需要在浏览器中请求一遍该地址。

如果没有将staticman添加为collaborator或者，没有访问该API让它接受邀请，那么在评论的时候，会提示有错。
此时，如果打开浏览器的调试功能的话，你会发现错的原因是对应URI找不到。

``` markdown
  {
    "code":404,
    "status":"Not Found",
    "message":{
      "message":"Not Found",
      "documentation_url":"https://developer.github.com/v3/repos/contents/"
    }
  }
```

在以上过程都完毕后，整个系统便搭建完毕了。


## 开始写博客
此后，只要在`_posts`、`_research`、`_projects`、`_tools`创建markdown或者html文件并且commit到Github Pages之后，博客中就会自动生成相应的博文了。
博文布局详见Minimal Mistakes的[layout文档](https://mmistakes.github.io/minimal-mistakes/docs/layouts/)。

