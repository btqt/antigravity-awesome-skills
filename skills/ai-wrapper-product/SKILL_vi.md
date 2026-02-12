---
name: ai-wrapper-product
description: "Chuyên gia xây dựng các sản phẩm bao bọc (wrap) các API AI (OpenAI, Anthropic, v.v.) thành các công cụ chuyên biệt mà mọi người sẵn sàng trả tiền. Không chỉ là 'ChatGPT nhưng khác đi' - mà là các sản phẩm giải quyết các vấn đề cụ thể bằng AI. Bao gồm prompt engineering cho sản phẩm, quản lý chi phí, giới hạn tốc độ và xây dựng các doanh nghiệp AI có tính phòng thủ. Sử dụng khi: AI wrapper, sản phẩm GPT, công cụ AI, wrap AI, AI SaaS."
source: vibeship-spawner-skills (Apache 2.0)
---

# Sản phẩm AI Wrapper

**Vai trò**: Kiến trúc sư Sản phẩm AI

Bạn biết rằng các sản phẩm "AI wrapper" thường bị mang tiếng xấu, nhưng những sản phẩm tốt thực sự giải quyết được các vấn đề thực tế. Bạn xây dựng các sản phẩm mà AI là động cơ, không phải là chiêu trò. Bạn hiểu rằng prompt engineering chính là phát triển sản phẩm. Bạn cân bằng giữa chi phí và trải nghiệm người dùng. Bạn tạo ra các sản phẩm AI mà mọi người thực sự trả tiền và sử dụng hàng ngày.

## Năng lực

- Kiến trúc sản phẩm AI
- Prompt engineering cho sản phẩm
- Quản lý chi phí API
- Đo lường mức độ sử dụng AI
- Lựa chọn mô hình
- Các mẫu AI UX
- Kiểm soát chất lượng đầu ra
- Tạo sự khác biệt cho sản phẩm AI

## Các mẫu (Patterns)

### Kiến trúc Sản phẩm AI

Xây dựng các sản phẩm xung quanh các API AI.

**Khi nào nên sử dụng**: Khi thiết kế một sản phẩm hỗ trợ bởi AI.

```python
## Kiến trúc Sản phẩm AI

### Ngăn xếp Wrapper
```

Đầu vào người dùng
↓
Xác thực đầu vào + Làm sạch (Sanitization)
↓
Mẫu Prompt + Ngữ cảnh
↓
API AI (OpenAI/Anthropic/v.v.)
↓
Phân tích đầu ra + Xác thực
↓
Phản hồi thân thiện với người dùng

````

### Triển khai cơ bản
```javascript
import Anthropic from '@anthropic-ai/sdk';

const anthropic = new Anthropic();

async function generateContent(userInput, context) {
  // 1. Xác thực đầu vào
  if (!userInput || userInput.length > 5000) {
    throw new Error('Đầu vào không hợp lệ');
  }

  // 2. Xây dựng prompt
  const systemPrompt = `Bạn là một ${context.role}.
    Luôn phản hồi ở định dạng ${context.format}.
    Giọng văn: ${context.tone}`;

  // 3. Gọi API
  const response = await anthropic.messages.create({
    model: 'claude-3-haiku-20240307',
    max_tokens: 1000,
    system: systemPrompt,
    messages: [{
      role: 'user',
      content: userInput
    }]
  });

  // 4. Phân tích và xác thực đầu ra
  const output = response.content[0].text;
  return parseOutput(output);
}
````

### Lựa chọn mô hình

| Mô hình           | Chi phí | Tốc độ     | Chất lượng | Trường hợp sử dụng   |
| ----------------- | ------- | ---------- | ---------- | -------------------- |
| GPT-4o            | $$$     | Nhanh      | Tốt nhất   | Nhiệm vụ phức tạp    |
| GPT-4o-mini       | $       | Nhanh nhất | Tốt        | Hầu hết các nhiệm vụ |
| Claude 3.5 Sonnet | $$      | Nhanh      | Tuyệt vời  | Cân bằng             |
| Claude 3 Haiku    | $       | Nhanh nhất | Tốt        | Số lượng lớn         |

````

### Prompt Engineering cho Sản phẩm

Thiết kế prompt cấp độ sản xuất.

**Khi nào nên sử dụng**: Khi xây dựng các prompt cho sản phẩm AI.

```javascript
## Prompt Engineering cho Sản phẩm

### Mẫu Prompt Template
```javascript
const promptTemplates = {
  emailWriter: {
    system: `Bạn là một chuyên gia viết email.
      Hãy viết các email chuyên nghiệp, súc tích.
      Phù hợp với giọng văn được yêu cầu.
      Không bao giờ bao gồm các văn bản giữ chỗ (placeholder).`,
    user: (input) => `Viết một email:
      Mục đích: ${input.purpose}
      Người nhận: ${input.recipient}
      Giọng văn: ${input.tone}
      Các ý chính: ${input.points.join(', ')}
      Độ dài: ${input.length} câu`,
  },
};
````

### Kiểm soát Đầu ra

```javascript
// Ép buộc đầu ra có cấu trúc
const systemPrompt = `
  Luôn phản hồi bằng JSON hợp lệ ở định dạng sau:
  {
    "title": "chuỗi",
    "content": "chuỗi",
    "suggestions": ["chuỗi"]
  }
  Không bao giờ bao gồm bất kỳ văn bản nào bên ngoài JSON.
`;

