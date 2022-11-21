---
title: '解决 403 错误: 修改 Hosts 使 PS5 在中国大陆区域共享截图'
date: 2022-07-11 15:33
tags: ["游戏", "魔法上网"]
---
![FF7RE Alice][1]
众所周知，国行 PS5 使用备份来使外区账号登陆上国行机器。PlayStation 下载边缘节点有国内的 CDN，所以中国大陆玩家下载游戏的体验还不错。前不久，SONY 推出了 [在 PS APP 查看 PS5 截图][2] 的功能，但在大陆使用会出现 403 错误，导致上传失败。初步猜测原因可能是 PlayStation 未在大陆开启这项功能，所以使用备份大法登录外区账号后，上传图片的边缘节点仍在大陆，所以出现 403 错误。
<!-- more -->
## 解决方案
非常简单!
使用 Ping 工具得到 `ps5.np.playstation.net` 的一个海外 IP (比如 69.192.218.178)，将路由器对此域名解析的 IP 指向该海外 IP 即可。
使用软路由者亦可将此地址直接加入到代理白名单。


  [1]: https://static.kuri.su/2022/11/838015a56bb056edca52710da5135e66.webp
  [2]: https://www.playstation.com/zh-hans-hk/support/games/ps5-game-captures-ps-app/