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
# Cài Node.js repository
curl -fsSL https://deb.nodesource.com/setup_22.x | bash -
apt-get install -y nodejs

# Kiểm tra phiên bản
node --version  # v22.22.2 hoặc mới hơn
```

## Bước 3: Cài Đặt OpenClaw

```bash
npm install -g openclaw@2026.5.22
openclaw --version
```

## Bước 4: Cấu Hình Environment

```bash
# Set OLLAMA_HOST (chỉ đến server Ollama)
echo 'export OLLAMA_HOST=http://103.151.52.186:11434' >> /root/.bashrc
source /root/.bashrc
```

## Bước 5: Tạo File Cấu Hình

```bash
mkdir -p /root/.openclaw

# Tạo openclaw.json
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
  "hooks": {
    "internal": {
      "enabled": true,
      "entries": {
        "session-memory": {
          "enabled": true
        }
      }
    }
  }
}
EOF
```

**Thay thế:**
- `YOUR_GATEWAY_TOKEN`: Token gateway (sinh ngẫu nhiên: `openssl rand -hex 16`)
- `YOUR_TELEGRAM_BOT_TOKEN`: Token bot Telegram từ BotFather

## Bước 6: Khởi Động Gateway

```bash
# Khởi động gateway
export OLLAMA_HOST='http://103.151.52.186:11434'
nohup /usr/bin/node /usr/lib/node_modules/openclaw/dist/index.js gateway --port 18789 > /var/log/openclaw.log 2>&1 &

# Kiểm tra status
cat /var/log/openclaw.log
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
# Kiểm tra gateway
curl -s http://127.0.0.1:18789/health

# Kiểm tra Ollama
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
# Kiểm tra log
cat /var/log/openclaw.log
# Hoặc
tail -f /tmp/openclaw/openclaw-$(date +%Y-%m-%d).log
```

### Ollama không kết nối
```bash
# Kiểm tra kết nối
curl -s http://103.151.52.186:11434/api/tags
# Nếu không có phản hồi, kiểm tra firewall hoặc IP
```

### Telegram bot không hoạt động
```bash
# Kiểm tra token trong config
cat /root/.openclaw/openclaw.json | python3 -c "import sys,json; print(json.load(sys.stdin)['channels']['telegram']['botToken'])"
```

## Thông Tin Server

| Thành phần | Thông tin |
|------------|-----------|
| Ollama Server | 103.151.52.186:11434 |
| Model | qwen3.6:35b-a3b (36B params) |
| OpenClaw Version | 2026.5.22 |
| Node.js | v22.22.2+ |

## Liên Hệ Hỗ Trợ

Nếu gặp sự cố, kiểm tra log:
```bash
journalctl -u openclaw-gateway -f
# Hoặc
tail -f /var/log/openclaw.log
```
