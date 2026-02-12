---
name: daily-news-report
description: Thu thập nội dung dựa trên danh sách URL đặt trước, lọc thông tin kỹ thuật chất lượng cao và tạo báo cáo Markdown hàng ngày.
argument-hint: [optional: date]
disable-model-invocation: false
user-invocable: true
allowed-tools: Task, WebFetch, Read, Write, Bash(mkdir*), Bash(date*), Bash(ls*), mcp__chrome-devtools__*
---

# Báo Cáo Tin Tức Hàng Ngày v3.0 (Daily News Report v3.0)

> **Nâng cấp Kiến trúc**: Điều phối Tác nhân Chính + Thực thi Tác nhân Phụ + Thu thập bằng Trình duyệt + Bộ nhớ đệm Thông minh

## Kiến trúc Cốt lõi

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Tác nhân Chính (Người điều phối)             │
│  Vai trò: Lập lịch, Giám sát, Đánh giá, Quyết định, Tổng hợp        │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│   │ 1. Khởi tạo │ → │ 2. Phân phối│ → │ 3. Giám sát │ → │ 4. Đánh giá │     │
│   │ Đọc Cấu hình│    │ Giao Nhiệm vụ│    │ Thu thập KQ │    │ Lọc/Sắp xếp │     │
│   └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘     │
│         │                  │                  │                  │           │
│         ▼                  ▼                  ▼                  ▼           │
│   ┌─────────────┐    ┌─────────────┐    ┌─────────────┐    ┌─────────────┐     │
│   │ 5. Quyết định│ ← │ Đủ 20?      │    │ 6. Tạo      │ → │ 7. Cập nhật │     │
│   │ Tiếp/Dừng   │    │ Y/N         │    │ File Báo cáo│    │ Cache Stats │     │
│   └─────────────┘    └─────────────┘    └─────────────┘    └─────────────┘     │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
         ↓ Phân phối                          ↑ Trả về Kết quả
┌─────────────────────────────────────────────────────────────────────┐
│                        Lớp Thực thi Tác nhân Phụ                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   ┌─────────────┐   ┌─────────────┐   ┌─────────────┐              │
│   │ Worker A    │   │ Worker B    │   │ Browser     │              │
│   │ (WebFetch)  │   │ (WebFetch)  │   │ (Headless)  │              │
│   │ Tier1 Batch │   │ Tier2 Batch │   │ JS Render   │              │
│   └─────────────┘   └─────────────┘   └─────────────┘              │
│         ↓                 ↓                 ↓                        │
│   ┌─────────────────────────────────────────────────────────────┐   │
│   │                    Trả về Kết quả Cấu trúc                  │   │
│   │  { status, data: [...], errors: [...], metadata: {...} }    │   │
│   └─────────────────────────────────────────────────────────────┘   │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

## Tệp Cấu hình

Kỹ năng này sử dụng các tệp cấu hình sau:

| Tệp            | Mục đích                                                                         |
| -------------- | -------------------------------------------------------------------------------- |
| `sources.json` | Cấu hình nguồn, độ ưu tiên, phương pháp thu thập                                 |
| `cache.json`   | Dữ liệu đã lưu trong bộ nhớ đệm, thống kê lịch sử, dấu vân tay loại bỏ trùng lặp |

## Chi tiết Quy trình Thực thi

### Giai đoạn 1: Khởi tạo

```yaml
Các bước: 1. Xác định ngày (đối số người dùng hoặc ngày hiện tại)
  2. Đọc sources.json cho cấu hình nguồn
  3. Đọc cache.json cho dữ liệu lịch sử
  4. Tạo thư mục đầu ra NewsReport/
  5. Kiểm tra xem báo cáo một phần có tồn tại cho hôm nay không (chế độ nối thêm)
```

### Giai đoạn 2: Phân phối Tác nhân Phụ

**Chiến lược**: Phân phối song song, thực thi hàng loạt, cơ chế dừng sớm

```yaml
Làn sóng 1 (Song song):
  - Worker A: Tier1 Batch A (HN, HuggingFace Papers)
  - Worker B: Tier1 Batch B (OneUsefulThing, Paul Graham)

Chờ kết quả → Đánh giá số lượng

Nếu < 15 mục chất lượng cao:
  Làn sóng 2 (Song song):
    - Worker C: Tier2 Batch A (James Clear, FS Blog)
    - Worker D: Tier2 Batch B (HackerNoon, Scott Young)

Nếu vẫn < 20 mục:
  Làn sóng 3 (Trình duyệt):
    - Browser Worker: ProductHunt, Latent Space (Yêu cầu hiển thị JS)
```

### Giai đoạn 3: Định dạng Nhiệm vụ Tác nhân Phụ

Định dạng nhiệm vụ nhận được bởi mỗi Tác nhân Phụ:

