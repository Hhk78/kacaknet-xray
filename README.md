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
      "listen": "0.0.0.0",
      "port": 1234,
      "protocol": "vmess",
      "settings": {
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
      "tag": "direct"
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
      "protocol": "vmess",
      "settings": {
        "vnext": [
          {
            "address": "server-ip",
            "port": 1234,
            "users": [
              {
                "id": "742893b0-0532-49d6-a369-2826085b822d",
                "security": "none"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "tlsSettings": {
          "serverName": "m.youtube.com",  // SNI spoofing burada istediginizi yazın
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
