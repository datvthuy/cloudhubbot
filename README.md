# Hướng Dẫn Cài Đặt OpenClaw - Bot Telegram AI

## Yêu Cầu Hệ Thống

| Thành phần | Yêu cầu |
|------------|----------|
| OS | Ubuntu 22.04+ |
| RAM | Tối thiểu 4GB |
| Disk | 16GB+ |
| Node.js | v22.19.0+ |

## Bước 1: Kết Nối VPS

```bash
ssh root@<IP_VPS>
# Password: <PASSWORD>
```

## Bước 2: Cài Đặt Node.js 22

```bash
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt-get install -y nodejs
node --version  # v22.22.2 hoặc mới hơn
```

## Bước 3: Cài Đặt OpenClaw

```bash
npm install -g openclaw@2026.5.22
openclaw --version
```

## Bước 4: Cấu Hình Environment

```bash
echo 'export OLLAMA_HOST=http://103.151.52.186:11434' >> /root/.bashrc
source /root/.bashrc
```

## Bước 5: Tạo File Cấu Hình

```bash
mkdir -p /root/.openclaw

cat > /root/.openclaw/openclaw.json << 'EOF'
{
  "agents": {
    "defaults": {
      "workspace": "/root/.openclaw/workspace",
      "model": {
        "primary": "ollama/qwen3.6:35b-a3b"
      },
      "models": {
        "ollama/qwen3.6:35b-a3b": {}
      }
    }
  },
  "gateway": {
    "mode": "local",
    "auth": {
      "mode": "token",
      "token": "YOUR_GATEWAY_TOKEN"
    },
    "port": 18789,
    "bind": "loopback"
  },
  "session": {
    "dmScope": "per-channel-peer"
  },
  "tools": {
    "profile": "coding"
  },
  "plugins": {
    "entries": {
      "telegram": {
        "enabled": true
      }
    }
  },
  "channels": {
    "telegram": {
      "enabled": true,
      "groups": {
        "*": {
          "requireMention": true
        }
      },
      "botToken": "YOUR_TELEGRAM_BOT_TOKEN"
    }
  },
  "commands": {
    "ownerAllowFrom": [
      "telegram:YOUR_TELEGRAM_USER_ID"
    ]
  },
  "models": {
    "providers": {
      "ollama": {
        "baseUrl": "http://103.151.52.186:11434",
        "models": [
          {
            "id": "qwen3.6:35b-a3b",
            "name": "qwen3.6:35b-a3b"
          }
        ]
      }
    }
  }
}
EOF
```

**Thay thế:**
- `YOUR_GATEWAY_TOKEN`: Token gateway (sinh ngẫu nhiên: `openssl rand -hex 16`)
- `YOUR_TELEGRAM_BOT_TOKEN`: Token bot Telegram từ BotFather
- `YOUR_TELEGRAM_USER_ID`: ID Telegram của bạn (chat với `@userinfobot`)

**⚠️ Lưu ý quan trọng:** Section `models.providers` **bắt buộc phải có** và mỗi model cần cả `id` lẫn `name`!

## Bước 6: Khởi Động Gateway

```bash
export OLLAMA_HOST='http://103.151.52.186:11434'
nohup /usr/bin/node /usr/lib/node_modules/openclaw/dist/index.js gateway --port 18789 > /var/log/openclaw.log 2>&1 &

# Kiểm tra log
cat /var/log/openclaw.log | grep 'agent model'
# Kết quả: agent model: ollama/qwen3.6:35b-a3b
```

## Bước 7: Cài Đặt Systemd Service (Tùy Chọn)

```bash
cat > /etc/systemd/system/openclaw-gateway.service << EOF
[Unit]
Description=OpenClaw Gateway
After=network.target

[Service]
Type=simple
User=root
Environment=OLLAMA_HOST=http://103.151.52.186:11434
ExecStart=/usr/bin/node /usr/lib/node_modules/openclaw/dist/index.js gateway --port 18789
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
EOF

systemctl daemon-reload
systemctl enable openclaw-gateway
systemctl start openclaw-gateway
```

