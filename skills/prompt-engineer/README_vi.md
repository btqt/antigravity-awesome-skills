# 🎯 Prompt Engineer

**Phiên bản:** 1.0.1  
**Trạng thái:** ✨ Zero-Config | 🌍 Universal

Chuyển đổi các prompt thô thành các prompt được tối ưu hóa, sẵn sàng cho môi trường production bằng cách sử dụng 11 framework prompting đã được thiết lập.

---

## 📋 Tổng quan

**Prompt Engineer** là một AI skill thông minh giúp phân tích ý định của bạn và tự động tạo ra các prompt được tối ưu hóa cho Claude, ChatGPT, hoặc bất kỳ AI model nào. Thay vì phải chật vật với cách diễn đạt yêu cầu phức tạp, bạn chỉ cần mô tả những gì bạn muốn - skill này sẽ xử lý phần còn lại.

Skill này hoạt động ở chế độ **"magic mode"** - nó hoạt động ngầm và chỉ đặt câu hỏi khi thực sự cần thiết. Bạn cung cấp một ý tưởng sơ bộ, và nó trả về một prompt có cấu trúc, được trau chuốt và sẵn sàng sử dụng.

### ✨ Các tính năng chính

- **🎯 Intent Analysis:** Hiểu được những gì bạn đang cố gắng hoàn thành
- **🧠 11 Frameworks:** RTF, Chain of Thought, RISEN, RODES, Chain of Density, RACE, RISE, STAR, SOAP, CLEAR, GROW
- **🔄 Framework Blending:** Kết hợp nhiều framework khi cần thiết
- **❓ Smart Clarification:** Chỉ đặt câu hỏi khi thực sự cần thiết
- **📏 Adaptive Output:** Điều chỉnh độ dài của prompt cho phù hợp với độ phức tạp của task
- **🌍 Universal:** Hoạt động trong bất kỳ project nào, ở bất kỳ terminal context nào
- **🪄 Magic Mode:** Không có giải thích kỹ thuật - chỉ đưa ra kết quả

---

## 🚀 Bắt đầu nhanh

### Kích hoạt Skill

Sử dụng bất kỳ trigger phrase nào sau đây:

```bash
copilot> improve this prompt: [your rough prompt]
copilot> optimize prompt: [your idea]
copilot> refine prompt: [what you want]
copilot> prompt engineering for: [task description]
copilot> transform into prompt: [concept]
copilot> create prompt for: [goal]
```

### Ví dụ 1: Task đơn giản

**Đầu vào:**

```bash
copilot> improve this prompt: help me write Python code
```

**Skill hỏi:**

```
What type of Python code do you need?
- [ ] Data analysis
- [ ] Web scraping
- [ ] Task automation
- [ ] API/backend development
```

**Bạn chọn:** "Web scraping"

**Đầu ra:**

````markdown
```
You are a Python developer expert in web scraping.

Task: Create a Python script to scrape data from [specify target website].

Requirements:
- Use BeautifulSoup or Scrapy
- Implement pagination and rate limiting
- Store data in structured format (CSV or JSON)
- Include error handling for network issues

Output format: Complete, commented Python code with usage examples.
```
````

---

### Ví dụ 2: Task phức tạp (Không cần làm rõ)

**Đầu vào:**

```bash
copilot> create prompt for: product recommendation system with purchase history and market trends
```

**Đầu ra:**

````markdown
```
Role: You are a senior machine learning engineer specializing in recommendation systems.

Objective: Design a product recommendation system that combines user purchase history, preferences, and market trends.

Details:
- Input data sources: user transaction logs, preference surveys, market trend APIs
- Algorithm considerations: collaborative filtering, content-based filtering, hybrid approaches
- Performance metrics: precision, recall, diversity, novelty
- Scalability: system must handle 100k+ users

Approach step-by-step:
1. Analyze the three data sources and their relationships
2. Propose 2-3 algorithm approaches with trade-offs
3. Design data pipeline architecture
4. Define evaluation metrics
5. Outline implementation roadmap

Examples: Show pseudocode for core recommendation logic

Sense Check: Validate that the solution balances personalization with discovery and scales to production.

Output format: Technical design document with architecture diagrams and implementation roadmap.
```
````

---

### Ví dụ 3: Task mơ hồ (Skill làm rõ)

**Đầu vào:**

```bash
copilot> transform into prompt: I want to improve my process
```

