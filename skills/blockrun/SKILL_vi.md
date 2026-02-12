---
name: blockrun
description: Sử dụng khi người dùng cần các khả năng mà Claude thiếu (tạo hình ảnh, dữ liệu X/Twitter thời gian thực) hoặc yêu cầu rõ ràng các mô hình bên ngoài ("blockrun", "use grok", "use gpt", "dall-e", "deepseek")
allowed-tools: Read, Bash(python:*), Bash(python3:*), Bash(pip:*), Bash(source:*)
---

# BlockRun

**BlockRun hoạt động với Claude Code và Google Antigravity.**

Bạn không thể tạo hình ảnh. Bạn không có quyền truy cập X/Twitter thời gian thực. BlockRun cung cấp cho bạn những khả năng này.

## Triết lý

Bạn có một ví tiền. Khi bạn cần một khả năng mà bạn thiếu, bạn trả tiền cho nó một cách tự chủ.

| Bạn Cần                  | Bạn Gọi            | Giá                           |
| ------------------------ | ------------------ | ----------------------------- |
| Tạo hình ảnh             | DALL-E             | $0.04/ảnh                     |
| Dữ liệu X thời gian thực | Grok + Live Search | $0.025/nguồn                  |
| Ý kiến thứ hai           | GPT-5.2            | $1.75/M input, $14/M output   |
| Xử lý rẻ hơn             | DeepSeek           | $0.14/M input, $0.28/M output |

**Cách hoạt động:** BlockRun sử dụng thanh toán vi mô x402 để định tuyến yêu cầu của bạn đến OpenAI, xAI, Google, và các nhà cung cấp khác. Không cần API key - ví của bạn trả tiền cho mỗi token.

## Kiểm soát Ngân sách (Tùy chọn)

Nếu người dùng chỉ định ngân sách (ví dụ: "sử dụng tối đa $1"), theo dõi chi tiêu và dừng lại khi đạt đến ngân sách:

```python
from blockrun_llm import setup_agent_wallet

client = setup_agent_wallet()
budget = 1.0  # Ngân sách của người dùng

# Trước mỗi cuộc gọi, kiểm tra xem có trong ngân sách không
spending = client.get_spending()
if spending['total_usd'] >= budget:
    print(f"Đã đạt ngân sách: đã chi ${spending['total_usd']:.4f}")
    # Dừng thực hiện cuộc gọi
else:
    response = client.chat("openai/gpt-5.2", "Xin chào!")

# Cuối cùng, báo cáo chi tiêu
spending = client.get_spending()
print(f"💰 Tổng chi: ${spending['total_usd']:.4f} qua {spending['calls']} cuộc gọi")
```

## Khi nào Sử dụng

| Kích hoạt                                                                                                                      | Hành động của Bạn                 |
| ------------------------------------------------------------------------------------------------------------------------------ | --------------------------------- |
| Người dùng yêu cầu rõ ràng ("blockrun ý kiến thứ hai với GPT về...", "sử dụng grok để kiểm tra...", "tạo hình ảnh với dall-e") | Thực thi qua BlockRun             |
| Người dùng cần thứ gì đó bạn không thể làm (hình ảnh, dữ liệu X trực tiếp)                                                     | Đề xuất BlockRun, đợi xác nhận    |
| Bạn có thể xử lý nhiệm vụ ổn thỏa                                                                                              | Tự làm, không đề cập đến BlockRun |

## Ví dụ Lời nhắc Người dùng

Người dùng sẽ nói những điều như:

| Người dùng Nói                                             | Bạn Làm Gì                                      |
| ---------------------------------------------------------- | ----------------------------------------------- |
| "blockrun tạo hình ảnh hoàng hôn"                          | Gọi DALL-E qua ImageClient                      |
| "sử dụng grok để kiểm tra xem cái gì đang trending trên X" | Gọi Grok với `search=True`                      |
| "blockrun GPT xem xét mã này"                              | Gọi GPT-5.2 qua LLMClient                       |
| "tin tức mới nhất về AI agents là gì?"                     | Đề xuất Grok (bạn thiếu dữ liệu thời gian thực) |
| "tạo logo cho startup của tôi"                             | Đề xuất DALL-E (bạn không thể tạo hình ảnh)     |
| "blockrun kiểm tra số dư của tôi"                          | Hiển thị số dư ví qua `get_balance()`           |
| "blockrun deepseek tóm tắt tệp này"                        | Gọi DeepSeek để tiết kiệm chi phí               |

