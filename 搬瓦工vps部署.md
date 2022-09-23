更新于 2022.9.20

[搬瓦工官网 ↗️](https://bwh88.net/)

⚠️ 以下操作基于：
- 已经购买了 vps
- 已经知道 vps ip 地址，以及 ssh 端口号
- 已经知道 root 密码
- 已经远程连接到 vps 主机上
- 了解一部分 Linux 操作指令
- 使用的是搬瓦工的 vps


ps：多说一嘴远程连接有两种方法
- 自带的 终端/cmd
- 软件，如 FinalShell 等

推荐用软件，方便后期操作，除非是 vi 、nano 命令操作很牛逼的大神。

1. 配置环境
   1. 输入代码```unminimize```一路 y 到底
   2. 重启 vps ```reboot```
2. 启用 BBR TCP 拥塞控制算法
```
apt update
```
```
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
sysctl -p
```
3. [x-ui 面板](https://github.com/vaxilu/x-ui)安装
一键脚本
```bash
bash <(curl -Ls https://raw.githubusercontent.com/vaxilu/x-ui/master/install.sh)
```
4. 安装 nginx
```
apt install nginx -y
```
5. 安装 acme
```
curl https://get.acme.sh | sh
# 添加软连接
ln -s  /root/.acme.sh/acme.sh /usr/local/bin/acme.sh
#切换CA机构 
acme.sh --set-default-ca --server letsencrypt
# 申请证书 
acme.sh  --issue -d dsmyyy.tk -k ec-256 --webroot  /var/www/html
# 安装证书
# 实例 acme.sh --install-cert -d dsmyyy.tk --ecc --key-file /etc/x-ui/server.key --fullchain-file /etc/x-ui/server.crt --reloadcmd "systemctl force-reload nginx"
acme.sh --install-cert -d $域名 --ecc --key-file $证书地址xx.key --fullchain-file $证书地址xx..crt --reloadcmd "systemctl force-reload nginx"
```
6. **配置 nginx**
配置文件地址：`/etc/nginx/nginx.conf`
配置文件参考：
```

```
⚠️ 修改好 nginx 配置文件后，记得重启 nginx
```
systemctl reload nginx
```