**Skill hỏi:**

```
What process do you want to improve?
- [ ] Work/productivity process
- [ ] Learning process
- [ ] Technical process (development, deployment, etc.)
```

**Bạn chọn:** "Work process"

**Skill hỏi:**

```
What specific aspect?
- [ ] Time management
- [ ] Team collaboration
- [ ] Decision making
```

**Bạn chọn:** "Decision making"

**Đầu ra:**

````markdown
```
Context: I struggle with making decisions quickly and effectively at work.

Objective: Develop a structured framework to improve decision speed and quality.

Current Reality:
- Decisions delayed by excessive analysis
- Lack of clear criteria
- Uncertainty about when to decide vs. gather information

Desired Outcome:
- Consistent decision-making framework
- Clear criteria for different decision types
- Reduced decision fatigue

Your task:
1. Assess current patterns and identify bottlenecks
2. Propose framework (Eisenhower Matrix, RAPID, Decision Tree)
3. Create decision criteria templates
4. Design tracking and iteration process

Output format: Practical playbook with templates, examples, and metrics.
```
````

---

## 📚 Các Framework được hỗ trợ

Skill **Prompt Engineer** sử dụng **11 framework đã được thiết lập** để tối ưu hóa prompt của bạn. Skill tự động lựa chọn và kết hợp các framework này tùy theo task - bạn không bao giờ cần phải biết hoặc chọn chúng theo cách thủ công.

---

### 1. **RTF (Role-Task-Format)**

**Cấu trúc:** Role → Task → Format
**Tốt nhất cho:** Các task yêu cầu chuyên môn hoặc một góc nhìn cụ thể
**Các thành phần:**

- **Role:** "Bạn là một [expert identity]"
- **Task:** "Nhiệm vụ của bạn là [specific action]"
- **Format:** "Định dạng đầu ra: [structure/style]"

**Ví dụ:**

```
You are a senior Python developer.
Task: Refactor this code for better performance.
Format: Provide refactored code with inline comments explaining changes.
```

---

### 2. **Chain of Thought**

**Cấu trúc:** Problem → Step 1 → Step 2 → ... → Solution
**Tốt nhất cho:** Quá trình suy luận phức tạp, debugging, các bài toán toán học, bài tập logic
**Các thành phần:**

- Chia bài toán thành các bước tuần tự
- Thể hiện suy luận ở từng giai đoạn
- Xây dựng hướng tới giải pháp cuối cùng

**Ví dụ:**

```
Solve this problem step-by-step:
1. Identify the core issue
2. Analyze contributing factors
3. Propose solution approach
4. Validate solution against requirements
```

---

### 3. **RISEN**

**Cấu trúc:** Role, Instructions, Steps, End goal, Narrowing
**Tốt nhất cho:** Các dự án chia nhiều giai đoạn với các deliverable và ràng buộc rõ ràng
**Các thành phần:**

- **Role:** Danh tính chuyên gia (Expert identity)
- **Instructions:** Những gì cần làm
- **Steps:** Các hành động tuần tự
- **End goal:** Kết quả mong muốn
- **Narrowing:** Các ràng buộc và khu vực tập trung (focus areas)

**Ví dụ:**

```
Role: You are a DevOps architect.
Instructions: Design a CI/CD pipeline for microservices.
Steps: 1) Analyze requirements 2) Select tools 3) Design workflow 4) Document
End goal: Automated deployment with zero-downtime releases.
Narrowing: Focus on AWS, limit to 3 environments (dev/staging/prod).
```

---

### 4. **RODES**

**Cấu trúc:** Role, Objective, Details, Examples, Sense check
**Tốt nhất cho:** Thiết kế phức tạp, system architecture, research proposals
**Các thành phần:**

- **Role:** Góc nhìn của chuyên gia
- **Objective:** Những gì cần đạt được
- **Details:** Ngữ cảnh chuyên sâu và các yêu cầu
- **Examples:** Hình minh họa cụ thể
- **Sense check:** Các tiêu chí kiểm tra (validation criteria)

**Ví dụ:**

```
Role: You are a system architect.
Objective: Design a scalable e-commerce platform.
Details: Handle 100k concurrent users, sub-200ms response time, multi-region.
Examples: Show database schema, caching strategy, load balancing.
Sense check: Validate solution meets latency and scalability requirements.
```

---

### 5. **Chain of Density**