## Ví & Số dư

Sử dụng `setup_agent_wallet()` để tự động tạo ví và lấy client. Điều này hiển thị mã QR và tin nhắn chào mừng trong lần sử dụng đầu tiên.

**Khởi tạo client (luôn bắt đầu với cái này):**

```python
from blockrun_llm import setup_agent_wallet

client = setup_agent_wallet()  # Tự động tạo ví, hiển thị QR nếu mới
```

**Kiểm tra số dư (khi người dùng hỏi "hiện số dư", "kiểm tra ví", v.v.):**

```python
balance = client.get_balance()  # Số dư USDC on-chain
print(f"Số dư: ${balance:.2f} USDC")
print(f"Ví: {client.get_wallet_address()}")
```

**Hiển thị mã QR để nạp tiền:**

```python
from blockrun_llm import generate_wallet_qr_ascii, get_wallet_address

# Mã QR ASCII để hiển thị terminal
print(generate_wallet_qr_ascii(get_wallet_address()))
```

## Sử dụng SDK

**Điều kiện tiên quyết:** Cài đặt SDK với `pip install blockrun-llm`

### Chat Cơ bản

```python
from blockrun_llm import setup_agent_wallet

client = setup_agent_wallet()  # Tự động tạo ví nếu cần
response = client.chat("openai/gpt-5.2", "2+2 bằng mấy?")
print(response)

# Kiểm tra chi tiêu
spending = client.get_spending()
print(f"Đã chi ${spending['total_usd']:.4f}")
```

### Tìm kiếm X/Twitter Thời gian thực (xAI Live Search)

**QUAN TRỌNG:** Đối với dữ liệu X/Twitter thời gian thực, bạn PHẢI bật Live Search với `search=True` hoặc `search_parameters`.

```python
from blockrun_llm import setup_agent_wallet

client = setup_agent_wallet()

# Đơn giản: Bật tìm kiếm trực tiếp với search=True
response = client.chat(
    "xai/grok-3",
    "Những bài đăng mới nhất từ @blockrunai trên X là gì?",
    search=True  # Bật tìm kiếm X/Twitter thời gian thực
)
print(response)
```

### Tìm kiếm X Nâng cao với Bộ lọc

```python
from blockrun_llm import setup_agent_wallet

client = setup_agent_wallet()

response = client.chat(
    "xai/grok-3",
    "Phân tích nội dung và tương tác gần đây của @blockrunai",
    search_parameters={
        "mode": "on",
        "sources": [
            {
                "type": "x",
                "included_x_handles": ["blockrunai"],
                "post_favorite_count": 5
            }
        ],
        "max_search_results": 20,
        "return_citations": True
    }
)
print(response)
```

### Tạo Hình ảnh

```python
from blockrun_llm import ImageClient

client = ImageClient()
result = client.generate("Một con mèo dễ thương đội mũ bảo hiểm không gian")
print(result.data[0].url)
```

## Tham khảo xAI Live Search

Live Search là API dữ liệu thời gian thực của xAI. Chi phí: **$0.025 mỗi nguồn** (mặc định 10 nguồn = ~$0.26).

Để giảm chi phí, đặt `max_search_results` thành giá trị thấp hơn:

```python
# Chỉ sử dụng 5 nguồn (~$0.13)
response = client.chat("xai/grok-3", "Cái gì đang trending?",
    search_parameters={"mode": "on", "max_search_results": 5})
```

### Tham số Tìm kiếm

| Tham số              | Kiểu   | Mặc định   | Mô tả                                                |
| -------------------- | ------ | ---------- | ---------------------------------------------------- |
| `mode`               | string | "auto"     | "off", "auto", hoặc "on"                             |
| `sources`            | array  | web,news,x | Nguồn dữ liệu để truy vấn                            |
| `return_citations`   | bool   | true       | Bao gồm URL nguồn                                    |
| `from_date`          | string | -          | Ngày bắt đầu (YYYY-MM-DD)                            |
| `to_date`            | string | -          | Ngày kết thúc (YYYY-MM-DD)                           |
| `max_search_results` | int    | 10         | Tối đa nguồn trả về (tùy chỉnh để kiểm soát chi phí) |

### Các Loại Nguồn

**Nguồn X/Twitter:**

