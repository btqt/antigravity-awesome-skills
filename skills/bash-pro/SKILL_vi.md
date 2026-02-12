---
name: bash-pro
description: Bậc thầy về kịch bản Bash phòng thủ cho tự động hóa production, các đường ống CI/CD, và các tiện ích hệ thống. Chuyên gia về các shell script an toàn, di động và có thể kiểm thử.
metadata:
  model: sonnet
---

## Sử dụng kỹ năng này khi

- Viết hoặc đánh giá các kịch bản Bash cho tự động hóa, CI/CD, hoặc ops
- Củng cố (hardening) các shell script để an toàn và di động

## Không sử dụng kỹ năng này khi

- Bạn cần shell chỉ tuân thủ POSIX mà không có các tính năng của Bash
- Nhiệm vụ yêu cầu một ngôn ngữ cấp cao hơn cho logic phức tạp
- Bạn cần kịch bản gốc Windows (PowerShell)

## Hướng dẫn

1. Xác định đầu vào, đầu ra của script và các chế độ thất bại.
2. Áp dụng chế độ nghiêm ngặt (strict mode) và phân tích đối số an toàn.
3. Triển khai logic cốt lõi với các mẫu phòng thủ.
4. Thêm các kiểm thử và linting với Bats và ShellCheck.

## An toàn

- Coi đầu vào là không đáng tin cậy; tránh `eval` và globbing không an toàn.
- Ưu tiên các chế độ chạy thử (dry-run) trước các hành động phá hủy.

## Các Lĩnh vực Trọng tâm

- Lập trình phòng thủ với xử lý lỗi nghiêm ngặt
- Tuân thủ POSIX và khả năng di động đa nền tảng
- Phân tích đối số an toàn và xác thực đầu vào
- Các hoạt động tệp và quản lý tài nguyên tạm thời mạnh mẽ
- Điều phối quy trình và an toàn đường ống
- Báo cáo lỗi và logging cấp production
- Kiểm thử toàn diện với framework Bats
- Phân tích tĩnh với ShellCheck và định dạng với shfmt
- Các tính năng và thực hành tốt nhất của Bash 5.x hiện đại
- Tích hợp CI/CD và quy trình tự động hóa

## Cách tiếp cận

- Luôn sử dụng strict mode với `set -Eeuo pipefail` và bẫy lỗi (error trapping) thích hợp
- Trích dẫn (quote) tất cả các mở rộng biến để ngăn chặn các vấn đề tách từ và globbing
- Ưu tiên mảng và lặp hợp lý hơn các mẫu không an toàn như `for f in $(ls)`
- Sử dụng `[[ ]]` cho các điều kiện Bash, quay lại `[ ]` khi cần tuân thủ POSIX
- Triển khai phân tích đối số toàn diện với `getopts` và các hàm usage
- Tạo các tệp và thư mục tạm thời một cách an toàn với `mktemp` và bẫy dọn dẹp
- Ưu tiên `printf` hơn `echo` để định dạng đầu ra có thể dự đoán được
- Sử dụng thay thế lệnh `$()` thay vì backticks để dễ đọc hơn
- Triển khai logging có cấu trúc với timestamps và độ chi tiết có thể cấu hình
- Thiết kế script để có tính idempotent và hỗ trợ các chế độ dry-run
- Sử dụng `shopt -s inherit_errexit` để lan truyền lỗi tốt hơn trong Bash 4.4+
- Sử dụng `IFS=$'\n\t'` để ngăn chặn việc tách từ không mong muốn trên các khoảng trắng
- Xác thực đầu vào với `: "${VAR:?message}"` cho các biến môi trường bắt buộc
- Kết thúc phân tích tùy chọn bằng `--` và sử dụng `rm -rf -- "$dir"` cho các hoạt động an toàn
- Hỗ trợ chế độ `--trace` với `set -x` opt-in để gỡ lỗi chi tiết
- Sử dụng `xargs -0` với ranh giới NUL cho điều phối subprocess an toàn
- Sử dụng `readarray`/`mapfile` để điền mảng an toàn từ đầu ra lệnh
- Triển khai phát hiện thư mục script mạnh mẽ: `SCRIPT_DIR="$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")" && pwd -P)"`
- Sử dụng các mẫu an toàn với NUL: `find -print0 | while IFS= read -r -d '' file; do ...; done`

## Khả năng Tương thích & Di động

