# setup-x-ui-v2ray
This is my personal experience of setting up a v2ray server 
# Requirements
- 2 Servers running Ubuntu 20.04 - One is your local server and one is your foreign server.
I personally userd a dual core with 2 gb of ram for each, how ever a single core with 1 gb of ram is still more than enough.

# Installation
1. First make updates and upgrades

```
sudo apt-get update -y 
sudo apt-get upgrade -y
sudo apt install software-properties-common

sudo apt install python3 python3-venv libaugeas0
sudo python3 -m venv /opt/certbot/
sudo /opt/certbot/bin/pip install --upgrade pip
sudo /opt/certbot/bin/pip install certbot certbot-nginx
sudo ln -s /opt/certbot/bin/certbot /usr/bin/certbot


```

2. Download Run the v2-ui installation script on each of your servers.

```
bash <(curl -Ls https://raw.githubusercontent.com/FranzKafkaYu/x-ui/master/install_en.sh)
```

3. Point your domains (or subdomains) to your servers with an A record.

4. Use certbot to get SSL Certificate. Use your email address and domain name. 

``
sudo certbot certonly --standalone --preferred-challenges http --agree-tos --email your-email-address -d your-domain.com 
``

If you get a note like “Congratulations!..”, it means that now you have SSL certificate for your  domain/sub-domain.

5. Install BBR script for better speed

``
wget -N --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && bash bbr.sh
``
6. Add ssl to your x-ui pannel

To enable HTTPS, choose 面板设置 (Panel settings).
You will need to specify your certificate and key.

```
面板证书公钥文件路径
填写一个 '/' 开头的绝对路径，重启面板生效
Panel certificate public key file path
Fill in an absolute path starting with'/', restart the panel to take effect
```
/etc/letsencrypt/live/your-domain.com/fullchain.pem

```
面板证书密钥文件路径
填写一个 '/' 开头的绝对路径，重启面板生效
Panel certificate key file path
Fill in an absolute path starting with'/', restart the panel to take effect 

```
/etc/letsencrypt/live/your-domain.com/privkey.pem

7. Install Iran Hosted Domains file on your local server

```
wget https://github.com/SamadiPour/iran-hosted-domains/releases/download/202209210046/iran.dat
cp ./iran.dat /usr/local/x-ui/bin
```

8. Make an inbound user in your foreign server and write down your id and port

```
protocol: vmess
port: any random 5 digit number
transmission: ws
```

9. Edit xray related settings (xray 相关设置) in your local server to connect to your foreign and exclude local domains

```
{
    "api": {
      "services": [
        "HandlerService",
        "LoggerService",
        "StatsService"
      ],
      "tag": "api"
    },
    "dns": {
      "servers": [
        "208.67.222.222",
        "208.67.220.220",
        "localhost"
      ]
    },
    "inbounds": [
      {
        "listen": "127.0.0.1",
        "port": 62789,
        "protocol": "dokodemo-door",
        "settings": {
          "address": "127.0.0.1"
        },
        "tag": "api"
      }
    ],
    "outbounds": [
        {
          "tag": "proxy",
          "protocol": "vmess",
          "settings": {
            "vnext": [
              {
                "address": "your-forgen-foreign.com",
                "port": your-foreign-port,
                "users": [
                  {
                    "id": "your-foreign-id",
                    "alterId": 0
                  }
                ]
              }
            ]
          },
          "streamSettings": {
            "network": "ws",
            "security": "none",
            "wsSettings": {
            "path": "/",
            "headers": {}
           }
          },
          "mux": {
            "enabled": true
          }
        },
        {
            "tag": "direct",
            "protocol": "freedom",
            "settings": {}
          }
      ],
    "inboundDetour": null,
    "outboundDetour": [
      {
        "protocol": "freedom",
        "tag": "freedom"
      },
      {
        "protocol": "blackhole",
        "tag": "blackhole"
      }
    ],
    "policy": {
      "system": {
        "statsInboundDownlink": true,
        "statsInboundUplink": true
      }
    },
  "routing": {
    "domainStrategy": "IPIfNonMatch",
    "rules": [
      {
        "inboundTag": [
          "api"
        ],
        "outboundTag": "api",
        "type": "field"
      },
      {
        "ip": [
          "geoip:private"
        ],
        "outboundTag": "blocked",
        "type": "field"
      },
      {
        "outboundTag": "blocked",
        "protocol": [
          "bittorrent"
        ],
        "type": "field"
      },
      { 
        "type": "field", 
        "outboundTag": "direct", 
        "domain": [ 
            "regexp:^.+\\\\.ir$", 
            "ext:iran.dat:ir" 
            ] 
        },
      { 
        "type": "field", 
        "outboundTag": "direct", 
        "ip": [
             "geoip:ir" 
        ] 
      }
    ]
  },
    "stats": {}
  }


```
10. Enjoy!


11. BTW Don't forget to setup an iptables firewall!

- allow port 22 for ssh
- allow port 80 and 433 just incase
- allow your admin panel port (default is 54321 unless you changed it during the installation)
- allow port 62789
- allow your inbound user ports

# User your server with clash

vmess-ws-tls is supported by clash,
use https://v2rayse.com/en/v2ray-clash/ to convert it to a clash config.
Enjoy!

## Free Clients

# Clash

    Windows / Mac / Linux : https://github.com/Fndroid/clash_for_windows_pkg/releases 
    Android: https://github.com/Kr328/ClashForAndroid/releases

# V2ray

Windows:

    Qv2ray : https://github.com/Qv2ray/Qv2ray/releases/download/v2.7.0/Qv2ray-v2.7.0-Windows-Installer.exe
    V2rayN : https://github.com/2dust/v2rayN/releases


Mac:

    Qv2ray : https://github.com/Qv2ray/Qv2ray/releases/download/v2.7.0/Qv2ray-v2.7.0-macOS-x64.dmg
    V2RayX : https://github.com/Cenmrev/V2RayX/releases/



Linux:

    Qv2ray : https://github.com/Qv2ray/Qv2ray/releases/download/v2.7.0/Qv2ray-v2.7.0-linux-x64.AppImage
    V2ray Core: https://github.com/v2fly/v2ray-core/releases
        

Android:

    V2rayNG : https://github.com/2dust/v2rayNG/releases

IOS: 
    
    91VPN : https://apps.apple.com/us/app/91vpn/id1483753706
    OneClick - Safe, Easy & Fast : https://apps.apple.com/us/app/id1545555197
    

# Sources
- https://privacymelon.com/how-to-setup-v2ray-ws-tls-cdn/
- https://privacymelon.com/how-to-install-vless-xtls-xray-core/
- https://seakfind.github.io/2021/10/10/X-UI/
- https://gist.github.com/mahmoud-eskandari/960899f3494a1bffa1a29631dbaf0aee
- https://github.com/SamadiPour/iran-hosted-domains
- https://github.com/proxykingdev/x-ui
- https://github.com/vaxilu/x-ui