```python
{
    "type": "x",
    "included_x_handles": ["handle1", "handle2"],  # Tối đa 10
    "excluded_x_handles": ["spam_account"],        # Tối đa 10
    "post_favorite_count": 100,  # Ngưỡng like tối thiểu
    "post_view_count": 1000      # Ngưỡng lượt xem tối thiểu
}
```

**Nguồn Web:**

```python
{
    "type": "web",
    "country": "US",  # Mã ISO alpha-2
    "allowed_websites": ["example.com"],  # Tối đa 5
    "safe_search": True
}
```

**Nguồn Tin tức:**

```python
{
    "type": "news",
    "country": "US",
    "excluded_websites": ["tabloid.com"]  # Tối đa 5
}
```

## Các Mô hình Khả dụng

| Mô hình                   | Tốt nhất Cho                       | Giá                     |
| ------------------------- | ---------------------------------- | ----------------------- |
| `openai/gpt-5.2`          | Ý kiến thứ hai, review code, chung | $1.75/M in, $14/M out   |
| `openai/gpt-5-mini`       | Suy luận tối ưu hóa chi phí        | $0.30/M in, $1.20/M out |
| `openai/o4-mini`          | Suy luận hiệu quả mới nhất         | $1.10/M in, $4.40/M out |
| `openai/o3`               | Suy luận nâng cao, vấn đề phức tạp | $10/M in, $40/M out     |
| `xai/grok-3`              | Dữ liệu X/Twitter thời gian thực   | $3/M + $0.025/nguồn     |
| `deepseek/deepseek-chat`  | Nhiệm vụ đơn giản, xử lý hàng loạt | $0.14/M in, $0.28/M out |
| `google/gemini-2.5-flash` | Tài liệu rất dài, nhanh            | $0.15/M in, $0.60/M out |
| `openai/dall-e-3`         | Ảnh chân thực như thật             | $0.04/ảnh               |
| `google/nano-banana`      | Ảnh nhanh, nghệ thuật              | $0.01/ảnh               |

_M = triệu tokens. Chi phí thực tế phụ thuộc vào độ dài prompt và phản hồi của bạn._

## Tham khảo Chi phí

Tất cả chi phí LLM là trên mỗi triệu tokens (M = 1,000,000 tokens).

| Mô hình               | Input   | Output   |
| --------------------- | ------- | -------- |
| GPT-5.2               | $1.75/M | $14.00/M |
| GPT-5-mini            | $0.30/M | $1.20/M  |
| Grok-3 (không search) | $3.00/M | $15.00/M |
| DeepSeek              | $0.14/M | $0.28/M  |

| Chi phí Cố định   |                                    |
| ----------------- | ---------------------------------- |
| Grok Live Search  | $0.025/nguồn (mặc định 10 = $0.25) |
| DALL-E image      | $0.04/ảnh                          |
| Nano Banana image | $0.01/ảnh                          |

**Chi phí điển hình:** Một prompt 500 từ (~750 tokens) đến GPT-5.2 tốn ~$0.001 input. Một phản hồi 1000 từ (~1500 tokens) tốn ~$0.02 output.

## Thiết lập & Nạp tiền

**Vị trí ví:** `$HOME/.blockrun/.session` (ví dụ, `/Users/username/.blockrun/.session`)

**Thiết lập lần đầu:**

1. Ví tự động tạo khi `setup_agent_wallet()` được gọi
2. Kiểm tra ví và số dư:

```python
from blockrun_llm import setup_agent_wallet
client = setup_agent_wallet()
print(f"Ví: {client.get_wallet_address()}")
print(f"Số dư: ${client.get_balance():.2f} USDC")
```

3. Nạp $1-5 USDC vào ví trên mạng Base

**Hiển thị mã QR để nạp tiền (ASCII cho terminal):**

```python
from blockrun_llm import generate_wallet_qr_ascii, get_wallet_address
print(generate_wallet_qr_ascii(get_wallet_address()))
```

## Khắc phục sự cố

**"Grok says it has no real-time access"**
→ Bạn quên bật Live Search. Thêm `search=True`:

```python
response = client.chat("xai/grok-3", "Cái gì đang trending?", search=True)
```

**Module not found**
→ Cài đặt SDK: `pip install blockrun-llm`

## Cập nhật

```bash
pip install --upgrade blockrun-llm
```