```yaml
task: fetch_and_extract
sources:
  - id: hn
    url: https://news.ycombinator.com
    extract: top_10
  - id: hf_papers
    url: https://huggingface.co/papers
    extract: top_voted

output_schema:
  items:
    - source_id: string # Source Identifier
      title: string # Title
      summary: string # 2-4 sentence summary
      key_points: string[] # Max 3 key points
      url: string # Original URL
      keywords: string[] # Keywords
      quality_score: 1-5 # Quality Score

constraints:
  filter: "Cutting-edge Tech/Deep Tech/Productivity/Practical Info"
  exclude: "General Science/Marketing Puff/Overly Academic/Job Posts"
  max_items_per_source: 10
  skip_on_error: true

return_format: JSON
```

### Giai đoạn 4: Giám sát & Phản hồi Tác nhân Chính

Trách nhiệm của Tác nhân Chính:

```yaml
Giám sát:
  - Kiểm tra trạng thái trả về của Tác nhân Phụ (thành công/một phần/thất bại)
  - Đếm các mục đã thu thập
  - Ghi lại tỷ lệ thành công trên mỗi nguồn

Vòng Phản hồi:
  - Nếu một Tác nhân Phụ thất bại, quyết định thử lại hoặc bỏ qua
  - Nếu một nguồn thất bại liên tục, đánh dấu là vô hiệu hóa
  - Điều chỉnh linh hoạt lựa chọn nguồn cho các đợt tiếp theo

Quyết định:
  - Các mục >= 25 VÀ Chất lượng cao >= 20 → Dừng thu thập
  - Các mục < 15 → Tiếp tục đợt tiếp theo
  - Tất cả các đợt đã xong nhưng < 20 → Tạo với nội dung có sẵn (Chất lượng hơn Số lượng)
```

### Giai đoạn 5: Đánh giá & Lọc

```yaml
Loại bỏ trùng lặp:
  - Khớp URL chính xác
  - Tương tự tiêu đề (>80% được coi là trùng lặp)
  - Kiểm tra cache.json để tránh trùng lặp lịch sử

Hiệu chỉnh Điểm số:
  - Thống nhất các tiêu chuẩn chấm điểm trên các Tác nhân Phụ
  - Điều chỉnh trọng số dựa trên độ tin cậy của nguồn
  - Điểm thưởng cho các nguồn chất lượng cao được tuyển chọn thủ công

Sắp xếp:
  - Thứ tự giảm dần theo quality_score
  - Sắp xếp theo ưu tiên nguồn nếu điểm bằng nhau
  - Lấy Top 20
```

### Giai đoạn 6: Thu thập bằng Trình duyệt (MCP Chrome DevTools)

Đối với các trang yêu cầu hiển thị JS, sử dụng trình duyệt không đầu:

```yaml
Quy trình: 1. Gọi mcp__chrome-devtools__new_page để mở trang
  2. Gọi mcp__chrome-devtools__wait_for để chờ tải nội dung
  3. Gọi mcp__chrome-devtools__take_snapshot để lấy cấu trúc trang
  4. Phân tích snapshot để trích xuất nội dung cần thiết
  5. Gọi mcp__chrome-devtools__close_page để đóng trang

Các tình huống Áp dụng:
  - ProductHunt (403 trên WebFetch)
  - Latent Space (Substack hiển thị JS)
  - Các ứng dụng SPA khác
```

### Giai đoạn 7: Tạo Báo cáo

```yaml
Đầu ra:
  - Thư mục: NewsReport/
  - Tên tệp: YYYY-MM-DD-news-report.md
  - Định dạng: Standard Markdown

Cấu trúc Nội dung:
  - Tiêu đề + Ngày
  - Tóm tắt Thống kê (Số lượng nguồn, số mục đã thu thập)
  - 20 Mục Chất lượng Cao (Dựa trên mẫu)
  - Thông tin Tạo (Phiên bản, Dấu thời gian)
```

### Giai đoạn 8: Cập nhật Cache

```yaml
Cập nhật cache.json:
  - last_run: Ghi lại thông tin lần chạy này
  - source_stats: Cập nhật thống kê mỗi nguồn
  - url_cache: Thêm các URL đã xử lý
  - content_hashes: Thêm dấu vân tay nội dung
  - article_history: Ghi lại các bài viết đã bao gồm
```

## Các ví dụ Gọi Tác nhân Phụ

### Sử dụng Tác nhân mục đích chung (general-purpose Agent)

Vì các tác nhân tùy chỉnh yêu cầu khởi động lại phiên để được phát hiện, hãy sử dụng mục đích chung và đưa vào lời nhắc worker:

