---
name: deep-research
description: "Thực hiện nghiên cứu đa bước tự chủ sử dụng Google Gemini Deep Research Agent. Sử dụng cho: phân tích thị trường, cảnh quan cạnh tranh, đánh giá tài liệu, nghiên cứu kỹ thuật, thẩm định. Mất 2-10 phút nhưng tạo ra các báo cáo chi tiết, có trích dẫn. Chi phí $2-5 mỗi tác vụ."
source: "https://github.com/sanjay3290/ai-skills/tree/main/skills/deep-research"
risk: safe
---

# Kỹ Năng Nghiên Cứu Chuyên Sâu Gemini (Gemini Deep Research Skill)

Chạy các tác vụ nghiên cứu tự chủ để lập kế hoạch, tìm kiếm, đọc và tổng hợp thông tin thành các báo cáo toàn diện.

## Khi nào Sử dụng Kỹ năng này

Sử dụng kỹ năng này khi:

- Thực hiện phân tích thị trường
- Tiến hành phân tích cảnh quan cạnh tranh
- Tạo đánh giá tài liệu
- Thực hiện nghiên cứu kỹ thuật
- Thực hiện thẩm định (due diligence)
- Cần các báo cáo nghiên cứu chi tiết, có trích dẫn

## Yêu cầu

- Python 3.8+
- httpx: `pip install -r requirements.txt`
- Biến môi trường GEMINI_API_KEY

## Thiết lập

1. Lấy khóa API Gemini từ [Google AI Studio](https://aistudio.google.com/)
2. Thiết lập biến môi trường:
   ```bash
   export GEMINI_API_KEY=your-api-key-here
   ```
   Hoặc tạo tệp `.env` trong thư mục kỹ năng.

## Sử dụng

### Bắt đầu một tác vụ nghiên cứu

```bash
python3 scripts/research.py --query "Research the history of Kubernetes"
```

### Với định dạng đầu ra có cấu trúc

```bash
python3 scripts/research.py --query "Compare Python web frameworks" \
  --format "1. Executive Summary\n2. Comparison Table\n3. Recommendations"
```

### Truyền phát tiến độ theo thời gian thực

```bash
python3 scripts/research.py --query "Analyze EV battery market" --stream
```

### Bắt đầu mà không cần chờ đợi

```bash
python3 scripts/research.py --query "Research topic" --no-wait
```

### Kiểm tra trạng thái của nghiên cứu đang chạy

```bash
python3 scripts/research.py --status <interaction_id>
```

### Chờ hoàn thành

```bash
python3 scripts/research.py --wait <interaction_id>
```

### Tiếp tục từ nghiên cứu trước đó

```bash
python3 scripts/research.py --query "Elaborate on point 2" --continue <interaction_id>
```

### Liệt kê nghiên cứu gần đây

```bash
python3 scripts/research.py --list
```

## Định dạng Đầu ra

- **Mặc định**: Báo cáo markdown dễ đọc cho người
- **JSON** (`--json`): Dữ liệu có cấu trúc cho sử dụng lập trình
- **Thô** (`--raw`): Phản hồi API chưa xử lý

## Chi phí & Thời gian

| Chỉ số        | Giá trị                                     |
| ------------- | ------------------------------------------- |
| Thời gian     | 2-10 phút mỗi tác vụ                        |
| Chi phí       | $2-5 mỗi tác vụ (thay đổi theo độ phức tạp) |
| Sử dụng Token | ~250k-900k đầu vào, ~60k-80k đầu ra         |

## Các Trường hợp Sử dụng Tốt nhất

- Phân tích thị trường và cảnh quan cạnh tranh
- Đánh giá tài liệu kỹ thuật
- Nghiên cứu thẩm định
- Nghiên cứu lịch sử và mốc thời gian
- Phân tích so sánh (khung, sản phẩm, công nghệ)

## Quy trình làm việc

1. Người dùng yêu cầu nghiên cứu → Chạy `--query "..."`
2. Thông báo cho người dùng về thời gian ước tính (2-10 phút)
3. Giám sát với `--stream` hoặc thăm dò với `--status`
4. Trả về kết quả đã định dạng
5. Sử dụng `--continue` cho các câu hỏi tiếp theo

## Mã Thoát

- **0**: Thành công
- **1**: Lỗi (lỗi API, vấn đề cấu hình, hết thời gian chờ)
- **130**: Đã hủy bởi người dùng (Ctrl+C)