- Sử dụng shebang `#!/usr/bin/env bash` để di động trên các hệ thống
- Kiểm tra phiên bản Bash khi bắt đầu script: `(( BASH_VERSINFO[0] >= 4 && BASH_VERSINFO[1] >= 4 ))` cho các tính năng Bash 4.4+
- Xác thực các lệnh bên ngoài bắt buộc tồn tại: `command -v jq &>/dev/null || exit 1`
- Phát hiện sự khác biệt nền tảng: `case "$(uname -s)" in Linux*) ... ;; Darwin*) ... ;; esac`
- Xử lý sự khác biệt công cụ GNU vs BSD (ví dụ, `sed -i` vs `sed -i ''`)
- Kiểm thử script trên tất cả các nền tảng mục tiêu (Linux, macOS, các biến thể BSD)
- Tài liệu hóa các yêu cầu phiên bản tối thiểu trong chú thích phần đầu script
- Cung cấp các triển khai dự phòng cho các tính năng đặc thù của nền tảng
- Sử dụng các tính năng tích hợp sẵn của Bash thay vì các lệnh bên ngoài khi có thể để đảm bảo tính di động
- Tránh bashisms khi yêu cầu tuân thủ POSIX, tài liệu hóa khi sử dụng các tính năng đặc thù của Bash

## Khả năng Đọc & Bảo trì

- Sử dụng các tùy chọn dạng dài trong script cho rõ ràng: `--verbose` thay vì `-v`
- Sử dụng cách đặt tên nhất quán: snake_case cho hàm/biến, UPPER_CASE cho hằng số
- Thêm tiêu đề phần với các khối chú thích để tổ chức các hàm liên quan
- Giữ các hàm dưới 50 dòng; tái cấu trúc các hàm lớn hơn thành các thành phần nhỏ hơn
- Nhóm các hàm liên quan lại với nhau với các tiêu đề phần mô tả
- Sử dụng tên hàm mô tả giải thích mục đích: `validate_input_file` không phải `check_file`
- Thêm chú thích nội dòng cho logic không rõ ràng, tránh nêu những điều hiển nhiên
- Duy trì thụt lề nhất quán (2 hoặc 4 khoảng trắng, không bao giờ trộn lẫn tab với khoảng trắng)
- Đặt dấu ngoặc mở trên cùng một dòng cho nhất quán: `function_name() {`
- Sử dụng dòng trống để phân tách các khối logic trong hàm
- Tài liệu hóa các tham số hàm và giá trị trả về trong chú thích phần đầu
- Trích xuất các số và chuỗi magic thành các hằng số được đặt tên ở đầu script

## Các Mẫu An toàn & Bảo mật

- Khai báo hằng số với `readonly` để ngăn chặn sửa đổi tình cờ
- Sử dụng từ khóa `local` cho tất cả các biến hàm để tránh làm ô nhiễm phạm vi toàn cục
- Triển khai `timeout` cho các lệnh bên ngoài: `timeout 30s curl ...` ngăn chặn treo
- Xác thực quyền tệp trước khi thao tác: `[[ -r "$file" ]] || exit 1`
- Sử dụng thay thế tiến trình `<(command)` thay vì tệp tạm thời khi có thể
- Làm sạch đầu vào người dùng trước khi sử dụng trong các lệnh hoặc thao tác tệp
- Xác thực đầu vào số với khớp mẫu: `[[ $num =~ ^[0-9]+$ ]]`
- Không bao giờ sử dụng `eval` trên đầu vào người dùng; sử dụng mảng để xây dựng lệnh động
- Đặt umask hạn chế cho các hoạt động nhạy cảm: `(umask 077; touch "$secure_file")`
- Ghi log các hoạt động liên quan đến bảo mật (xác thực, thay đổi đặc quyền, truy cập tệp)
- Sử dụng `--` để tách tùy chọn khỏi đối số: `rm -rf -- "$user_input"`
- Xác thực biến môi trường trước khi sử dụng: `: "${REQUIRED_VAR:?not set}"`
- Kiểm tra mã thoát của tất cả các hoạt động quan trọng về bảo mật một cách rõ ràng
- Sử dụng `trap` để đảm bảo dọn dẹp xảy ra ngay cả khi thoát bất thường

## Tối ưu hóa Hiệu năng

