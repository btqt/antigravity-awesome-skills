---
name: prompt-engineer
description: "Chuyển đổi các prompt người dùng thành các prompt được tối ưu hóa bằng các framework (RTF, RISEN, Chain of Thought, RODES, Chain of Density, RACE, RISE, STAR, SOAP, CLEAR, GROW)"
category: automation
risk: safe
source: community
tags: "[prompt-engineering, optimization, frameworks, ai-enhancement]"
date_added: "2026-02-27"
---

## Mục đích (Purpose)

Skill này chuyển đổi các prompt thô, chưa có cấu trúc của người dùng thành các prompt được tối ưu hóa cao bằng cách sử dụng các prompting framework đã được thiết lập. Nó phân tích ý định của người dùng, xác định mức độ phức tạp của task, và tự động chọn lựa (các) framework phù hợp nhất để tối đa hóa chất lượng output của Claude/ChatGPT.

Skill hoạt động ở chế độ "magic mode" - nó hoạt động ngầm tự động và chỉ tương tác với người dùng khi cần clarification. Người dùng cũng sẽ nhận được các prompt đã được trau chuốt, sẵn sàng sử dụng mà không bao hàm các thuật ngữ kỹ thuật hay framework jargon.

Đây là một **universal skill** hoạt động trong mọi terminal context, không giới hạn trong nội bộ các file Obsidian vault hay cấu trúc project cụ thể.

## Khi nào nên sử dụng (When to Use)

Kích hoạt skill này khi:

- Người dùng cung cấp một prompt có phần generic hoặc mơ hồ (vd: "help me code Python")
- Người dùng có một ý tưởng nhưng gặp khó khăn khi mô tả nó rõ ràng
- Người dùng thiếu structure, content context, hoặc requirements định nghĩa cụ thể
- Task đòi hỏi suy luận logic step-by-step (debugging, analysis, design)
- Người dùng cần một AI task riêng rẽ chuyên biệt nhưng không rõ về các prompting frameworks quy tụ
- Người dùng muốn cải thiện chất lượng prompt sẵn có
- Người dùng đặt câu hỏi dưới định dạng "how do I ask AI to..." hay "create a prompt for..."

## Quy trình (Workflow)

### Bước 1: Analyze Intent (Phân tích ý định)

**Objective:** Để hiểu điều người dùng hoàn toàn muốn hướng tới đích đến cuối cùng là gì.

**Actions:**

1. Khảo soát prompt đưa vào
2. Detect task characteristics:
   - **Type:** coding, writing, analysis, design, learning, planning, decision-making, creative, v.v.
   - **Complexity:** simple (một tác vụ chung chung single-step), moderate (một tiến trình multi-step), complex (khối design đa tầng có tính logic sâu đặc)
   - **Clarity:** Sự thông đạt triệt diện hay cực mơ hồ chưa bến đỗ
   - **Domain:** technical, business, creative, academic, personal, v.v.
3. Nhận dạng các implicit requirements:
   - User có cần examples?
   - Output format đã chỉ rõ chưa?
   - Có constraints hạn ngạch (về thì, lượng, phạm vi)?
   - Gốc công việc thuộc hàng exploratory hay execution-focused?

**Detection Patterns:**

- **Simple tasks:** Ít chữ (<50 chars), rỗng ngữ cảnh
- **Complex tasks:** Nhiều ngữ ngôn diễn hóa (>200 chars), đa lổ yêu cầu rẽ nhánh
- **Ambiguous tasks:** Rất mập mờ, chỉ trỏ với động từ kiểu "help", "improve"
- **Structured tasks:** Chứa chuỗi quá trình, giai đoạn rõ rệt

### Bước 3: Select Framework(s) (Lựa Framework)

**Objective:** Map các attributes quy chiếu trực diện sang điểm optimal của framework(s).

**Framework Mapping Logic:**

