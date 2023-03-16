---
title: 我是如何优化博客访问的
date: 2022-12-05 17:09:49
tags: ["Web"]
---
![Rocket](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2022/12/edc0f7b21060d5445e20aab05d6c7eb2.webp&%24format=octetStream&api-version=5.0)
借着换域名的机会，给博客的速度也优化一手。这篇博文将循序渐进地阐述我为优化访问做出的些许尝试，还有这个过程中的一些思考。
<!-- more -->
这是我最近在听的一首歌，现代刻意复古的 City Pop，喜欢极了：
{% meting "1956998853" "netease" "song" %}
## DNS 分流
目前使用的是由 Hexo 驱动的静态博客，所以对 CDN 的使用非常友好。我使用了这一套方案：海外用户访问此站，访问到由 CloudFlare 提供的 CDN 服务；而中国大陆用户访问本站，则使用 AWS CloudFront。因为 CloudFront 在中国大陆的线路比 CloudFlare 要好些。  
为了达成上述目的，需要从 DNS 开始分流。博主使用 [华为云国际版](https://huaweicloud.com/intl/) 提供的 DNS 解析服务。（免费的分地域、国别解析，点击就送233）  
其次，CloudFlare 免费版并不原生支持 CNAME 解析，需要一个小 Hack：使用 CloudFlare SaaS 提供的 CNAME 服务，将 fallback 修改为目的 IP。具体怎么操作网上也有不少教程，不再赘述。  
> 这里还有一个小插曲，使用 .ru/.su 域名在网页仪表盘无法添加进 Custom Hostnames，但可以通过 API 添加。  

![Custom Hostnames](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2022/12/4e775551f8eac6b106527264a216445c.webp&%24format=octetStream&api-version=5.0)
## CDN 回源优化
为了最大化地缩短 FFTB (Time To First Byte, 第一字节时间)，直接使用由 CDN 厂商提供的静态托管服务，这样就极大地缩短了厂商回源到自己服务器的时间，因此访问者的访问时间就只取决于**访问者至云厂商之间的速度**，这也就是 Serverless 的用处之一。  
### CloudFlare
CloudFlare 在此前推出了 Worker（一个 FaaS 服务）可以作为静态托管来使用。也可以使用 CloudFlare Page，但是得将域名 DNS 改为由 CloudFlare 托管。  
#### 安装 Wrangler CLI
````shell
npm install -g wrangler
#yarn global add wrangler 或者使用 yarn
````
#### 初始化项目
````shell
wrangler init <YOUR_WORKER> # <YOUR_WORKER> 是 Worker 项目名称
````
修改 `wrangler.toml` ：
````yaml
name = "foobar-blog" #博客名称
main = "workers-site/index.js" #index.js 路径
compatibility_date = "2022-11-17"

[site]
bucket = "./public" #Hexo dist 输出的路径，默认就是 public
````
修改 `index.js` 为 [index.js](https://gist.github.com/YukiCoco/503fcc50ac2dcc4853a5260d3884866d).
#### 发布
使用 `hexo g` 生成 dist 到 public 目录后就可以发布了。
````shell
wrangler publish # 将 Workers Site 发布到生产环境
````
### CloudFront
博主选择使用 AWS S3 提供的静态托管服务。  
#### 创建 S3 储存桶
将 `Block All Public Access` 修改为 False.
![S3](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2022/12/6541d8acba5d6c1ce1fa8abf5dce407c.webp&%24format=octetStream&api-version=5.0)  
#### 创建 CloudFront Distribution
前面的内容不再赘述，只说关键的部分。  
创建分发后修改默认的源配置，如下图
![CloudFront Source Config](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2022/12/b47d89ece12a96aded07dc5948b1e7ba.webp&%24format=octetStream&api-version=5.0)  
再点 Copy Policy，将复制到的策略粘贴到 S3 桶的桶策略中：  
![S3 Policy](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2022/12/d42867dc20a29b09015647d8649cdbbe.webp&%24format=octetStream&api-version=5.0)
还需要修改默认访问地址为 index.html，创建一个 CloudFront Function，填写下列内容：
````javascript
function handler(event) {
    var request = event.request;
    var uri = request.uri;

    // Check whether the URI is missing a file name.
    if (uri.endsWith('/')) {
        request.uri += 'index.html';
    }
    // Check whether the URI is missing a file extension.
    else if (!uri.includes('.')) {
        request.uri += '/index.html';
    }

    return request;
}
````
#### 配置 Hexo Deploy 支持
安装 [hexo-deployer-s3-cloudfront](https://github.com/Wouter33/hexo-deployer-s3-cloudfront) 这个插件。  
````shell
npm install hexo-deployer-s3-cloudfront --save
````
我的配置如下，需要提前配置好 `~/.aws/credentials`，你也可以直接在配置中填，参考上面项目的 README.
````yaml
deploy:
  type: aws-s3
  bucket: kurisublog
  region: us-east-1
````
### CDN 的缓存策略
因为没有动态内容需要更新，博主直接设置了全站缓存，当有新文章发布时使用 API 清理旧缓存的内容就行了。  
下面是我使用的文章发布脚本，这里使用到的 AWS cli 需要提前配置好，参考 [AWS Command Line Interface](https://aws.amazon.com/cli/), CloudFlare 的 Token 也要预先申请好并给予 Purge Cache 的权限。  
````shell
CF_TOKEN=YourToken

hexo clean
hexo g
npx wrangler publish #cloudflare
hexo d #aws s3

curl --request POST \
  --url https://api.cloudflare.com/client/v4/zones/<Your Zone Id>/purge_cache \
  -H "Authorization: Bearer $CF_TOKEN" \
  --data '{"files":["https://example.com/","https://example.com/posts/*"]}'

aws cloudfront create-invalidation --distribution-id <Your Distribution Id> --paths "/post/*" "/" --no-cli-pager
````
## 源文件优化
### 主题
博主将原主题使用 CDN 缓存的外部资源都修改为本地提供，除非这个 CDN 使用了国内节点，并能保证 SLA。  
只需要放到 `public/js` 里面，在替换原链接即可。  
因为 HTTP/2 的广泛使用，同一域名下的资源浏览器可以并行下载，所以请将外部资源放在同一个域名下。  
> 安利一本 HTTP 松鼠书的姊妹篇（？）《HTTP/2基础教程》，讲 HTTP/2，言简意赅  

![HTTP/2 基础教程](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2022/12/61df187b6eb2302a431c9a343b2f7524.webp&%24format=octetStream&api-version=5.0)
### 缩小 dist
博主使用 [minify](https://github.com/tdewolff/minify) 进一步缩小生成 dist 的体积。 
### 图片
作为托慢速度的大头，怎么处理图片是一个大问题。  
博主在上传图片之前，使用 [tinypng](https://tinypng.com/) 将图片缩小，再使用 [cwebp](https://developers.google.com/speed/webp/docs/cwebp) 转换为 webp 格式，可以将图片大小降至一半甚至更多。  
可以将下面的脚本保存为 `upimg.sh`，以后直接使用这个命令就可以一键缩小并上传图片：
````shell
upimg.sh /path/to/img.png
````
````shell
path=$1

npx tinypng $path -k <Your Token>
cwebp $path -o $path.webp
npx picgo u $path.webp
````
使用到的 [tinypng](https://www.npmjs.com/package/tinypng-cli), [picgo cli](https://picgo.github.io/PicGo-Core-Doc/) 工具参考官方的 wiki 配置好。  
博主的图片托管在 CloudFlare R2。它是免费的，关于这个如果有机会可以再水一篇。  
## 参考
[将 Hexo 部署到 Cloudflare Workers Site 上的趟坑记录](https://blog.skk.moe/post/deploy-blog-to-cf-workers-site/)  
[How do you set a default root object for subdirectories for a statically hosted website on Cloudfront?](https://stackoverflow.com/questions/31017105/how-do-you-set-a-default-root-object-for-subdirectories-for-a-statically-hosted)  
[Serverless blog with Hexo and S3](https://mkhan.me/serverless-blog-with-hexo-and-s3/)  