**Cấu trúc:** Iteration 1 (verbose) → Iteration 2 → ... → Iteration 5 (maximum density)
**Tốt nhất cho:** Tóm tắt, cô đọng nội dung, tổng hợp những văn bản dài
**Quy trình:**

- Bắt đầu với giải thích chi tiết (verbose explanation)
- Thay phiên nén lặp lại trong khi vẫn giữ nguyên thông tin cốt lõi
- Chuyển tiếp kết thúc bằng phiên bản cô đọng tối đa

**Ví dụ:**

```
Compress this article into progressively denser summaries:
1. Initial summary (300 words)
2. Compressed (200 words)
3. Further compressed (100 words)
4. Dense (50 words)
5. Maximum density (25 words, all critical points)
```

---

### 6. **RACE**

**Cấu trúc:** Role, Audience, Context, Expectation
**Tốt nhất cho:** Giao tiếp, thuyết trình, cập nhật dự án cho stakeholder, storytelling
**Các thành phần:**

- **Role:** Danh tính của người thực hiện giao tiếp
- **Audience:** Đối tượng bạn đang hướng tới
- **Context:** Bối cảnh/tình huống của sự việc
- **Expectation:** Những gì đối tượng cần phải biết hoặc làm

**Ví dụ:**

```
Role: You are a product manager.
Audience: Non-technical executives.
Context: Quarterly business review, product performance down 5%.
Expectation: Explain root causes and recovery plan in non-technical terms.
```

---

### 7. **RISE**

**Cấu trúc:** Research, Investigate, Synthesize, Evaluate
**Tốt nhất cho:** Điều tra, đánh giá có hệ thống, công việc chẩn đoán vấn đề
**Quy trình:**

1. **Research:** Thu thập thông tin
2. **Investigate:** Đi sâu tìm hiểu vào các phát hiện
3. **Synthesize:** Kết hợp những insights (thông tin chi tiết)
4. **Evaluate:** Thẩm định và đưa ra những khuyến nghị

**Ví dụ:**

```
Analyze customer churn data using RISE:
Research: Collect churn metrics, exit surveys, support tickets.
Investigate: Identify patterns in churned users.
Synthesize: Combine findings into themes.
Evaluate: Recommend retention strategies based on evidence.
```

---

### 8. **STAR**

**Cấu trúc:** Situation, Task, Action, Result
**Tốt nhất cho:** Giải quyết các vấn đề đi sâu vào bối cảnh phong phú, case study, retrospectives
**Các thành phần:**

- **Situation:** Ngữ cảnh, bối cảnh
- **Task:** Thách thức hoặc nhiệm vụ cụ thể
- **Action:** Những hành động cần làm
- **Result:** Mục tiêu và kết quả dự kiến

**Ví dụ:**

```
Situation: Legacy monolith causing deployment delays (2 weeks per release).
Task: Modernize architecture to enable daily deployments.
Action: Migrate to microservices, implement CI/CD, containerize.
Result: Deploy 10+ times per day with <5% rollback rate.
```

---

### 9. **SOAP**

**Cấu trúc:** Subjective, Objective, Assessment, Plan
**Tốt nhất cho:** Lưu trữ tài liệu có cấu trúc, hồ sơ y tế, technical logs, incident reports
**Các thành phần:**

- **Subjective:** Thông trình báo (symptoms, complaints)
- **Objective:** Các dữ kiện quan sát mang tính hiện thực (metrics, data)
- **Assessment:** Quá trình phân tích chuyên môn và chẩn đoán
- **Plan:** Các bước hành động được đề xuất

**Ví dụ:**

```
Incident Report (SOAP):
Subjective: Users report slow page loads starting 10 AM.
Objective: Average response time increased from 200ms to 3s. CPU at 95%.
Assessment: Database connection pool exhausted due to traffic spike.
Plan: 1) Scale pool size 2) Add monitoring alerts 3) Review query performance.
```

---

### 10. **CLEAR**

**Cấu trúc:** Collaborative, Limited, Emotional, Appreciable, Refinable
**Tốt nhất cho:** Thiết lập mục tiêu, OKRs, measurable objectives, team alignment
**Các thành phần:**

- **Collaborative:** Ai liên quan
- **Limited:** Phạm vi định lượng và giới hạn thời gian thực thi
- **Emotional:** Tại sao việc này lại quan trọng (động lực)
- **Appreciable:** Quá trình đo lường tiến độ có thể đong đếm
- **Refinable:** Nền tảng cải thiện để tối ưu