- Tránh subshells trong vòng lặp; sử dụng `while read` thay vì `for i in $(cat file)`
- Sử dụng Bash built-ins hơn các lệnh bên ngoài: `[[ ]]` thay vì `test`, `${var//pattern/replacement}` thay vì `sed`
- Gom nhóm các hoạt động thay vì lặp lại các hoạt động đơn lẻ (ví dụ, một `sed` với nhiều biểu thức)
- Sử dụng `mapfile`/`readarray` để điền mảng hiệu quả từ đầu ra lệnh
- Tránh thay thế lệnh lặp lại; lưu kết quả vào biến một lần
- Sử dụng mở rộng số học `$(( ))` thay vì `expr` để tính toán
- Ưu tiên `printf` hơn `echo` cho đầu ra được định dạng (nhanh hơn và đáng tin cậy hơn)
- Sử dụng mảng kết hợp (associative arrays) để tra cứu thay vì grep lặp lại
- Xử lý tệp từng dòng cho các tệp lớn thay vì tải toàn bộ tệp vào bộ nhớ
- Sử dụng `xargs -P` để xử lý song song khi các hoạt động độc lập

## Tiêu chuẩn Tài liệu

- Triển khai cờ `--help` và `-h` hiển thị cách sử dụng, tùy chọn và ví dụ
- Cung cấp cờ `--version` hiển thị phiên bản script và thông tin bản quyền
- Bao gồm các ví dụ sử dụng trong đầu ra trợ giúp cho các trường hợp sử dụng phổ biến
- Tài liệu hóa tất cả các tùy chọn dòng lệnh với mô tả về mục đích của chúng
- Liệt kê các đối số bắt buộc vs tùy chọn rõ ràng trong thông báo sử dụng
- Tài liệu hóa mã thoát: 0 cho thành công, 1 cho lỗi chung, mã cụ thể cho các lỗi cụ thể
- Bao gồm phần điều kiện tiên quyết liệt kê các lệnh và phiên bản cần thiết
- Thêm khối chú thích tiêu đề với mục đích script, tác giả, và ngày sửa đổi
- Tài liệu hóa các biến môi trường mà script sử dụng hoặc yêu cầu
- Cung cấp phần khắc phục sự cố trong trợ giúp cho các vấn đề phổ biến
- Tạo tài liệu với `shdoc` từ các định dạng chú thích đặc biệt
- Tạo trang man sử dụng `shellman` cho tích hợp hệ thống
- Bao gồm sơ đồ kiến trúc sử dụng Mermaid hoặc GraphViz cho các script phức tạp

## Các Tính năng Bash Hiện đại (5.x)

- **Bash 5.0**: Cải tiến mảng kết hợp, chuyển đổi chữ hoa `${var@U}`, chữ thường `${var@L}`
- **Bash 5.1**: Nâng cao các biến đổi `${parameter@operator}`, tùy chọn shopt `compat` cho khả năng tương thích
- **Bash 5.2**: Tùy chọn `varredir_close`, cải thiện xử lý lỗi `exec`, độ chính xác micro giây `EPOCHREALTIME`
- Kiểm tra phiên bản trước khi sử dụng tính năng hiện đại: `[[ ${BASH_VERSINFO[0]} -ge 5 && ${BASH_VERSINFO[1]} -ge 2 ]]`
- Sử dụng `${parameter@Q}` cho đầu ra được trích dẫn shell (Bash 4.4+)
- Sử dụng `${parameter@E}` cho mở rộng chuỗi thoát (Bash 4.4+)
- Sử dụng `${parameter@P}` cho mở rộng prompt (Bash 4.4+)
- Sử dụng `${parameter@A}` cho định dạng gán (Bash 4.4+)
- Sử dụng `wait -n` để đợi bất kỳ công việc nền nào (Bash 4.3+)
- Sử dụng `mapfile -d delim` cho các dấu phân cách tùy chỉnh (Bash 4.4+)

## Tích hợp CI/CD

- **GitHub Actions**: Sử dụng `shellcheck-problem-matchers` cho chú thích nội dòng
- **Pre-commit hooks**: Cấu hình `.pre-commit-config.yaml` với `shellcheck`, `shfmt`, `checkbashisms`
- **Matrix testing**: Kiểm thử trên Bash 4.4, 5.0, 5.1, 5.2 trên Linux và macOS
- **Container testing**: Sử dụng hình ảnh Docker chính thức bash:5.2 cho các kiểm thử có thể tái tạo
- **CodeQL**: Bật quét script shell cho các lỗ hổng bảo mật
- **Actionlint**: Xác thực các tệp workflow GitHub Actions sử dụng shell scripts
- **Automated releases**: Gắn thẻ phiên bản và tạo changelogs tự động
- **Coverage reporting**: Theo dõi độ bao phủ kiểm thử và fail khi có hồi quy
- Ví dụ quy trình: `shellcheck *.sh && shfmt -d *.sh && bats test/`

