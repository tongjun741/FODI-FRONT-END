---
tags:
    - NAS
---

## 安装docker插件 + v2ray 映像

**注意：X86 架构的 CPU 支持 docker，ARM 架构的 CPU 不支持 docker**

在群辉套件中心搜索 `docker` 并安装

![img](/img-post/开发/NAS/群辉 NAS docker 中配置 v2ray 客户端.assets/nas-docker-1.png)

打开 docker，搜索 v2ray，双击下载 `v2ray/official` 即可

![img](/img-post/开发/NAS/群辉 NAS docker 中配置 v2ray 客户端.assets/nas-docker-2.png)

选择刚刚下载的映像，点击启动

![img](/img-post/开发/NAS/群辉 NAS docker 中配置 v2ray 客户端.assets/nas-docker-3.png)

输入容器名称，点击高级设置

![img](/img-post/开发/NAS/群辉 NAS docker 中配置 v2ray 客户端.assets/nas-docker-4.png)

在高级设置中勾选启用自动重新启动和创建桌面快捷方式

![img](/img-post/开发/NAS/群辉 NAS docker 中配置 v2ray 客户端.assets/nas-docker-5.png)

在卷中添加文件夹，装载路径为 `/etc`。需要提前在file station 中创建好 `docker/v2ray` 文件夹

![img](/img-post/开发/NAS/群辉 NAS docker 中配置 v2ray 客户端.assets/nas-docker-6.png)

在网络中勾选 使用与 Docker Host 相同的网络

![img](/img-post/开发/NAS/群辉 NAS docker 中配置 v2ray 客户端.assets/nas-docker-7.png)

其他默认，点击应用

点击下一步

![img](/img-post/开发/NAS/群辉 NAS docker 中配置 v2ray 客户端.assets/nas-docker-8.png)

将 `向导完成后运行此容器` 取消选中（因为还没有配置文件）。点击应用

![img](/img-post/开发/NAS/群辉 NAS docker 中配置 v2ray 客户端.assets/nas-docker-9.png)

## v2ray 客户端配置文件

可以用gui客户端中配置好服务器，直接导出客户端配置文件为 `config.json`，我使用的是 v2rayN

编辑 config.json， 修改 listen 为 0.0.0.0, protocol 为 `http`，port 随意，也可以不改

在 v2rayN 程序文件夹中 找到 geoip.dat geosite.dat 文件

在刚刚创建的 `docker/v2ray`文件夹中在创建一个 v2ray 文件夹，将 config.json geoip.dat geosite.dat 三个文件放进去

我的部分 `config.json` 配置

```conf
{
  "inbounds": [
    {
      "tag": "proxy",
      "port": 10808,
      "listen": "0.0.0.0",
      "protocol": "http",
      "sniffing": {
        "enabled": true,
        "destOverride": [
          "http",
          "tls"
        ]
      },
      "settings": {
        "auth": "noauth",
        "udp": true,
        "ip": null,
        "address": null,
        "clients": null
      },
      "streamSettings": null
    }
  ]
}
```

最后启动刚刚创建好的 docker 即可

### 连接

到这了只是在群辉中安装了 v2ray 客户端，并配置了服务器，还需要手动连接代理，在群辉中，你需要在 控制面板-网络-常规-代理服务器 中添加代理

![img](/img-post/开发/NAS/群辉 NAS docker 中配置 v2ray 客户端.assets/A44354EE-BCE3-40b5-B9D8-642D492954F6.png)

如果你的其他设备想要连接也需要相同的方式手动配置代理

在 win10 设置中，如下图所示位置，代理方式只支持 http 代理（这就是为什么上步中要把protocol改为http）地址为你群规的ip，端口就是上步中设置的端口，配置完之后保存即可

![img](/img-post/开发/NAS/群辉 NAS docker 中配置 v2ray 客户端.assets/2020-08-16-13-09-36.png)

手机中同理，在无线中配置代理

如果你想实现只需连接家里无线就可以直接实现科学上网，你需要在路由器中做一些配置，可以参考下文【待整理】



https://wqdy.top/1165.html