## Kiểm Tra Hoạt Động

```bash
curl -s http://127.0.0.1:18789/health
curl -s $OLLAMA_HOST/api/tags | python3 -c "import sys,json; print(json.load(sys.stdin)['models'][0]['name'])"
```

## Kiểm Tra Bot Telegram

1. Mở Telegram
2. Tìm bot: `@Cloudhub_Free_OpenClaw_bot`
3. Gửi tin nhắn `/start`
4. Bot sẽ phản hồi nếu đã kết nối thành công

## Khắc Phục Sự Cố

### Gateway không khởi động
```bash
cat /var/log/openclaw.log
# Hoặc
tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log
```

### Lỗi: Unknown model / FailoverError
**Triệu chứng:** Bot trả về "Something went wrong while processing your request"
**Nguyên nhân:** Thiếu hoặc sai cấu hình `models.providers`

**Cách sửa:**
```bash
# Kiểm tra config
cat /root/.openclaw/openclaw.json | python3 -m json.tool | grep -A 10 'models'

# Khẩn cấp: Sửa lại models.providers
python3 << 'PYEOF'
import json
with open('/root/.openclaw/openclaw.json') as f:
    config = json.load(f)

if 'models' not in config:
    config['models'] = {}
if 'providers' not in config['models']:
    config['models']['providers'] = {}
config['models']['providers']['ollama'] = {
    'baseUrl': 'http://103.151.52.186:11434',
    'models': [{'id': 'qwen3.6:35b-a3b', 'name': 'qwen3.6:35b-a3b'}]
}

with open('/root/.openclaw/openclaw.json', 'w') as f:
    json.dump(config, f, indent=2)
print('Fixed!')
PYEOF

# Restart gateway
systemctl restart openclaw-gateway
# Hoặc nếu chạy manual:
pkill -9 -f 'openclaw.*gateway'
export OLLAMA_HOST='http://103.151.52.186:11434'
nohup /usr/bin/node /usr/lib/node_modules/openclaw/dist/index.js gateway --port 18789 > /var/log/openclaw.log 2>&1 &
```

### Ollama không kết nối
```bash
curl -s http://103.151.52.186:11434/api/tags
```

### Telegram bot không hoạt động
```bash
cat /root/.openclaw/openclaw.json | python3 -c "import sys,json; print(json.load(sys.stdin)['channels']['telegram']['botToken'])"
```

### OpenClaw access not configured (cần approve pairing)
```bash
# Khi user mới chat với bot lần đầu, bot sẽ hiển thị pairing code
# Owner chạy lệnh approve:
openclaw pairing approve telegram <PAIRING_CODE>
# Ví dụ: openclaw pairing approve telegram TYD7P9KS
```

### Thêm User Khác Sử Dụng Bot
**Cách 1: Dùng lệnh pairing**
```bash
# Khi user chat lần đầu, bot hiển thị pairing code
openclaw pairing approve telegram <PAIRING_CODE>
```

**Cách 2: Thêm trực tiếp vào config**
```bash
# Thêm section pairing vào openclaw.json
"pairing": {
  "allowedUsers": [
    "telegram:1042785372",
    "telegram:XXXXXXXXXX"
  ]
}
```

**Tìm Telegram User ID:** Chat với `@userinfobot` trên Telegram

## Thông Tin Server

| Thành phần | Thông tin |
|------------|-----------|
| Ollama Server | 103.151.52.186:11434 |
| Model | qwen3.6:35b-a3b (36B params) |
| OpenClaw Version | 2026.5.22 |
| Node.js | v22.22.2+ |

## Liên Hệ Hỗ Trợ

```bash
journalctl -u openclaw-gateway -f
# Hoặc
tail -f /var/log/openclaw.log
```
