---
name: bats-testing-patterns
description: Nắm vững Hệ thống Kiểm thử Tự động Bash (Bats) để kiểm thử shell script toàn diện. Sử dụng khi viết kiểm thử cho shell scripts, đường ống CI/CD, hoặc yêu cầu phát triển hướng kiểm thử (TDD) cho các tiện ích shell.
---

# Các Mẫu Kiểm thử Bats (Bats Testing Patterns)

Hướng dẫn toàn diện để viết các kiểm thử đơn vị (unit tests) toàn diện cho shell scripts sử dụng Bats (Bash Automated Testing System), bao gồm các mẫu kiểm thử, fixtures, và thực hành tốt nhất cho kiểm thử shell cấp production.

## Sử dụng kỹ năng này khi

- Viết unit tests cho shell scripts
- Triển khai TDD cho scripts
- Thiết lập kiểm thử tự động trong đường ống CI/CD
- Kiểm thử các trường hợp biên và điều kiện lỗi
- Xác thực hành vi trên các môi trường shell

## Không sử dụng kỹ năng này khi

- Dự án không sử dụng shell scripts
- Bạn cần kiểm thử tích hợp (integration tests) vượt quá hành vi của shell
- Mục tiêu chỉ là linting hoặc định dạng (formatting)

## Hướng dẫn

- Xác nhận các phương ngữ shell (shell dialects) và môi trường được hỗ trợ.
- Thiết lập cấu trúc kiểm thử với helpers và fixtures.
- Viết kiểm thử cho mã thoát (exit codes), đầu ra, và tác dụng phụ (side effects).
- Thêm thiết lập/dọn dẹp (setup/teardown) và chạy kiểm thử trong CI.
- Nếu cần ví dụ chi tiết, mở `resources/implementation-playbook.md`.

## Tài nguyên

- `resources/implementation-playbook.md` cho các mẫu và ví dụ chi tiết.