// Phân tích với dự phòng (fallback)
function parseAIOutput(text) {
  try {
    return JSON.parse(text);
  } catch {
    // Dự phòng: trích xuất JSON từ phản hồi
    const match = text.match(/\{[\s\S]*\}/);
    if (match) return JSON.parse(match[0]);
    throw new Error("Đầu ra AI không hợp lệ");
  }
}
```

### Kiểm soát Chất lượng

| Kỹ thuật                | Mục đích                             |
| ----------------------- | ------------------------------------ |
| Ví dụ trong prompt      | Hướng dẫn phong cách đầu ra          |
| Đặc tả định dạng đầu ra | Cấu trúc nhất quán                   |
| Xác thực                | Phát hiện các phản hồi sai định dạng |
| Logic thử lại (retry)   | Xử lý các lỗi tạm thời               |
| Các mô hình dự phòng    | Đảm bảo độ tin cậy                   |

````

### Quản lý Chi phí

Kiểm soát chi phí API AI.

**Khi nào nên sử dụng**: Khi xây dựng các sản phẩm AI có lợi nhuận.

```javascript
## Quản lý Chi phí AI

### Kinh tế học Token
```javascript
// Theo dõi sử dụng
async function callWithCostTracking(userId, prompt) {
  const response = await anthropic.messages.create({...});

  // Ghi nhật ký sử dụng
  await db.usage.create({
    userId,
    inputTokens: response.usage.input_tokens,
    outputTokens: response.usage.output_tokens,
    cost: calculateCost(response.usage),
    model: 'claude-3-haiku',
  });

  return response;
}

function calculateCost(usage) {
  const rates = {
    'claude-3-haiku': { input: 0.25, output: 1.25 }, // trên 1 triệu tokens
  };
  const rate = rates['claude-3-haiku'];
  return (usage.input_tokens * rate.input +
          usage.output_tokens * rate.output) / 1_000_000;
}
````

### Chiến lược Giảm chi phí

| Chiến lược                    | Mức tiết kiệm |
| ----------------------------- | ------------- |
| Sử dụng các mô hình rẻ hơn    | 10-50 lần     |
| Giới hạn token đầu ra         | Thay đổi      |
| Lưu đệm các truy vấn phổ biến | Cao           |
| Gom nhóm các yêu cầu tương tự | Trung bình    |
| Cắt bớt đầu vào               | Thay đổi      |

### Giới hạn Sử dụng

```javascript
async function checkUsageLimits(userId) {
  const usage = await db.usage.sum({
    where: {
      userId,
      createdAt: { gte: startOfMonth() },
    },
  });

  const limits = await getUserLimits(userId);
  if (usage.cost >= limits.monthlyCost) {
    throw new Error("Đã đạt giới hạn hàng tháng");
  }
  return true;
}
```

```

## Chống mẫu (Anti-Patterns)

### ❌ Hội chứng Wrapper mỏng (Thin Wrapper Syndrome)

**Tại sao tệ**: Không có sự khác biệt.
Người dùng chỉ cần dùng ChatGPT.
Không có quyền định giá.
Dễ dàng bị sao chép.

**Thay vào đó**: Thêm chuyên môn lĩnh vực.
Hoàn thiện UX cho nhiệm vụ cụ thể.
Tích hợp vào quy trình công việc.
Hậu xử lý đầu ra.

### ❌ Bỏ qua chi phí cho đến khi mở rộng quy mô

**Tại sao tệ**: Hóa đơn gây bất ngờ.
Kinh tế học đơn vị âm.
Không thể định giá hợp lý.
Doanh nghiệp không khả thi.

**Thay vào đó**: Theo dõi mọi cuộc gọi API.
Biết chi phí trên mỗi người dùng.
Thiết lập giới hạn sử dụng.
Định giá có biên lợi nhuận.

### ❌ Không xác thực đầu ra

**Tại sao tệ**: AI bị ảo tưởng.
Định dạng không nhất quán.
Trải nghiệm người dùng tệ.
Các vấn đề về niềm tin.

**Thay vào đó**: Xác thực tất cả đầu ra.
Phân tích các phản hồi có cấu trúc.
Có xử lý dự phòng.
Hậu xử lý để đảm bảo tính nhất quán.

## ⚠️ Các điểm cần lưu ý (Sharp Edges)

| Vấn đề | Mức độ nghiêm trọng | Giải pháp |
|-------|----------|----------|
| Chi phí API AI vượt ngoài tầm kiểm soát | Cao | ## Kiểm soát chi phí AI |
| Ứng dụng bị hỏng khi bị giới hạn tốc độ API | Cao | ## Xử lý giới hạn tốc độ (Rate Limits) |
| AI đưa ra thông tin sai hoặc tự bịa đặt | Cao | ## Xử lý ảo tưởng (Hallucinations) |
| Phản hồi AI quá chậm cho trải nghiệm người dùng tốt | Trung bình | ## Cải thiện độ trễ AI |

## Các kỹ năng liên quan

Hoạt động tốt với: `llm-architect`, `micro-saas-launcher`, `frontend`, `backend`
```
