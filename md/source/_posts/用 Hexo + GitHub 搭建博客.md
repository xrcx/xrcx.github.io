---
title: 用 Hexo + GitHub 搭建博客
categories: 工具
toc: true 
theme: next
date: 2017-04-30 23:00:00
---

# 概述
![fow](用 Hexo + GitHub 搭建博客\flow.png)

## 产品介绍
1. WordPress
    - PHP语言开发的博客平台，用户可以在支持PHP和MYSQL数据库的服务器上架设博客平台
2. Jekyll 和 Octopress
    - 出于Ruby，通过命令快速生成静态网页,将Markedown转换html，通过GitHub Pages 发布
    - 只使用命令行进行部署和发布
    - 性能比较js会略慢
3. Ghost
    - 基于 Node.js，本身就具有发布功能，类似轻量级 WordPress
    - 有个简化一体化平台
    - 刚创立不久，文档还不成熟
4. Hexo
    - 基于 Node.js,生成静态Html文件，部署到各个托管平台发布
    - 效率比 Jekyll 和 Octopress 高
    
## 前提
1. Git 
    - 安装 Git
    - 创建 用户名
2. 安装 Node.js
#  GitPage 
前提：已经有github 用户名,已经设置SSH
- 创建 仓库，名字为 <username>.github.io

  ![public](用 Hexo + GitHub 搭建博客\public.png)

- 更换域名
  如何申请域名，见下面章节

  点击仓库的settings,选择GitHub Pages 的 Custom domain 输入指定的域名,保存

  ![settings](用 Hexo + GitHub 搭建博客\settings.png)

  ![github domain](用 Hexo + GitHub 搭建博客\github domain.png)

  实际效果是 仓库里面的有个CNAME文件,里面内容如下
  > myblog.org

# Hexo
## 1. 安装 Hexo
	npm install -g hexo-cli
   查看安装版本  `hexo -v`

## 2. 建站 ，folder 即为博客的文件夹
	hexo init <folder>   // 初始化
	cd <folder>          // 进入博客文件夹
	npm install          // 据博客既定的 dependencies 配置安装所有的依赖包
   博客文件夹如下
![folder](用 Hexo + GitHub 搭建博客\folder.png)

**说明**
- source：博客资源文件夹
- source/_drafts:草稿文件夹
- source/_posts:文章文件夹
- themes:存放主题的文件夹
- themes/landscape 默认主题
- _config.yml：全局配置文件

## 3. 配置博客(_config.yml)
注意冒号后面是带空格的
- 标题等基本信息
    >参数      		描述
    >title			网站标题
    >subtitle		网站副标题
    >description	网站描述
    >author		您的名字
    >language	网站使用的语言
    >timezone	网站时区。Hexo 默认使用您电脑的时区。时区列表。比如说：America/New_York, Japan, 和 UTC 

- 主题信息
    下载主题
    `git clone https://github.com/iissnan/hexo-theme-next themes/next`

    将下载的主题放到 themes 文件夹下,修改配置文件
    > theme: next
    
- `next` 文件夹下 `config.yml` 修改如下
> 选主题 `scheme: Mist`

- 部署信息
  > deploy:
  >   type: git
  >   repo: git@github.com:xrcx/xrcx.github.io.git
  >   branch: master
  
- `post_asset_folder:true`

### 菜单设置
1. 修改 `next` 主题下的 `_config.yml`的`menu`字段
其中，home代表主页，categories代表分类页，about代表关于页面，archives代表归档页，commonweal代表404页面（page not found时候显示的页面）。
2. 输入命令
```
  hexo new page about
  //结果 source\about\index.md
```

3. 如果是`categories` 菜单

  + hexo new page categories
  + categories的 index.md 页面中添加 `type: "categories"`,就会自动分类
  
## 4. 部署
- **本地部署，启用本地服务器查看(退出 ctrl+c)**
  	hexo clean
  	hexo g		// hexo generate,生成静态文件
  	hexo s 		// hexo server
  浏览器打开查看效果：http://localhost:4000/

- **git 部署**

  1) 安装 hexo-deployer-git
  	npm install hexo-deployer-git --save
  2) 部署
  	hexo clean
  	hexo g		// hexo generate,生成静态文件
  	hexo d 		//hexo deploy,部署到网站,将本地blog部署到github的仓库

  也可以自动部署
  	hexo g -d  /  hexo d -g
  查看效果，在浏览器输入：`<username>.github.io` username是你github用户名
  >注意：用cmd的`hexo d` 的时候报权限拒绝，网上有说可能是git版本问题，换成1.9的版本即可解决，我直接在`git bash`中执行`hexo d`可以部署到github上

## 6. 其他问题
### 6.1 如果首页乱码

`_config.yml` 这个保存 `utf-8` 格式保存.并且设置 `language: zh-Hans`

### 6.2 使用图片
1. 插件
- 安装插件
  npm install https://github.com/CodeFalling/hexo-asset-image --save
- 修改_config.yml
  `post_asset_folder: true`
- /source/_posts下创建和md文件同名的目录，在里面放该md需要的图片
- 插入图片
  `![](目录名/文件名.png)`
2. 放在根目录
   图片放在 source/img 下，然后在 markdown 里写 `![img](/source/img/img.png)` 。显然这样在本地的编辑器里完全不能正确识别图片的位置。

### 6.3 指令

- hexo clean,清缓存
- hexo new 'postName',新建文章
- hexo new page 'pageName', 新建页面,eg :添加 about 页面
- hexo generate , 生成静态页面到 public 目录
- hexo server , 开启预览访问端口（默认：4000，`ctrl+c` 关闭 server）
- hexo deploy , 将 .deploy 目录部署到服务器

### 6.4 上传readme.md
- 在本地写好的 README.mdown 文件复制到 source 文件夹中，再去执行hexo deploy
- 在本地写好的 README.md 文件复制到 source 文件夹中，修改Hexo目录下的_config.yml。将skip_render参数的值设置上。skip_render: README.md，再去执行hexo deploy

# 申请域名
- 去[万宝](https://wanwang.aliyun.com/?spm=5176.8142029.388261.24.rP16I5)注册域名

- 申请完后，添加解析
  ![jiexi](用 Hexo + GitHub 搭建博客\jiexi.png)

  ​

  添加如下解析
  ![jiexi2](用 Hexo + GitHub 搭建博客\jiexi2.png)

- 在浏览器输入你自己域名，可以查看结果


# 参考
1.[hexo 部署至Git遇到的坑](http://pyrinelaw.github.io/2015/09/26/hexo-d/)
2.[官方文档]( https://hexo.io/zh-cn/docs/index.html)
3.[手把手教你使用Hexo + Github Pages搭建个人独立博客](https://linghucong.js.org/2016/04/15/2016-04-15-hexo-github-pages-blog/)
4.[Next使用文档](http://theme-next.iissnan.com/getting-started.html)
5.[Next 作者博文](http://notes.iissnan.com/)