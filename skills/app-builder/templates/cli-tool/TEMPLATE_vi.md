---
name: cli-tool
description: Nguyên tắc mẫu công cụ CLI Node.js. Commander.js, các lời nhắc tương tác.
---

# Mẫu Công cụ CLI

## Tech Stack

| Thành phần         | Công nghệ    |
| ------------------ | ------------ |
| Runtime            | Node.js 20+  |
| Ngôn ngữ           | TypeScript   |
| CLI Framework      | Commander.js |
| Lời nhắc (Prompts) | Inquirer.js  |
| Đầu ra (Output)    | chalk + ora  |
| Cấu hình           | cosmiconfig  |

---

## Cấu trúc Thư mục

```
project-name/
├── src/
│   ├── index.ts         # Điểm vào (Entry point)
│   ├── cli.ts           # Thiết lập CLI
│   ├── commands/        # Trình xử lý lệnh
│   ├── lib/
│   │   ├── config.ts    # Config loader
│   │   └── logger.ts    # Styled output
│   └── types/
├── bin/
│   └── cli.js           # Executable
└── package.json
```

---

## Các Nguyên tắc Thiết kế CLI

| Nguyên tắc             | Mô tả                           |
| ---------------------- | ------------------------------- |
| Lệnh con (Subcommands) | Nhóm các hành động liên quan    |
| Tùy chọn (Options)     | Cờ (Flags) với giá trị mặc định |
| Tương tác              | Lời nhắc khi cần thiết          |
| Không tương tác        | Hỗ trợ cờ --yes                 |

---

## Các Thành phần Chính

| Thành phần  | Mục đích                              |
| ----------- | ------------------------------------- |
| Commander   | Phân tích lệnh                        |
| Inquirer    | Lời nhắc tương tác                    |
| Chalk       | Đầu ra có màu sắc                     |
| Ora         | Vòng quay/đang tải (Spinners/loading) |
| Cosmiconfig | Khám phá tệp cấu hình                 |

---

## Các bước Thiết lập

1. Tạo thư mục dự án
2. `npm init -y`
3. Cài đặt các phụ thuộc: `npm install commander @inquirer/prompts chalk ora cosmiconfig`
4. Cấu hình bin trong package.json
5. `npm link` để kiểm thử cục bộ

---

## Xuất bản

```bash
npm login
npm publish
```

---

## Thực hành Tốt nhất

- Cung cấp thông báo lỗi hữu ích
- Hỗ trợ cả hai chế độ tương tác và không tương tác
- Sử dụng kiểu đầu ra nhất quán
- Xác thực đầu vào với Zod
- Thoát với mã phù hợp (0 thành công, 1 lỗi)