```
Task Call:
  subagent_type: general-purpose
  model: haiku
  prompt: |
    You are a stateless execution unit. Only do the assigned task and return structured JSON.

    Task: Scrape the following URLs and extract content

    URLs:
    - https://news.ycombinator.com (Extract Top 10)
    - https://huggingface.co/papers (Extract top voted papers)

    Output Format:
    {
      "status": "success" | "partial" | "failed",
      "data": [
        {
          "source_id": "hn",
          "title": "...",
          "summary": "...",
          "key_points": ["...", "...", "..."],
          "url": "...",
          "keywords": ["...", "..."],
          "quality_score": 4
        }
      ],
      "errors": [],
      "metadata": { "processed": 2, "failed": 0 }
    }

    Filter Criteria:
    - Keep: Cutting-edge Tech/Deep Tech/Productivity/Practical Info
    - Exclude: General Science/Marketing Puff/Overly Academic/Job Posts

    Return JSON directly, no explanation.
```

### Sử dụng Tác nhân worker (Yêu cầu khởi động lại phiên)

```
Task Call:
  subagent_type: worker
  prompt: |
    task: fetch_and_extract
    input:
      urls:
        - https://news.ycombinator.com
        - https://huggingface.co/papers
    output_schema:
      - source_id: string
      - title: string
      - summary: string
      - key_points: string[]
      - url: string
      - keywords: string[]
      - quality_score: 1-5
    constraints:
      filter: Cutting-edge Tech/Deep Tech/Productivity/Practical Info
      exclude: General Science/Marketing Puff/Overly Academic
```

## Mẫu Đầu ra

```markdown
# Daily News Report (YYYY-MM-DD)

> Curated from N sources today, containing 20 high-quality items
> Generation Time: X min | Version: v3.0
>
> **Warning**: Sub-agent 'worker' not detected. Running in generic mode (Serial Execution). Performance might be degraded.

---

## 1. Title

- **Summary**: 2-4 lines overview
- **Key Points**:
  1. Point one
  2. Point two
  3. Point three
- **Source**: [Link](URL)
- **Keywords**: `keyword1` `keyword2` `keyword3`
- **Score**: ⭐⭐⭐⭐⭐ (5/5)

---

## 2. Title

...

---

_Generated by Daily News Report v3.0_
_Sources: HN, HuggingFace, OneUsefulThing, ..._
```

## Các Ràng buộc & Nguyên tắc

1.  **Chất lượng hơn Số lượng**: Nội dung chất lượng thấp không vào báo cáo.
2.  **Dừng Sớm**: Dừng thu thập khi đạt 20 mục chất lượng cao.
3.  **Ưu tiên Song song**: Các Tác nhân Phụ trong cùng một đợt thực thi song song.
4.  **Khả năng chịu lỗi**: Thất bại của một nguồn đơn lẻ không ảnh hưởng đến toàn bộ quy trình.
5.  **Tái sử dụng Cache**: Tránh thu thập lại cùng một nội dung.
6.  **Kiểm soát Tác nhân Chính**: Mọi quyết định được đưa ra bởi Tác nhân Chính.
7.  **Nhận thức Dự phòng**: Phát hiện tính khả dụng của tác nhân phụ, xuống cấp nhẹ nhàng nếu không khả dụng.

## Hiệu suất Mong đợi

| Kịch bản        | Thời gian Mong đợi | Ghi chú                         |
| --------------- | ------------------ | ------------------------------- |
| Tối ưu          | ~2 phút            | Tier1 đủ, không cần trình duyệt |
| Bình thường     | ~3-4 phút          | Yêu cầu bổ sung Tier2           |
| Cần Trình duyệt | ~5-6 phút          | Bao gồm các trang hiển thị JS   |

## Xử lý Lỗi

| Loại Lỗi                   | Xử lý                                          |
| -------------------------- | ---------------------------------------------- |
| Hết thời gian Tác nhân Phụ | Ghi nhật ký lỗi, tiếp tục sang cái tiếp theo   |
| Nguồn 403/404              | Đánh dấu vô hiệu hóa, cập nhật sources.json    |
| Trích xuất Thất bại        | Trả về nội dung thô, Tác nhân Chính quyết định |
| Sự cố Trình duyệt          | Bỏ qua nguồn, ghi nhật ký                      |

## Tương thích & Dự phòng

Để đảm bảo khả năng sử dụng trên các môi trường Tác nhân khác nhau, các kiểm tra sau phải được thực hiện:

1.  **Kiểm tra Môi trường**:
    - Trong Giai đoạn 1 khởi tạo, cố gắng phát hiện xem tác nhân phụ `worker` có tồn tại không.
    - Nếu không tồn tại (hoặc plugin chưa được cài đặt), tự động chuyển sang **Chế độ Thực thi Tuần tự**.

2.  **Chế độ Thực thi Tuần tự**:
    - Không sử dụng khối song song.
    - Tác nhân Chính thực thi các nhiệm vụ thu thập cho mỗi nguồn một cách tuần tự.
    - Chậm hơn, nhưng đảm bảo chức năng cơ bản.

3.  **Cảnh báo Người dùng**:
    - PHẢI bao gồm một cảnh báo rõ ràng trong tiêu đề báo cáo được tạo cho biết chế độ xuống cấp hiện tại.
