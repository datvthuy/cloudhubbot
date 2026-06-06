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

## Bước 3: Cấu hình thay đổi Telegram BOT API để có thể chat với bot của mình

Đi đến đường dẫn có file cấu hình
```
cd /root/.openclaw
```
Chỉnh sữa file cấu hình sử dụng bot API token của bạn bằng lệnh sau:

```
nano openclaw.json
```
Đi tới dòng 157 và thay đổi API token của riêng bạn đã lấy ở bước trên. Lưu lại file.
<img width="1708" height="722" alt="image" src="https://github.com/user-attachments/assets/f13fc2a3-3189-43e8-9f8c-bb8b9b6b0122" />

Tiến hành restart lại cấu hình openclaw gateway
```
openclaw gateway restart --safe
```
<img width="1328" height="374" alt="image" src="https://github.com/user-attachments/assets/f9d759df-8b5d-4066-b964-c9860482e857" />

## Chat với bot của bạn thôi.
<img width="1706" height="950" alt="image" src="https://github.com/user-attachments/assets/946639ae-a444-422c-ab77-7dbbbc9a5751" />