| Task Type                                                   | Recommended Framework(s)                                              | Rationale                                    |
| ----------------------------------------------------------- | --------------------------------------------------------------------- | -------------------------------------------- |
| **Role-based tasks** (act as expert, consultant)            | **RTF** (Role-Task-Format)                                            | Clear role definition + task + output format |
| **Step-by-step reasoning** (debugging, proof, logic)        | **Chain of Thought**                                                  | Encourages explicit reasoning steps          |
| **Structured projects** (multi-phase, deliverables)         | **RISEN** (Role, Instructions, Steps, End goal, Narrowing)            | Comprehensive structure for complex work     |
| **Complex design/analysis** (systems, architecture)         | **RODES** (Role, Objective, Details, Examples, Sense check)           | Balances detail with validation              |
| **Summarization** (compress, synthesize)                    | **Chain of Density**                                                  | Iterative refinement to essential info       |
| **Communication** (reports, presentations, storytelling)    | **RACE** (Role, Audience, Context, Expectation)                       | Audience-aware messaging                     |
| **Investigation/analysis** (research, diagnosis)            | **RISE** (Research, Investigate, Synthesize, Evaluate)                | Systematic analytical approach               |
| **Contextual situations** (problem-solving with background) | **STAR** (Situation, Task, Action, Result)                            | Context-rich problem framing                 |
| **Documentation** (medical, technical, records)             | **SOAP** (Subjective, Objective, Assessment, Plan)                    | Structured information capture               |
| **Goal-setting** (OKRs, objectives, targets)                | **CLEAR** (Collaborative, Limited, Emotional, Appreciable, Refinable) | Goal clarity and actionability               |
| **Coaching/development** (mentoring, growth)                | **GROW** (Goal, Reality, Options, Will)                               | Developmental conversation structure         |

**Blending Strategy (Chiến lược kết nối phối hòa):**

- **Kết chéo đan lồng ghép 2-3 framework** lúc gặp thế bí của đa mảng
- Ví dụ rập: Complex technical project → **RODES + Chain of Thought** (sợi khung vách + trí tuệ tư tính)
- Ví dụ đôn: Leadership decision → **CLEAR + GROW** (Mục đích trong rặng + tính định vị thân tiến lớn mạnh cốt ngã)

**Selection Criteria:**

- Primary framework = Lựa sát sườn phù hợp mức core task
- Secondary framework(s) = Lấp khuyết cho góc phụ mờ đục
- Xin cẩn cáo về tính rườm rà (over-engineering): simple task chớ đâm framework nằng nặc

**Critical Rule (Định luật sinh học thiết kỷ):** Tiến trình diễn thế xảy ra **silently** - không luyên thuyên phân bua giãi bày framework với users.

Role: You are a senior software architect. [RTF - Role]

Objective: Design a microservices architecture for [system]. [RODES - Objective]

Approach this step-by-step: [Chain of Thought]

1. Analyze current monolithic constraints
2. Identify service boundaries
3. Design inter-service communication
4. Plan data consistency strategy

Details: [RODES - Details]

- Expected traffic: [X]
- Data volume: [Y]
- Team size: [Z]

Output Format: [RTF - Format]
Provide architecture diagram description, service definitions, and migration roadmap.

Sense Check: [RODES - Sense check]
Validate that services are loosely coupled, independently deployable, and aligned with business domains.

```

**4.5. Language Adaptation (Thích nghi thể hiện ngôn từ)**
- Original xài tiếng Portuguese (Bồ Đào Nha), output vọt ra theo dạng Portuguese nguyễn
- English đầu vào thì English ngõ sinh đầu ra
- Xào xáo (mixed), luật ngầm default cho AI nghiễm nhiên phải là English

**4.6. Quality Checks**
Cuối cùng khi tung thành bản thân, nhớ tự verify checklist:
- [ ] Prompt nội tính độc lập sinh thể tự tồn tại đứng một mình (self-contained)
- [ ] Tính năng Task bộc lộ rạch ròi cụ thể riêng có (specific & measurable)
- [ ] Định vách output form ngời ngợi
- [ ] Ngôn nhãn diễn từ đừng chênh vênh mấp mô lập lờ (ambiguous language)
- [ ] Nội trị tiểu tiết cho cái mức complexity là phải tinh xuy


## Critical Rules (Luật lõi định sinh)

### **NEVER:**

- ❌ Assume missing clues - ĐỪNG CỐ chấp nối mông lung giả định điều gì khuất lấp - LUÔN GỌI HỎI nếu bặt vô âm tín các thứ mấu lõi quan trọng (critical details missing)
- ❌ Hở dãi bày trò chỉ ra framework cho vui miệng giải thích lèo mép (magic mode - im ỉm tàng hình)
- ❌ Rập khuôn nhạt toẹt máy móc cái generic model prompt (always customize mớ bòng bong)
- ❌ Dùng chát cả mũi technical jargon ở prompt chốt sổ (trừ thể loại user nhà nghề yêu cầu vậy)
- ❌ Xin vặn hỏi user quá 3 hit (mệt rã rời họ)
- ❌ Ghép lời dẫn lê thê sáo rỗng ngoài rìa (meta-commentary output như: "This prompt uses...", "Note that...")
- ❌ Show kết quả thả nổi phơi không nấp dưới dạng Markdown code block
- ❌ Đảo trộn loạn xí ngầu ngôn ngữ lẩm cẩm

### **ALWAYS:**

- ✅ Chạy rà intent kỹ trước khi nhả cúp còi xuất quân bắt đầu khâu (Step 1 is mandatory)
- ✅ Có những câu hỏi đánh động để gỡ đục cặn (Step 2 conditional)
- ✅ Cầm kim chỉ nam Selection framework hợp thời lý (Step 3 mapping)
- ✅ Pha phối trộn các frameworks nhuần nhuyễn khi nhận thấy điểm vọt có lợi về chóp phẩm chất của code generate
- ✅ Ước lệ chiều trường mức độ dông dài hợp với nguyên vị độ khó input gốc
- ✅ Hiện diện cọng ngạc format chỉ định kết quả thành phẩm (include format specifier)
- ✅ Phơi bài chốt đóng gói bên trong nằng nặng Markdown block mượt nhẵn
- ✅ Prompt khôn là prompt không ngoéo nhờ cựa vào chóa context nào ngoài thân nó
- ✅ Rải vài mầm vụn ví dụ cho người nhận dễ tưởng tượng được hình bóng của output
- ✅ Điểm duyệt qua cửa ngõ validation cho gọn chất trước khi release (qua quy trình check 4.6)


### **Ví dụ 2: Complex Clear Prompt → RODES + Chain of Thought (No Clarification)**

**User Input:**
```

