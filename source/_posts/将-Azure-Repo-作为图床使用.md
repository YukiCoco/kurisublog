---
title: 将 Azure Repo 作为图床使用
date: 2023-03-16 16:12:14
tags: ["工具"]
---
![Azure](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2023/2/6/63587bb9f219.png&$format=octetStream&api-version=5.0)
Azure Repo 是微软提供的免费代码托管仓库，得益于优秀的线路，国内访问体验非常不错，目前博主的图片都托管在这里。将它来作为图床（文件）是一个不错的选择，希望未来不会因为被滥用而拔线吧。
<!-- more -->
## 线路
Azure Repo 的服务器 `dev.azure.com` 是 [Anycast](https://www.cloudflare.com/zh-cn/learning/cdn/glossary/anycast-network/) 线路。  
 ![Anycast CDN 图示](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2023/2/6/c7b0b748924c.png&$format=octetStream&api-version=5.0)
### Traceroute
![中国移动](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2023/2/6/d38e83e5b9fc.png&$format=octetStream&api-version=5.0)
![中国电信](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2023/2/6/446cdf9b0220.png&$format=octetStream&api-version=5.0)
![中国联通](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2023/2/6/7ffc1d925743.png&$format=octetStream&api-version=5.0)

## 用作图床
### Picgo 插件
[Picgo](https://github.com/PicGo/PicGo-Core) 是使用 Node 制作的图片上传软件，提供了简单易用的 API，社区可以方便地制作插件。  
我为这个简单需求写了 [picgo-plugin-azureimg](https://github.com/YukiCoco/picgo-plugin-azureimg)，只需要简单配置就能使用了。  
#### 安装 Picgo
使用安装命令安装 Picgo：
````
npm install picgo -g
````
#### 安装插件
使用安装命令安装此插件：
````
picgo install azureimg
````
#### 配置
使用配置命令配置插件：
````
picgo set uploader azureimg #配置 azureimg
picgo use transformer #选择 path
picgo use uploader #选择 azureimg 作为 uploader
````
#### 使用
使用命令上传文件：
````
picgo u /path/to/image.jpg
````