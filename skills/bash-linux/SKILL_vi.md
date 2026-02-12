---
name: bash-linux
description: Các mẫu Bash/Linux terminal. Các lệnh quan trọng, piping, xử lý lỗi, scripting. Sử dụng khi làm việc trên hệ thống macOS hoặc Linux.
allowed-tools: Read, Write, Edit, Glob, Grep, Bash
---

# Các Mẫu Bash Linux

> Các mẫu thiết yếu cho Bash trên Linux/macOS.

---

## 1. Cú pháp Toán tử

### Chuỗi Lệnh

| Toán tử | Ý nghĩa                        | Ví dụ                               |
| ------- | ------------------------------ | ----------------------------------- |
| `;`     | Chạy tuần tự                   | `cmd1; cmd2`                        |
| `&&`    | Chạy nếu lệnh trước thành công | `npm install && npm run dev`        |
| `\|\|`  | Chạy nếu lệnh trước thất bại   | `npm test \|\| echo "Tests failed"` |
| `\|`    | Pipe đầu ra                    | `ls \| grep ".js"`                  |

---

## 2. Hoạt động Tệp

### Các Lệnh Thiết yếu

| Tác vụ             | Lệnh                                 |
| ------------------ | ------------------------------------ |
| Liệt kê tất cả     | `ls -la`                             |
| Tìm tệp            | `find . -name "*.js" -type f`        |
| Nội dung tệp       | `cat file.txt`                       |
| N dòng đầu         | `head -n 20 file.txt`                |
| N dòng cuối        | `tail -n 20 file.txt`                |
| Theo dõi log       | `tail -f log.txt`                    |
| Tìm kiếm trong tệp | `grep -r "pattern" --include="*.js"` |
| Kích thước tệp     | `du -sh *`                           |
| Sử dụng đĩa        | `df -h`                              |

---

## 3. Quản lý Tiến trình

| Tác vụ             | Lệnh                          |
| ------------------ | ----------------------------- |
| Liệt kê tiến trình | `ps aux`                      |
| Tìm theo tên       | `ps aux \| grep node`         |
| Diệt theo PID      | `kill -9 <PID>`               |
| Tìm user cổng      | `lsof -i :3000`               |
| Diệt cổng          | `kill -9 $(lsof -t -i :3000)` |
| Chạy nền           | `npm run dev &`               |
| Công việc (Jobs)   | `jobs -l`                     |
| Đưa lên nền trước  | `fg %1`                       |

---

## 4. Xử lý Văn bản

### Công cụ Cốt lõi

| Công cụ | Mục đích       | Ví dụ                           |
| ------- | -------------- | ------------------------------- |
| `grep`  | Tìm kiếm       | `grep -rn "TODO" src/`          |
| `sed`   | Thay thế       | `sed -i 's/old/new/g' file.txt` |
| `awk`   | Trích xuất cột | `awk '{print $1}' file.txt`     |
| `cut`   | Cắt trường     | `cut -d',' -f1 data.csv`        |
| `sort`  | Sắp xếp dòng   | `sort -u file.txt`              |
| `uniq`  | Dòng duy nhất  | `sort file.txt \| uniq -c`      |
| `wc`    | Đếm            | `wc -l file.txt`                |

---

## 5. Biến Môi trường

| Tác vụ           | Lệnh                            |
| ---------------- | ------------------------------- |
| Xem tất cả       | `env` hoặc `printenv`           |
| Xem một          | `echo $PATH`                    |
| Đặt tạm thời     | `export VAR="value"`            |
| Đặt trong script | `VAR="value" command`           |
| Thêm vào PATH    | `export PATH="$PATH:/new/path"` |

---

## 6. Mạng

| Tác vụ         | Lệnh                                                                        |
| -------------- | --------------------------------------------------------------------------- |
| Tải xuống      | `curl -O https://example.com/file`                                          |
| Yêu cầu API    | `curl -X GET https://api.example.com`                                       |
| POST JSON      | `curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' URL` |
| Kiểm tra cổng  | `nc -zv localhost 3000`                                                     |
| Thông tin mạng | `ifconfig` (cũ) hoặc `ip addr` (mới)                                        |

---

## 7. Mẫu Script

```bash
#!/bin/bash
set -euo pipefail  # Thoát khi lỗi, biến không xác định, pipe thất bại

# Màu sắc (tùy chọn)
RED='\033[0;31m'
GREEN='\033[0;32m'
NC='\033[0m'

# Thư mục script
SCRIPT_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

# Hàm
log_info() { echo -e "${GREEN}[INFO]${NC} $1"; }
log_error() { echo -e "${RED}[ERROR]${NC} $1" >&2; }

# Main
main() {
    log_info "Starting..."
    # Logic của bạn ở đây
    log_info "Done!"
}

main "$@"
```

---

## 8. Các Mẫu Phổ biến

### Kiểm tra xem lệnh có tồn tại không

```bash
if command -v node &> /dev/null; then
    echo "Node is installed"
fi
```

### Giá trị biến mặc định

```bash
NAME=${1:-"default_value"}
```

### Đọc tệp từng dòng

```bash
while IFS= read -r line; do
    echo "$line"
done < file.txt
```

### Vòng lặp qua các tệp

```bash
for file in *.js; do
    echo "Processing $file"
done
```

---

## 9. Sự khác biệt so với PowerShell

| Tác vụ        | PowerShell               | Bash                  |
| ------------- | ------------------------ | --------------------- |
| Liệt kê tệp   | `Get-ChildItem`          | `ls -la`              |
| Tìm tệp       | `Get-ChildItem -Recurse` | `find . -type f`      |
| Môi trường    | `$env:VAR`               | `$VAR`                |
| Nối chuỗi     | `"$a$b"`                 | `"$a$b"` (giống nhau) |
| Kiểm tra null | `if ($x)`                | `if [ -n "$x" ]`      |
| Pipeline      | Dựa trên đối tượng       | Dựa trên văn bản      |

---

## 10. Xử lý Lỗi

### Thiết lập các tùy chọn

```bash
set -e          # Thoát khi có lỗi
set -u          # Thoát khi biến không xác định
set -o pipefail # Thoát khi pipe thất bại
set -x          # Debug: in các lệnh
```

### Trap để dọn dẹp

```bash
cleanup() {
    echo "Cleaning up..."
    rm -f /tmp/tempfile
}
trap cleanup EXIT
```

---

> **Ghi nhớ:** Bash dựa trên văn bản. Sử dụng `&&` cho các chuỗi thành công, `set -e` để an toàn, và trích dẫn các biến của bạn!
