---
title: Hexo 配置说明
date: 2016-05-19 12:16:49
tags: [Hexo]

---

# Hexo的配置
### 目录和文件

1. scaffolds：模板文件夹，新建文章时，Hexo 会根据 scaffold 来建立文件。Hexo 有三种默认布局：post、page 和 draft，它们分别对应不同的路径。新建文件的默认布局是post，可以在配置文件中更改布局。用draft布局生成的文件会被保存到 source/_drafts 文件夹。 
2. source：资源文件夹是存放用户资源的地方。 
3. source/_post：文件箱。（低版本的hexo还会存在一个_draft，这是草稿箱）除 posts 文件夹之外，开头命名为 (下划线)的文件/ 文件夹和隐藏的文件将会被忽略。Markdown 和 HTML 文件会被解析并放到 public 文件夹，而其他文件会被拷贝过去 
4. themes：主题 文件夹。Hexo 会根据主题来生成静态页面。 
5. themes/landscape：默认的皮肤文件夹 
6. _config.yml：全局的配置文件，每次更改要重启服务。
<br/><br/>低版本的Hexo还会生成scripts文件夹，里面用于保存扩展Hexo的脚本文件。<br/>

### 全局配置
可以在_config.yml 中修改：<br/>
```{bash}

	# Hexo Configuration
	## Docs: http://hexo.io/docs/configuration.html
	## Source: https://github.com/hexojs/hexo/
	# Site 站点配置
	title: Hexo-demo  #网站标题
	subtitle: hexo is simple and easy to study   #网站副标题
	description: this is hexo-demo #网栈描述
	author: pomy #你的名字
	language:  zh-CN #网站使用的语言
	timezone: Asia/Shanghai  #网站时区
	
	# URL #可以不用配置
	## If your site is put in a subdirectory, set url as 'http://yoursite.com/child' and root as '/child/'
	url: http://yoursite.com #网址，搜索时会在搜索引擎中显示
	root: /  #网站根目录
	permalink: :year/:month/:day/:title/ #永久链接格式
	permalink_defaults: #永久链接中各部分的默认值
	
	# Directory 目录配置
	source_dir: source #资源文件夹，这个文件夹用来存放内容
	public_dir: public #公共文件夹，这个文件夹用于存放生成的站点文件
	tag_dir: tags #标签文件夹
	archive_dir: archives #归档文件夹
	category_dir: categories #分类文件夹
	code_dir: downloads/code #Include code 文件夹
	i18n_dir: :lang   #国际化文件夹
	skip_render:      #跳过指定文件的渲染，您可使用 glob 来配置路径
	
	# Writing 写作配置
	new_post_name: :title.md # 新文章的文件名称
	default_layout: post   #默认布局
	titlecase: false # Transform title into titlecase
	external_link: true # Open external links in new tab
	filename_case: 0 #把文件名称转换为 (1) 小写或 (2) 大写
	render_drafts: false #显示草稿
	post_asset_folder: false #是否启动资源文件夹
	relative_link: false #把链接改为与根目录的相对位址
	future: true  
	highlight:  
	#代码块的设置
	enable: true
	line_number: true
	auto_detect: true
	tab_replace:
	
	# Category & Tag 分类 & 标签
	default_category: uncategorized #默认分类
	category_map:   #分类别名   
	tag_map:        #标签别名
	
	# Date / Time format  时间和日期
	## Hexo uses Moment.js to parse and display date
	## You can customize the date format as defined in
	## http://momentjs.com/docs/#/displaying/format/
	date_format: YYYY-MM-DD
	time_format: HH:mm:ss
	
	# Pagination 分页
	## Set per_page to 0 to disable pagination
	per_page: 10 #每页显示的文章量 (0 = 关闭分页功能)
	pagination_dir: page #分页目录
	
	# Extensions 扩展
	## Plugins: http://hexo.io/plugins/ 插件
	## Themes: http://hexo.io/themes/  主题
	theme: landscape #当前主题名称
	
	# Deployment #部署到github
	## Docs: http://hexo.io/docs/deployment.html
	deploy:
	  type:
```

一般主题下有一个 languages 文件夹，用于对应 language 配置项。 比如在 ejs 中有：<br/>

	"<%= __('tags') %>：
	
language 的配置项是 zh-CN，则会在languages 文件夹下找到 zh-CN.yml 文件中对应的项来解释。<br/>
修改全局配置时，注意缩进，同时注意冒号后面要有一个空格。<br/>


### 主题配置
主题的配置文件在`/themes/主题文件夹/_config.yml`，一般包括导航配置(menu)，内容配置(content)，评论插件，图片效果(fancybox)和边栏(sidebar)。<br/>

Hexo提高了大量的主题，可以在全局配置文件中更改主题：<br/>

	# Extensions 扩展
	## Plugins: http://hexo.io/plugins/ 插件
	## Themes: http://hexo.io/themes/  主题
	theme: 你的主题名称


#### 基本使用

写文章 
通过new命令新建一篇文章：
	
	$ hexo new [layout] <title>
	//same as
	hexo n
	
其中layout是可选参数，默认值为post。

如果没有设置 layout 的话，默认使用 _config.yml 中的 default_layout 参数代替。如果标题包含空格的话，需用引号括起来。
Hexo提供的layout在scaffolds目录下，也可以在此目录下自建layout文件。新建的文件则会保存到source/_post目录下。


然后启动服务器，便能看到刚刚发表的文章

发表的文章会全部显示，如果文章很长，就只要显示文章的摘要就行了。在需要显示摘要的地方添加如下代码即可：
	
	以上是摘要
	<!--more-->
	以下是余下全文


原文：http://www.ido321.com/1650.html