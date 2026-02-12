---
name: busybox-on-windows
description: Cách sử dụng bản dựng Win32 của BusyBox để chạy nhiều công cụ dòng lệnh UNIX tiêu chuẩn trên Windows.
license: MIT
---

BusyBox là một tệp nhị phân đơn lẻ cài đặt nhiều công cụ Unix phổ biến.

Sử dụng kỹ năng này chỉ trên Windows. Nếu bạn đang ở trên UNIX, thì hãy dừng lại ở đây.

Chỉ chạy các bước sau nếu bạn không thể tìm thấy tệp `busybox.exe` trong cùng thư mục với tài liệu này.
Đây là các lệnh PowerShell, nếu bạn có terminal `cmd.exe` cổ điển, thì bạn phải sử dụng `powershell -Command "..."` để chạy chúng.

1. In loại CPU: `Get-CimInstance -ClassName Win32_Processor | Select-Object Name, NumberOfCores, MaxClockSpeed`
2. In phiên bản OS: `Get-ItemProperty "HKLM:\SOFTWARE\Microsoft\Windows NT\CurrentVersion" | Select-Object ProductName, DisplayVersion, CurrentBuild`
3. Tải xuống bản dựng phù hợp của BusyBox bằng cách chạy một trong các lệnh PowerShell này:
   - 32-bit x86 (ANSI): `$ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -Uri https://frippery.org/files/busybox/busybox.exe -OutFile busybox.exe`
   - 64-bit x86 (ANSI): `$ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -Uri https://frippery.org/files/busybox/busybox64.exe -OutFile busybox.exe`
   - 64-bit x86 (Unicode): `$ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -Uri https://frippery.org/files/busybox/busybox64u.exe -OutFile busybox.exe`
   - 64-bit ARM (Unicode): `$ProgressPreference = 'SilentlyContinue'; Invoke-WebRequest -Uri https://frippery.org/files/busybox/busybox64a.exe -OutFile busybox.exe`

Các lệnh hữu ích:

- Trợ giúp: `busybox.exe --list`
- Các lệnh UNIX có sẵn: `busybox.exe --list`

Sử dụng: Thêm tiền tố lệnh UNIX với `busybox.exe`, ví dụ: `busybox.exe ls -1`

Nếu bạn cần chạy lệnh UNIX dưới một CWD khác, hãy sử dụng đường dẫn tuyệt đối đến `busybox.exe`.

Tài liệu: https://frippery.org/busybox/
BusyBox gốc: https://busybox.net/
