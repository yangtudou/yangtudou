# 搬瓦工 vps 安装 x-ui 面板

⚠️ 这个是用于个人实践，请不要用这个方法做什么违法乱纪的事情，遵守我中华人民共和国的法律。

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
- 自带的 终端 / cmd
- 软件，如 [FinalShell](https://www.hostbuf.com/t/988.html) 等

推荐用软件，方便后期操作，除非是 vi 、nano 命令操作很牛逼的大神。

1. 配置环境
   1. 输入代码
   ```unminimize```
   一路 y 到底
   1. 重启 vps 
   ```reboot```
2. 启用 BBR TCP 拥塞控制算法
```
apt update
```
```
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
```
```
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
```
```
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
```
添加软连接
```
ln -s  /root/.acme.sh/acme.sh /usr/local/bin/acme.sh
```
切换CA机构 
```
acme.sh --set-default-ca --server letsencrypt
```
申请证书 
```
acme.sh  --issue -d $域名 -k ec-256 --webroot  /var/www/html
```
安装证书
```
acme.sh --install-cert -d $域名 --ecc --key-file /etc/x-ui/server.key --fullchain-file /etc/x-ui/server.crt --reloadcmd "systemctl force-reload nginx"
```
6. **配置 nginx**

配置文件地址：`/etc/nginx/nginx.conf`

配置文件参考：
```
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
    worker_connections 1024;
}

http {
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    include /etc/nginx/mime.types;
    default_type application/octet-stream;
    gzip on;

    server {
        listen 443 ssl;
        
        server_name nicename.co;  #你的域名
        ssl_certificate       /etc/x-ui/server.crt;  #证书位置
        ssl_certificate_key   /etc/x-ui/server.key; #私钥位置
        
        ssl_session_timeout 1d;
        ssl_session_cache shared:MozSSL:10m;
        ssl_session_tickets off;
        ssl_protocols    TLSv1.2 TLSv1.3;
        ssl_prefer_server_ciphers off;

        location / {
            proxy_pass https://bing.com; #伪装网址
            proxy_redirect off;
            proxy_ssl_server_name on;
            sub_filter_once off;
            sub_filter "bing.com" $server_name;
            proxy_set_header Host "bing.com";
            proxy_set_header Referer $http_referer;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header User-Agent $http_user_agent;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto https;
            proxy_set_header Accept-Encoding "";
            proxy_set_header Accept-Language "zh-CN";
        }


        location /ray {   #分流路径
            proxy_redirect off;
            proxy_pass http://127.0.0.1:10000; #Xray端口
            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        
        location /xui {   #xui路径
            proxy_redirect off;
            proxy_pass http://127.0.0.1:9999;  #xui监听端口
            proxy_http_version 1.1;
            proxy_set_header Host $host;
        }
    }

    server {
        listen 80;
        location /.well-known/ {
               root /var/www/html;
            }
        location / {
                rewrite ^(.*)$ https://$host$1 permanent;
            }
    }
}
```
⚠️ 修改好 nginx 配置文件后，记得重启 nginx
```
systemctl reload nginx
```