**Ví dụ:**

```
Q1 Objective (CLEAR):
Collaborative: Engineering + Product teams.
Limited: Complete by March 31, budget $50k, 2 engineers allocated.
Emotional: Reduces customer support load by 30%, improves satisfaction.
Appreciable: Track weekly via tickets resolved, NPS score, deployment count.
Refinable: Bi-weekly retrospectives, adjust priorities based on feedback.
```

---

### 11. **GROW**

**Cấu trúc:** Goal, Reality, Options, Will
**Tốt nhất cho:** Hoạt động huấn luyện (coaching), personal development
**Các thành phần:**

- **Goal:** Mục tiêu đạt được
- **Reality:** Tình huống thực tại
- **Options:** Các lựa chọn và hướng tiếp cận
- **Will:** Cam kết hành động

**Ví dụ:**

```
Career Development (GROW):
Goal: Become senior engineer within 12 months.
Reality: Strong coding skills, weak in system design and leadership.
Options: 1) Take system design course 2) Lead a project 3) Find mentor.
Will: Commit to 5 hours/week study, lead Q2 project, find mentor by Feb.
```

---

### Framework Selection Logic

Skill phân tích input đầu khởi định của bạn và:

1. **Detects task type (Xác định loại task)**
   - Coding, writing, analysis, design, communication, v.v.
2. **Identifies complexity (Xác định độ phức tạp)**
   - Đơn giản (1-2 câu) → Nhanh chóng, cấu trúc tối thiểu
   - Trung bình (1 đoạn văn) → Framework tiêu chuẩn
   - Phức tạp (requirements chi tiết) → Framework nâng cao / kết hợp
3. **Selects primary framework (Chọn framework chính)**
   - RTF → Các task dựa trên Role
   - Chain of Thought → Suy luận step-by-step
   - RISEN/RODES → Các project phức tạp
   - RACE → Thông tin giao tiếp
   - STAR → Giải quyết tình huống
4. **Blends secondary frameworks when needed (Kết hợp frameworks)**
   - RODES + Chain of Thought → Các project kiến trúc kỹ thuật tinh xảo
   - CLEAR + GROW → Nhắm đích mục tiêu nâng cấp kỹ thuật định chuẩn nhà quản lý
   - RACE + STAR → Cấu trúc giao tiếp thông tin

**Bạn không tốn thời gian phải chọn thủ công.** - Framework sẽ tự động hóa xử lý nhờ "magic mode".

---

### Common Framework Blends

| Task Type                | Primary Framework | Blended With     | Result                                            |
| ------------------------ | ----------------- | ---------------- | ------------------------------------------------- |
| Complex technical design | RODES             | Chain of Thought | Structured design with step-by-step reasoning     |
| Leadership development   | CLEAR             | GROW             | Measurable goals with action commitment           |
| Strategic communication  | RACE              | STAR             | Audience-aware storytelling with context          |
| Incident investigation   | RISE              | SOAP             | Systematic analysis with structured documentation |
| Project planning         | RISEN             | RTF              | Multi-phase delivery with role clarity            |

---

## 🎯 Cách hoạt động

```
User Input (rough prompt)
         ↓
┌────────────────────────┐
│ 1. Analyze Intent      │  Người dùng đang cố gắng hoàn thiện mục đích gì?
│    - Task type         │  Coding? Writing? Analysis? Design?
│    - Complexity        │  Đơn giản, độ phức tạp trung bình, hay tổng khối phức tạp?
│    - Clarity           │  Tính rõ ràng trực diện hay vẫn luẩn quất trong sự mơ hồ?
└────────┬───────────────┘
         ↓
┌────────────────────────┐
│ 2. Clarify (Optional)  │  Chỉ kích hoạt khi thực độ thiếu thốn về chi tiết cực kỳ lớn
│    - Ask 2-3 questions │  Multiple choice được tận dụng đính kèm khi có thể
│    - Fill missing gaps │
└────────┬───────────────┘
         ↓
┌────────────────────────┐
│ 3. Select Framework(s) │  Silent selection (Lựa chọn ngầm)
│    - Map task → framework
│    - Blend if needed   │
└────────┬───────────────┘
         ↓
┌────────────────────────┐
│ 4. Generate Prompt     │  Áp dụng các framework rules đi đúng hướng
│    - Add role/context  │
│    - Structure task    │
│    - Define format     │
│    - Add examples      │
└────────┬───────────────┘
         ↓
┌────────────────────────┐
│ 5. Output              │  Văn bản sạch sẵn sàng để copy
│    Markdown code block │  Không chứa các ngôn từ diễn giải luộm thuộm
└────────────────────────┘
```