## Quét Bảo mật & Củng cố (Hardening)

- **SAST**: Tích hợp Semgrep với các quy tắc tùy chỉnh cho các lỗ hổng đặc thù của shell
- **Secrets detection**: Sử dụng `gitleaks` hoặc `trufflehog` để ngăn chặn rò rỉ thông tin xác thực
- **Supply chain**: Xác minh checksum của các script bên ngoài được source
- **Sandboxing**: Chạy các script không đáng tin cậy trong container với đặc quyền hạn chế
- **SBOM**: Tài liệu hóa các phụ thuộc và công cụ bên ngoài để tuân thủ
- **Security linting**: Sử dụng ShellCheck với các quy tắc tập trung vào bảo mật được bật
- **Privilege analysis**: Kiểm toán script cho các yêu cầu root/sudo không cần thiết
- **Input sanitization**: Xác thực tất cả đầu vào bên ngoài so với danh sách cho phép (allowlists)
- **Audit logging**: Ghi log tất cả các hoạt động liên quan đến bảo mật vào syslog
- **Container security**: Quét môi trường thực thi script cho các lỗ hổng

## Observability & Logging

- **Structured logging**: Xuất JSON cho các hệ thống tổng hợp log
- **Log levels**: Triển khai DEBUG, INFO, WARN, ERROR với độ chi tiết có thể cấu hình
- **Syslog integration**: Sử dụng lệnh `logger` cho tích hợp log hệ thống
- **Distributed tracing**: Thêm trace IDs để tương quan quy trình làm việc đa script
- **Metrics export**: Xuất metrics định dạng Prometheus cho giám sát
- **Error context**: Bao gồm stack traces, thông tin môi trường trong log lỗi
- **Log rotation**: Cấu hình xoay vòng log cho các script chạy lâu dài
- **Performance metrics**: Theo dõi thời gian thực thi, sử dụng tài nguyên, độ trễ gọi bên ngoài
- Ví dụ: `log_info() { logger -t "$SCRIPT_NAME" -p user.info "$*"; echo "[INFO] $*" >&2; }`

## Danh sách Kiểm tra Chất lượng

- Script vượt qua phân tích tĩnh ShellCheck với tối thiểu suppressions
- Mã được định dạng nhất quán với shfmt sử dụng các tùy chọn tiêu chuẩn
- Độ bao phủ kiểm thử toàn diện với Bats bao gồm các trường hợp biên
- Tất cả các mở rộng biến đều được trích dẫn đúng cách
- Xử lý lỗi bao gồm tất cả các chế độ thất bại với thông báo có ý nghĩa
- Tài nguyên tạm thời được dọn dẹp đúng cách với EXIT traps
- Script hỗ trợ `--help` và cung cấp thông tin sử dụng rõ ràng
- Xác thực đầu vào ngăn chặn các cuộc tấn công injection và xử lý các trường hợp biên
- Script di động trên các nền tảng mục tiêu (Linux, macOS)
- Hiệu năng phù hợp cho khối lượng công việc và kích thước dữ liệu dự kiến

## Đầu ra

- Các script Bash sẵn sàng cho production với các thực hành lập trình phòng thủ
- Các bộ kiểm thử toàn diện sử dụng bats-core hoặc shellspec với đầu ra TAP
- Cấu hình đường ống CI/CD (GitHub Actions, GitLab CI) cho kiểm thử tự động
- Tài liệu được tạo với shdoc và trang man với shellman
- Bố cục dự án có cấu trúc với các hàm thư viện có thể tái sử dụng và quản lý phụ thuộc
- Các tệp cấu hình phân tích tĩnh (.shellcheckrc, .shfmt.toml, .editorconfig)
- Các điểm chuẩn hiệu năng và báo cáo profiling cho các quy trình quan trọng
- Đánh giá bảo mật với SAST, quét bí mật, và báo cáo lỗ hổng
- Các tiện ích gỡ lỗi với chế độ trace, logging có cấu trúc, và observability
- Hướng dẫn di chuyển cho nâng cấp Bash 3→5 và hiện đại hóa legacy
- Cấu hình phân phối gói (công thức Homebrew, thông số deb/rpm)
- Hình ảnh Container cho môi trường thực thi có thể tái tạo

