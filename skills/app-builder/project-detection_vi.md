# Phát hiện Loại Dự án

> Phân tích yêu cầu của người dùng để xác định loại dự án và mẫu.

## Ma trận Từ khóa

| Từ khóa                            | Loại Dự án                                 | Mẫu                |
| ---------------------------------- | ------------------------------------------ | ------------------ |
| blog, post, article                | Blog                                       | astro-static       |
| e-commerce, product, cart, payment | Thương mại điện tử (E-commerce)            | nextjs-saas        |
| dashboard, panel, management       | Bảng điều khiển Quản trị (Admin Dashboard) | nextjs-fullstack   |
| api, backend, service, rest        | Dịch vụ API                                | express-api        |
| python, fastapi, django            | Python API                                 | python-fastapi     |
| mobile, android, ios, react native | Ứng dụng Di động (RN)                      | react-native-app   |
| flutter, dart                      | Ứng dụng Di động (Flutter)                 | flutter-app        |
| portfolio, personal, cv            | Portfolio (Hồ sơ năng lực)                 | nextjs-static      |
| crm, customer, sales               | CRM                                        | nextjs-fullstack   |
| saas, subscription, stripe         | SaaS                                       | nextjs-saas        |
| landing, promotional, marketing    | Landing Page                               | nextjs-static      |
| docs, documentation                | Tài liệu                                   | astro-static       |
| extension, plugin, chrome          | Tiện ích mở rộng Trình duyệt               | chrome-extension   |
| desktop, electron                  | Ứng dụng Máy tính để bàn                   | electron-desktop   |
| cli, command line, terminal        | Công cụ CLI                                | cli-tool           |
| monorepo, workspace                | Monorepo                                   | monorepo-turborepo |

## Quy trình Phát hiện

```
1. Phân tách (Tokenize) yêu cầu người dùng
2. Trích xuất từ khóa
3. Xác định loại dự án
4. Phát hiện thông tin thiếu → chuyển tiếp đến conversation-manager
5. Đề xuất tech stack
```