---

## 🎨 Use Cases

### Coding

```bash
copilot> optimize prompt: create REST API in Python
```

→ Generates structured prompt with role, requirements, output format, examples

---

### Writing

```bash
copilot> create prompt for: write technical article about microservices
```

→ Generates audience-aware prompt with structure, tone, and content guidelines

---

### Analysis

```bash
copilot> refine prompt: analyze sales data and identify trends
```

→ Generates step-by-step analytical framework with visualization requirements

---

### Decision Making

```bash
copilot> improve this prompt: I need to decide between technology A and B
```

→ Generates decision framework with criteria, trade-offs, and validation

---

### Learning

```bash
copilot> transform into prompt: learn machine learning from zero
```

→ Generates learning path prompt with phases, resources, and milestones

---

## ❓ Câu hỏi thường gặp

### Q: Skill này có hoạt động không chỉ trong mỗi giới trị Obsidian vaults chứ?

**A:** Có! Đây là một **universal skill** hoạt động được mọi vùng nền terminal context. Nó không phụ thuộc vào vault structure, hay các external files.

---

### Q: Tôi có cần biết về các Prompting framework không?

**A:** Không. Skill nắm rất chắc 11 frameworks và tự động chọn phương thức xử lý tối ưu.

---

### Q: Skill có giải đáp lý do cho việc tại sao nó lại được chọn framework đấy không?

**A:** Không. Nó hoạt hành tại cõi "magic mode" - bạn nhận kết quả hoàn thiện mà không có các giải thích kỹ thuật.

---

### Q: Skill sẽ có đặt câu hỏi ngược lại với tôi không?

**A:** Tối đa 2-3 câu hỏi. Và chỉ khi thông tin thực sự bị thiếu quan trọng. Hầu hết thời gian, nó tạo prompt trực tiếp.

---

### Q: Tôi có thể tùy chỉnh các frameworks không?

**A:** Skill sử dụng các công cụ chuẩn. Bạn không thể tuỳ chỉnh chúng, nhưng bạn được thay đổi tùy ý trong prompt input.

---

### Q: Nó có hỗ trợ các ngôn ngữ khác ngoài tiếng Anh không?

**A:** Có. Nếu bạn nhập bằng PT, prompt cũng sẽ là PT. Nếu mixed, mặc định là English.

---

### Q: Điều gì xảy ra nếu tôi không thích kết quả hiển thị?

**A:** Hãy yêu cầu nó refine tiếp: "make it shorter", "add more examples", v.v.

---

### Q: Có thể dùng cho mọi AI model (Claude, ChatGPT, Gemini)?

**A:** Có. Prompts là model-agnostic và chạy mượt mà cho conversational AI nào cũng chiều được.

---

## 🔧 Cài đặt (Global Setup)

Skill này thiết kế để hoạt động **globally** trên tất cả project.

### Option 1: Dùng từ Repository

1. Clone repository:

   ```bash
   git clone https://github.com/eric.andrade/cli-ai-skills.git
   ```

2. Config Copilot to load skills globally:
   ```bash
   # Add to ~/.copilot/config.json
   {
     "skills": {
       "directories": [
         "/path/to/cli-ai-skills/.github/skills"
       ]
     }
   }
   ```

### Option 2: Copy to Global Skills Directory

```bash
cp -r /path/to/cli-ai-skills/.github/skills/prompt-engineer ~/.copilot/global-skills/
```

Then configure:

```bash
# Add to ~/.copilot/config.json
{
  "skills": {
    "directories": [
      "~/.copilot/global-skills"
    ]
  }
}
```

---

## 📖 Tìm hiểu thêm

- **Skill Development Guide** - Học cách tự tạo skill của riêng bạn
- **[SKILL.md](./SKILL.md)** - Đặc tả kỹ thuật đầy đủ của skill này
- **[Repository README](../../README.md)** - Tổng quan sơ bộ cho toàn bộ skills

---

## 📄 Version

**v1.0.1** | Zero-Config | Universal  
_Hoạt động trong bất kỳ project nào, bất kỳ context nào, bất kỳ terminal nào._
