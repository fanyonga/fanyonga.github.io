---
layout: post
title: 如何在windows环境在搭建github个人博客
---
## 目录  
1. 简单认识git  
2. 为什么要用github pages
3. 创建github pages
4. 配置本地安装jekyll环境
5. jekyll的安装和配置
6. 写一个简单的博客

### 简单认识git  
git是一个开源的分布式版本控制系统，用以有效、高效地处理从很小到很大的项目版本管理。  
github可以托管各种git库的站点。  
github pages免费的静态站点，主要特点：免费托管、自带主题、支持自制页面和jekyll。  

### 为什么要用github pages  
1. 搭建简单且免费；
2. 支持静态脚本；
3. 可以绑定github账号的域名；
4. 理想的写博客环境：git+github+markdown+jekll。  

### 创建github pages
1. 首先你必须有一个github的账号，没有的可以在这里申请[github](https://github.com/);  
2. 创建一个专属的源，取名为username.github.io(username必须是你的用户名，不能更改);  
3. 利用git工具在该源的根目录下创建一个简单的index.html页面;  
4. 在浏览器中访问username.github.io，可以看到你上次提交的index页面。  

### 配置本地安装jekyll环境  
   jekyll是一种简单的、适用于博客和静态网站生成的使用Ruby语言的模板引擎，它支持Markdown、Textile等标记语言的解析，提供了模板、变量、插件等功能，最终生成一个完整的静态Web站点。  
1. 安装Ruby运行环境，点击这里下载 [RubyInstaller](http://rubyforge.org/frs/?group_id=167),然后输入以下命令更新;    
```
gem update --system
```  
2. 安装windows下模拟linux平台下的make,gcc,编译本地c/c++扩展包的工具，[Devkit](https://github.com/oneclick/rubyinstaller/downloads/)，进入解压的目录，输入安装编译工具；  
	```    
	ruby dk.rb init     
	ruby dk.rb install    
	```    
3. 配置ruby运行自动导入所需要依赖的工具bundle，命令行安装并更新；
```  
gem install bundle  
bundle update  
```  
 4. 安装用来解析Markdown标记的包Rdiscount，使用如下命令:  
```  
gem install rdiscount  
```    

### jekyll的安装和配置
1. 下载jekyll静态服务器，输入以下命令：  	
``` 
gem install jekyll  
jekyll --vesion
```  
2. 安装jeky--bootstrap及配置远程github仓库到本体：  
```  
git clone https://github.com/plusjade/jekyll-bootstrap.git USERNAME.github.com  
cd USERNAME.github.com  
git remote set-url origin git@github.com:USERNAME/USERNAME.github.com.git  
git pu origin master  
```
3. 修改样式配置,进入~assets/themes,将三个文件夹复制到bootstrap-3文件夹下   
```  
 mkdir bootstrap-3  
```  
4. 在USERNAME.github.com目录下开启服务器  
```
 jekyll serve  
```  
5. 在浏览器中访问127.0.0.1:4000可以访问index页面，进入127.0.0.1:4000/archive进入博客目录页面；  
6. 修改_config.yml文件  
 	修改标题：title : My Blog =)  
 	修改个人信息： 
```  
author :  
name : Name Lastname  
email : blah@email.test  
github : username  
twitter : username  
feedburner : feedname  
```  
修改博客命名规则：permalink: /:title  

7. 利用git工具发布到github上。  

### 写一个简单的博客    
1. jekyll目录的简单认识  
	* _posts: 用来存放你的博客的markdown文件，命名规则 yyyy-mm-dd-title.md;  
	* _layouts: 指向 _include/themes,用来存放页面显示的模板，通常用来定义不同的头部和尾部；  
	* _includes  
		- JB：存放一些常用的工具，用于列表显示、评论等；  
		- themes： 定义网站的模板；  
2. 在_posts目录下创建一个简单的markdown文件，在首部加上解释信息：  
	---  
	layout: post  
	title: 测试模板是否正常运行  
	---  
3. 运行服务器，并访问127.0.0.1:4000/title，可看到已经解释的博客页面。  