## Các Công cụ Thiết yếu

### Phân tích Tĩnh & Định dạng

- **ShellCheck**: Công cụ phân tích tĩnh với cấu hình `enable=all` và `external-sources=true`
- **shfmt**: Định dạng shell script với cấu hình tiêu chuẩn (`-i 2 -ci -bn -sr -kp`)
- **checkbashisms**: Phát hiện các cấu trúc đặc thù của bash cho phân tích khả năng di động
- **Semgrep**: SAST với các quy tắc tùy chỉnh cho các vấn đề bảo mật đặc thù của shell
- **CodeQL**: Quét bảo mật của GitHub cho shell scripts

### Framework Kiểm thử

- **bats-core**: Bản fork được bảo trì của Bats với các tính năng hiện đại và phát triển tích cực
- **shellspec**: Framework kiểm thử kiểu BDD với các khẳng định phong phú và mocking
- **shunit2**: Framework kiểm thử kiểu xUnit cho shell scripts
- **bashing**: Framework kiểm thử với hỗ trợ mocking và cô lập kiểm thử

### Công cụ Phát triển Hiện đại

- **bashly**: Trình tạo framework CLI để xây dựng các ứng dụng dòng lệnh
- **basher**: Trình quản lý gói Bash cho quản lý phụ thuộc
- **bpkg**: Trình quản lý gói bash thay thế với giao diện giống npm
- **shdoc**: Tạo tài liệu markdown từ chú thích shell script
- **shellman**: Tạo trang man từ shell scripts

### CI/CD & Tự động hóa

- **pre-commit**: Framework pre-commit hook đa ngôn ngữ
- **actionlint**: Trình linter cho GitHub Actions workflow
- **gitleaks**: Quét bí mật để ngăn chặn rò rỉ thông tin xác thực
- **Makefile**: Tự động hóa cho các quy trình lint, format, test, và release

## Các Cạm bẫy Phổ biến cần Tránh

- `for f in $(ls ...)` gây ra lỗi tách từ/globbing (sử dụng `find -print0 | while IFS= read -r -d '' f; do ...; done`)
- Mở rộng biến không được trích dẫn dẫn đến hành vi không mong muốn
- Dựa vào `set -e` mà không có bẫy lỗi thích hợp trong các luồng phức tạp
- Sử dụng `echo` cho đầu ra dữ liệu (ưu tiên `printf` để tin cậy)
- Thiếu bẫy dọn dẹp cho các tệp và thư mục tạm thời
- Điền mảng không an toàn (sử dụng `readarray`/`mapfile` thay vì thay thế lệnh)
- Bỏ qua xử lý tệp an toàn nhị phân (luôn xem xét dấu phân cách NUL cho tên tệp)

## Quản lý Phụ thuộc

- **Trình quản lý gói**: Sử dụng `basher` hoặc `bpkg` để cài đặt các phụ thuộc shell script
- **Vendoring**: Sao chép các phụ thuộc vào dự án cho các bản build có thể tái tạo
- **Lock files**: Tài liệu hóa phiên bản chính xác của các phụ thuộc được sử dụng
- **Xác minh Checksum**: Xác minh tính toàn vẹn của các script bên ngoài được source
- **Phiên bản ghim (Version pinning)**: Khóa các phụ thuộc vào các phiên bản cụ thể để ngăn chặn thay đổi phá vỡ
- **Cô lập phụ thuộc**: Sử dụng các thư mục riêng biệt cho các bộ phụ thuộc khác nhau
- **Tự động cập nhật**: Tự động hóa cập nhật phụ thuộc với Dependabot hoặc Renovate
- **Quét bảo mật**: Quét các phụ thuộc cho các lỗ hổng đã biết
- Ví dụ: `basher install username/repo@version` hoặc `bpkg install username/repo -g`

## Kỹ thuật Nâng cao

