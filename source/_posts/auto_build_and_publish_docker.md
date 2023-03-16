---
title: 让 GitHub Actions 自动 Build 并 Push Docker 到 Dockcer Hub
date: 2022-08-18 21:41
tags: DevOps
---
![GitHub Actions](https://niconiacg.visualstudio.com/a158c0e6-f968-4c72-b336-690f5a7c8b4c/_apis/git/repositories/f4cb0aa8-76ef-49d2-bee3-9a3deda56975/items?path=/2022/11/c5d70f9575dc4f9cc93dd21e9133c01a.webp&%24format=octetStream&api-version=5.0)
最近在写 [CheapSteam][1] 的验证相关内容，现在已经可以将它 [部署在服务器][2]。为此也制作了一个 Docker 镜像，当然不能每一个版本都靠我手动 Push，和自动发布到 Release 一样，安排上 GitHub Actions 的自动构建。
<!-- more -->
## 编写 Actions 配置
这里直接拉 CheapSteam 我写好的 [配置文件][3] 过来用。
````yml
name: Publish Docker

on:
  push:
    tags:        
      - 'v*'           # Push events to every tag not containing /

    # Pattern matched against refs/tags
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set env
      run: echo "RELEASE_VERSION=${GITHUB_REF#refs/*/}" >> $GITHUB_ENV
    - name: Test
      run: |
        echo $RELEASE_VERSION
        echo ${{ env.RELEASE_VERSION }}
    - name: Setup .NET
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 6.0.x
    - name: Restore dependencies
      run: dotnet restore CheapSteam.UI/CheapSteam.UI/CheapSteam.UI.csproj
    - name: Publish Linux x64
      run: dotnet publish CheapSteam.UI/CheapSteam.UI/CheapSteam.UI.csproj --runtime linux-x64 -p:PublishSingleFile=true --self-contained true -o ./CheapSteam-${{ env.RELEASE_VERSION }}-linux-x64 -c Release
    - name: Edit default listen url
      run: sed -i 's/127.0.0.1/0.0.0.0/g' ./CheapSteam-${{ env.RELEASE_VERSION }}-linux-x64/appsettings.json
    - name: Zip files
      run: zip -r CheapSteam-${{ env.RELEASE_VERSION }}-linux-x64.zip ./CheapSteam-${{ env.RELEASE_VERSION }}-linux-x64
    - name: Copy bin files
      run: cp CheapSteam-${{ env.RELEASE_VERSION }}-linux-x64.zip docker/
    -
      name: Set up QEMU
      uses: docker/setup-qemu-action@v2
    -
      name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    -
      name: Login to DockerHub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
    -
      name: Edit default dockerfile
      run: sed -i 's/RELEASE_VERSION/${{ env.RELEASE_VERSION }}/g' docker/dockerfile
    -
      name: Build and push
      uses: docker/build-push-action@v3
      with:
        push: true
        tags: sayokurisu/cheapsteam:latest,sayokurisu/cheapsteam:${{ env.RELEASE_VERSION }}
        context: docker
````
需要关注的地方在
````yml
    -
      name: Set up QEMU
      uses: docker/setup-qemu-action@v2
    -
      name: Set up Docker Buildx
      uses: docker/setup-buildx-action@v2
    -
      name: Login to DockerHub
      uses: docker/login-action@v2
      with:
        username: ${{ secrets.DOCKERHUB_USERNAME }}
        password: ${{ secrets.DOCKERHUB_TOKEN }}
    -
      name: Build and push
      uses: docker/build-push-action@v3
      with:
        push: true
        tags: sayokurisu/cheapsteam:latest,sayokurisu/cheapsteam:${{ env.RELEASE_VERSION }}
        context: docker
````
在 `Security -> Actions secrets -> New repository secret` 添加 secret.  
创建 `DOCKERHUB_USERNAME` 填写为 [Docker Hub][4] 的用户名。  
创建 `DOCKERHUB_TOKEN` 填写为 [Access Token][5].  
这样就有了上面的环境变量，接着只需要修改最后一个 step。
`context` 为项目的 dockerfile 所在文件夹，我这里是在根目录的 `docker` 文件夹里面。
`tags` 为要发布的 Docker 的 Tag，可以添加多个，用 ',' 号分隔。  
一般改了这些东西就能用，更多的修改请在 [官方仓库][6] 查看。  


  [1]: https://github.com/YukiCoco/CheapSteam
  [2]: https://github.com/YukiCoco/CheapSteam/blob/master/Docs/Deploy.md
  [3]: https://github.com/YukiCoco/CheapSteam/blob/master/.github/workflows/publish-docker.yml
  [4]: https://hub.docker.com/
  [5]: https://hub.docker.com/settings/security
  [6]: https://github.com/marketplace/actions/build-and-push-docker-images