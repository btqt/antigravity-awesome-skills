---
name: brand-guidelines
description: Áp dụng màu sắc và kiểu chữ thương hiệu chính thức của Anthropic cho bất kỳ loại tạo tác nào có thể hưởng lợi từ việc có giao diện và cảm nhận của Anthropic. Sử dụng khi màu sắc thương hiệu hoặc hướng dẫn phong cách, định dạng trực quan, hoặc tiêu chuẩn thiết kế công ty được áp dụng.
license: Các điều khoản đầy đủ trong LICENSE.txt
---

# Phong cách Thương hiệu Anthropic

## Tổng quan

Để truy cập danh tính thương hiệu và tài nguyên phong cách chính thức của Anthropic, hãy sử dụng kỹ năng này.

**Từ khóa**: thương hiệu (branding), danh tính doanh nghiệp, danh tính trực quan, hậu xử lý, tạo kiểu, màu sắc thương hiệu, kiểu chữ, thương hiệu Anthropic, định dạng trực quan, thiết kế trực quan

## Hướng dẫn Thương hiệu

### Màu sắc

**Màu Chính:**

- Dark: `#141413` - Văn bản chính và nền tối
- Light: `#faf9f5` - Nền sáng và văn bản trên nền tối
- Mid Gray: `#b0aea5` - Các yếu tố phụ
- Light Gray: `#e8e6dc` - Nền tinh tế

**Màu Nhấn:**

- Orange: `#d97757` - Nhấn chính
- Blue: `#6a9bcc` - Nhấn phụ
- Green: `#788c5d` - Nhấn thứ ba

### Kiểu chữ

- **Tiêu đề**: Poppins (với fallback Arial)
- **Văn bản Thân**: Lora (với fallback Georgia)
- **Lưu ý**: Phông chữ nên được cài đặt sẵn trong môi trường của bạn để có kết quả tốt nhất

## Tính năng

### Áp dụng Phông chữ Thông minh

- Áp dụng phông chữ Poppins cho các tiêu đề (24pt và lớn hơn)
- Áp dụng phông chữ Lora cho văn bản thân
- Tự động quay về Arial/Georgia nếu phông chữ tùy chỉnh không khả dụng
- Bảo tồn khả năng đọc trên tất cả các hệ thống

### Tạo kiểu Văn bản

- Tiêu đề (24pt+): Phông chữ Poppins
- Văn bản thân: Phông chữ Lora
- Lựa chọn màu thông minh dựa trên nền
- Bảo tồn phân cấp văn bản và định dạng

### Hình dạng và Màu Nhấn

- Các hình dạng không phải văn bản sử dụng màu nhấn
- Xoay vòng qua các màu nhấn cam, xanh dương, và xanh lá cây
- Duy trì sự thú vị trực quan trong khi vẫn giữ đúng thương hiệu

## Chi tiết Kỹ thuật

### Quản lý Phông chữ

- Sử dụng phông chữ Poppins và Lora được cài đặt trong hệ thống khi khả dụng
- Cung cấp fallback tự động sang Arial (tiêu đề) và Georgia (thân)
- Không yêu cầu cài đặt phông chữ - hoạt động với phông chữ hệ thống hiện có
- Để có kết quả tốt nhất, hãy cài đặt trước phông chữ Poppins và Lora trong môi trường của bạn

### Áp dụng Màu sắc

- Sử dụng giá trị màu RGB để khớp chính xác thương hiệu
- Được áp dụng thông qua lớp RGBColor của python-pptx
- Duy trì độ trung thực màu sắc trên các hệ thống khác nhau
