
# Xray Senaryo 1 [WS over TLS (server-client)]
```bash
{
  "log": {
    "loglevel": "debug"
  },
  "routing": {
    "domainStrategy": "AsIs",
    "rules": [
      {
        "type": "field",
        "ip": [
          "geoip:private"
        ],
        "outboundTag": "block"
      }
    ]
  },
  "inbounds": [
    {
      "listen": "127.0.0.1",
      "port": 65535,
      "protocol": "vless",
      "settings": {
        "decryption": "none",
        "clients": [
          {
            "id": "742893b0-0532-49d6-a369-2826085b822d"
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "tlsSettings": {
          "serverName": "m.youtube.com",
          "certificates": [
            {
              "certificateFile": "./certificate.crt",
              "keyFile": "./private.key"
            }
          ]
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "tag": "direct",
      "streamSettings": {
        "sockopt": {
          "mark": 100
        }
      }
    },
    {
      "protocol": "blackhole",
      "tag": "block"
    }
  ]
}


```

```bash
{
  "log": {
    "loglevel": "debug"
  },
  "routing": {
    "domainStrategy": "AsIs",
    "rules": [
      {
        "type": "field",
        "ip": [
          "geoip:private"
        ],
        "outboundTag": "direct"
      }
    ]
  },
  "inbounds": [
    {
      "listen": "127.0.0.1",
      "port": 1080,
      "protocol": "socks",
      "settings": {
        "auth": "noauth",
        "udp": true,
        "ip": "127.0.0.1"
      }
    },
    {
      "listen": "127.0.0.1",
      "port": 1081,
      "protocol": "http"
    }
  ],
  "outbounds": [
    {
      "protocol": "vless",
      "settings": {
        "vnext": [
          {
            "address": "x.x.x.x",
            "port": 443,
            "users": [
              {
                "id": "742893b0-0532-49d6-a369-2826085b822d",
                "encryption": "none"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "tlsSettings": {
          "serverName": "m.youtube.com",
          "fingerprint": "chrome",
          "allowInsecure": true
        }
      },
      "tag": "proxy"
    },
    {
      "protocol": "freedom",
      "tag": "direct"
    }
  ]
}


```

```bash
<VirtualHost *:443>
    ServerName m.youtube.com

    SSLEngine On
    SSLCertificateFile /root/xray-cert.pem
    SSLCertificateKeyFile /root/xray-key.pem

    SSLProxyEngine On
    SSLProxyVerify none
    SSLProxyCheckPeerCN off
    SSLProxyCheckPeerName off
    SSLProxyCheckPeerExpire off

    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} =websocket [NC]
    RewriteRule /(.*) wss://localhost:65535/$1 [P,L]

    ProxyPass / https://127.0.0.1:65535
    ProxyPassReverse / https://127.0.0.1:65535
</VirtualHost>
```
```bash
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /root/xray-key.pem -out /root/xray-cert.pem
```
