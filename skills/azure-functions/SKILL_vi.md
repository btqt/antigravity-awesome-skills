---
name: azure-functions
description: "Các mẫu chuyên gia cho phát triển Azure Functions bao gồm mô hình isolated worker, điều phối Durable Functions, tối ưu hóa khởi động lạnh (cold start), và các mẫu sản xuất. Bao gồm các mô hình lập trình .NET, Python, và Node.js. Sử dụng khi: azure function, azure functions, durable functions, azure serverless, function app."
source: vibeship-spawner-skills (Apache 2.0)
---

# Azure Functions

## Các Mẫu

### Isolated Worker Model (.NET)

Mô hình thực thi .NET hiện đại với sự cô lập quy trình (process isolation).

### Node.js v4 Programming Model

Cách tiếp cận tập trung vào mã (code-centric) hiện đại cho TypeScript/JavaScript.

### Python v2 Programming Model

Cách tiếp cận dựa trên decorator cho các hàm Python.

## Anti-Patterns (Nên Tránh)

### ❌ Blocking Async Calls (Chặn các cuộc gọi Async)

### ❌ New HttpClient Per Request (Tạo HttpClient mới cho mỗi yêu cầu)

### ❌ In-Process Model for New Projects (Mô hình In-Process cho dự án mới)

## ⚠️ Sharp Edges (Các Vấn đề Cần Lưu ý)

| Vấn đề | Mức độ nghiêm trọng | Giải pháp                                      |
| ------ | ------------------- | ---------------------------------------------- |
| Issue  | high                | ## Sử dụng mẫu async với Durable Functions     |
| Issue  | high                | ## Sử dụng IHttpClientFactory (Khuyên dùng)    |
| Issue  | high                | ## Luôn sử dụng async/await                    |
| Issue  | medium              | ## Cấu hình thời gian chờ tối đa (Consumption) |
| Issue  | high                | ## Sử dụng isolated worker cho các dự án mới   |
| Issue  | medium              | ## Cấu hình Application Insights đúng cách     |
| Issue  | medium              | ## Kiểm tra extension bundle (phổ biến nhất)   |
| Issue  | medium              | ## Thêm warmup trigger để khởi tạo mã của bạn  |