- **Ngữ cảnh Lỗi**: Sử dụng `trap 'echo "Error at line $LINENO: exit $?" >&2' ERR` để gỡ lỗi
- **Xử lý Temp An toàn**: `trap 'rm -rf "$tmpdir"' EXIT; tmpdir=$(mktemp -d)`
- **Kiểm tra Phiên bản**: `(( BASH_VERSINFO[0] >= 5 ))` trước khi sử dụng các tính năng hiện đại
- **Mảng An toàn Nhị phân**: `readarray -d '' files < <(find . -print0)`
- **Trả về Hàm**: Sử dụng `declare -g result` để trả về dữ liệu phức tạp từ hàm
- **Mảng Kết hợp**: `declare -A config=([host]="localhost" [port]="8080")` cho cấu trúc dữ liệu phức tạp
- **Mở rộng Tham số**: `${filename%.sh}` xóa đuôi mở rộng, `${path##*/}` tên cơ sở (basename), `${text//old/new}` thay thế tất cả
- **Xử lý Tín hiệu**: `trap cleanup_function SIGHUP SIGINT SIGTERM` cho tắt máy duyên dáng (graceful shutdown)
- **Nhóm Lệnh**: `{ cmd1; cmd2; } > output.log` chia sẻ chuyển hướng, `( cd dir && cmd )` sử dụng subshell để cô lập
- **Đồng tiến trình (Co-processes)**: `coproc proc { cmd; }; echo "data" >&"${proc[1]}"; read -u "${proc[0]}" result` cho các pipe hai chiều
- **Here-documents**: `cat <<-'EOF'` với `-` loại bỏ tab đầu dòng, dấu ngoặc kép ngăn chặn mở rộng
- **Quản lý Tiến trình**: `wait $pid` để đợi công việc nền, `jobs -p` liệt kê PID nền
- **Thực thi Có điều kiện**: `cmd1 && cmd2` chỉ chạy cmd2 nếu cmd1 thành công, `cmd1 || cmd2` chạy cmd2 nếu cmd1 thất bại
- **Mở rộng Ngoặc nhọn**: `touch file{1..10}.txt` tạo nhiều tệp hiệu quả
- **Biến Nameref**: `declare -n ref=varname` tạo tham chiếu đến biến khác (Bash 4.3+)
- **Bẫy Lỗi Cải tiến**: `set -Eeuo pipefail; shopt -s inherit_errexit` cho xử lý lỗi toàn diện
- **Thực thi Song song**: `xargs -P $(nproc) -n 1 command` cho xử lý song song với số lượng lõi CPU
- **Đầu ra Có cấu trúc**: `jq -n --arg key "$value" '{key: $key}'` cho tạo JSON
- **Profiling Hiệu năng**: Sử dụng `time -v` cho sử dụng tài nguyên chi tiết hoặc `TIMEFORMAT` cho thời gian tùy chỉnh

## Tài liệu tham khảo & Đọc thêm

### Hướng dẫn Phong cách & Thực hành Tốt nhất

- [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html) - Hướng dẫn phong cách toàn diện bao gồm trích dẫn, mảng, và khi nào nên sử dụng shell
- [Bash Pitfalls](https://mywiki.wooledge.org/BashPitfalls) - Danh mục các lỗi Bash phổ biến và cách tránh chúng
- [Bash Hackers Wiki](https://wiki.bash-hackers.org/) - Tài liệu Bash toàn diện và các kỹ thuật nâng cao
- [Defensive BASH Programming](https://www.kfirlavi.com/blog/2012/11/14/defensive-bash-programming/) - Các mẫu lập trình phòng thủ hiện đại

### Công cụ & Framework

- [ShellCheck](https://github.com/koalaman/shellcheck) - Công cụ phân tích tĩnh và tài liệu wiki phong phú
- [shfmt](https://github.com/mvdan/sh) - Định dạng shell script với tài liệu cờ chi tiết
- [bats-core](https://github.com/bats-core/bats-core) - Framework kiểm thử Bash được bảo trì
- [shellspec](https://github.com/shellspec/shellspec) - Framework kiểm thử kiểu BDD cho shell scripts
- [bashly](https://bashly.dannyb.co/) - Trình tạo framework Bash CLI hiện đại
- [shdoc](https://github.com/reconquest/shdoc) - Trình tạo tài liệu cho shell scripts

### Bảo mật & Chủ đề Nâng cao

- [Bash Security Best Practices](https://github.com/carlospolop/PEASS-ng) - Các mẫu shell script tập trung vào bảo mật
- [Awesome Bash](https://github.com/awesome-lists/awesome-bash) - Danh sách tài nguyên và công cụ Bash được tuyển chọn
- [Pure Bash Bible](https://github.com/dylanaraps/pure-bash-bible) - Bộ sưu tập các lựa chọn thay thế thuần bash cho các lệnh bên ngoài
