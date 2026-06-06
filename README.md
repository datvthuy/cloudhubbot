# Hướng Dẫn Cài Đặt OpenClaw - Bot Telegram AI - Ubuntu

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

## Bước 2: Tạo Telegram BOT API

Các bước tạo Telegram Bot và lấy API Token:Mở BotFather: Mở ứng dụng Telegram, tìm kiếm từ khóa @BotFather và chọn tài khoản có dấu tích xanh.

Khởi động: Nhấn nút Start (Bắt đầu) hoặc gõ /start và gửi cho BotFather.

Tạo bot mới: Gõ lệnh /newbot và gửi đi.Đặt tên cho bot: Nhập tên hiển thị cho bot của bạn (ví dụ: Shop Bot) và gửi đi.

Tạo tên người dùng: Nhập Username (Tên tài khoản) cho bot. Lưu ý: Tên này phải là duy nhất và bắt buộc kết thúc bằng chữ "bot" (ví dụ: MyNew_bot hoặc mynewbot).

Lấy API Token: Ngay sau đó, BotFather sẽ gửi cho bạn một tin nhắn chứa Token API (một chuỗi dài có dạng 123456789:ABCdefGhIJKlmNoPQRsTUVwxyZ). Hãy giữ bí mật mã này.

<img width="1588" height="388" alt="image" src="https://github.com/user-attachments/assets/d574b132-1632-4c39-9978-376cbff3a7f6" />
