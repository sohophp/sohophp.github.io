---
title: hexo图片插件
date: 2021-01-24 12:18:32
tags: 
 - Hexo
 - Image 
 - Javascript
categories:
- Hexo
---



1. 卸载旧的图片插件（如果有） 例如 `hexo-asset-image`或者`hexo-simple-image` ，这两个图片网址都有错误 
2. 修改_config.yml 的 `post_asset_folder: true`
3. `hexo new post "test"` 试一个会自动生成source/_posts/test文件夹用来存放图片文件
4. `npm install hexo-image-link --save` 安装插件
5. hexo server --debug 启动server

![logo](hexo图片插件/logo20210123204140.png)