copilot> create prompt for: I need to build a product recommendation system considering purchase history, user preferences, and market trends

`````

**Skill Internal Process:**
1. **Analyze:** Rõ ràng mười mươi, nặng đô mảng kỹ nghệ (ML system design), gộc technical
2. **Clarify:** Pass! (Thông tin đã dư sức tải)
3. **Select:** RODES (chuyên vụ design khó gặm) + Chain of Thought (vắt óc lý trí rạch ròi)

**Skill Output:**

````markdown
`````

Role: You are a senior machine learning engineer specializing in recommendation systems.

Objective: Design a product recommendation system that combines user purchase history, preferences, and market trends.

Details:

- Input data sources: user transaction logs, preference surveys, market trend APIs
- Algorithm considerations: collaborative filtering, content-based filtering, hybrid approaches
- Performance metrics: precision, recall, diversity, novelty
- Scalability: system must handle 100k+ users
- Real-time constraints: recommendations updated within 500ms

Approach step-by-step:

1. Analyze the three data sources and their relationships
2. Propose 2-3 algorithm approaches with trade-offs analysis
3. Design data pipeline architecture (ingestion, processing, serving)
4. Define evaluation metrics and A/B testing strategy
5. Outline implementation roadmap with phases

Examples:

- Show pseudocode for core recommendation logic
- Illustrate data flow diagram
- Provide sample API response structure

Sense Check: Validate that the proposed solution balances personalization with discovery, scales to production load, and incorporates real-time market trends effectively.

Output format: Technical design document with architecture diagrams, algorithm justification, and implementation roadmap.

```

```

### **Ví dụ 4: Code Debugging → Chain of Thought**

**User Input:**

```
copilot> optimize prompt: my Python code isn't working, needs debugging
```

**Skill Internal Process:**

1. **Analyze:** Nhắm mắt cũng thấu đây là đồ án đi gỡ gạc lỗi (Debugging task), nòi logic rặt
2. **Clarify:** Tàm tạm Pass (user khất nợ chờ phun code theo line sau)
3. **Select:** Rút ra gươm báu Chain of Thought

**Skill Output:**

````markdown
```
You are an expert Python debugger.

Task: Debug the Python code provided by the user by analyzing it step-by-step.

Debugging approach:
1. Read and understand the code's intended purpose
2. Identify the error or unexpected behavior
3. Trace variable states at each execution step
4. Locate the root cause of the issue
5. Propose fix with explanation
6. Suggest preventive measures to avoid similar bugs

For each step, show your reasoning:
- What you're checking
- What you found
- Why it matters

Output format:
- **Issue identified:** [Describe the bug]
- **Root cause:** [Why it's happening]
- **Fix:** [Corrected code with comments]
- **Prevention:** [Best practices to avoid this in the future]

Include a working example to verify the fix.
```
````

## Notes

Dòng dõi này được mệnh danh thiết chế hóa **platform-agnostic** ở thể tự do nên bạn có thể ném mẻ skill này vào bất kỳ lồng quy trắc nào trên terminal CLI Copilot mọc rễ. Ngàn vạn cấm kỵ cũng chẳng bó buộc:

- Bất chấp định dạng cấu kiện Obsidian
- Vô tâm cả dự án project lỉnh kỉnh
- Phớt lờ mọi tệp hay templates móc nối ngoại lai

Cắm rễ và tồn sinh độc bản, tự trích thu dưỡng nạp input lấy năng lượng và bật ra tri thức framework tự hành nội tại rực sáng bừng.
