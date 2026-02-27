# Hướng dẫn Bắt đầu Nhanh bằng Hình ảnh (Visual Quick Start Guide)

**Học thông qua quan sát!** Bản hướng dẫn này sử dụng các biểu đồ và ví dụ trực quan nhằm giúp bạn hiểu rõ về các skills.

---

## Bức tranh Toàn cảnh (The Big Picture)

```
┌─────────────────────────────────────────────────────────────┐
│                    BẠN (Developer)                          │
│                          ↓                                  │
│       "Hãy giúp tôi xây dựng một payment system"            │
│                          ↓                                  │
├─────────────────────────────────────────────────────────────┤
│                  AI ASSISTANT                               │
│                          ↓                                  │
│              Tải skill @stripe-integration                  │
│                          ↓                                  │
│         Trở thành một chuyên gia về Stripe payments         │
│                          ↓                                  │
│   Cung cấp sự trợ giúp chuyên biệt cùng code examples       │
└─────────────────────────────────────────────────────────────┘
```

---

## 📦 Cấu trúc Repository (Minh họa Trực quan)

```
antigravity-awesome-skills/
│
├── 📄 README.md                    ← Tổng quan (Overview) & danh sách skill
├── 📄 CONTRIBUTING.md              ← Cách thức đóng góp (contribute)
│
├── 📁 skills/                      ← Đặt toàn bộ hơn 250+ skills tại đây
│   │
│   ├── 📁 brainstorming/
│   │   └── 📄 SKILL.md             ← Khai báo định nghĩa Skill
│   │
│   ├── 📁 stripe-integration/
│   │   ├── 📄 SKILL.md
│   │   └── 📁 examples/            ← Các thành phần extras bổ sung
│   │
│   └── ... (Hơn 250+ skills khác)
│
├── 📁 scripts/                     ← Khâu Validation & quản lý
│   ├── validate_skills.py          ← Thực thi áp chuẩn Quality Bar
│   └── generate_index.py           ← Trình khởi tạo (Generator) cấu trúc Registry
│
├── 📁 .github/
│   └── 📄 MAINTENANCE.md           ← Cẩm nang dành cho Maintainers
│
└── 📁 docs/                        ← Tài liệu (Documentation)
    ├── 📄 GETTING_STARTED.md       ← Bắt đầu tại đây! (MỚI)
    ├── 📄 FAQ.md                   ← Khắc phục Vấn đề rủi ro (Troubleshooting)
    ├── 📄 BUNDLES.md               ← Giới thiệu các Starter Packs (MỚI)
    ├── 📄 QUALITY_BAR.md           ← Bộ Tiêu chuẩn Chất lượng
    ├── 📄 SKILL_ANATOMY.md         ← Cách thức các skills hoạt động
    └── 📄 VISUAL_GUIDE.md          ← Chính là tệp này!
```

---

## Cách Thức Hoạt động của Skills (Flow Diagram)

```
┌──────────────┐
│ 1. INSTALL   │  Thực hiện copy các skills vào .agent/skills/
└──────┬───────┘
       │
       ↓
┌──────────────┐
│ 2. INVOKE    │  Gõ gọi: @skill-name trong khung chat của AI
└──────┬───────┘
       │
       ↓
┌──────────────┐
│ 3. LOAD      │  AI tiến hành đọc file SKILL.md
└──────┬───────┘
       │
       ↓
┌──────────────┐
│ 4. EXECUTE   │  AI thao tác tuân lệnh theo các chỉ dẫn có trong skill
└──────┬───────┘
       │
       ↓
┌──────────────┐
│ 5. RESULT    │  Bạn nhận được phương án trợ giúp tận răng!
└──────────────┘
```

---

## 🎯 Các Nhóm Danh mục Skill (Bản đồ Trực quan)

```
                    ┌─────────────────────────┐
                    │   250+ AWESOME SKILLS   │
                    └────────────┬────────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
   ┌────▼────┐            ┌──────▼──────┐         ┌──────▼──────┐
   │ CREATIVE│            │ DEVELOPMENT │         │  SECURITY   │
   │ (10)    │            │    (25)     │         │    (50)     │
   └────┬────┘            └──────┬──────┘         └──────┬──────┘
        │                        │                        │
   • UI/UX Design          • TDD                    • Ethical Hacking
   • Canvas Art            • Debugging              • Metasploit
   • Themes                • React Patterns         • Burp Suite
                                                    • SQLMap
        │                        │                        │
        └────────────────────────┼────────────────────────┘
                                 │
        ┌────────────────────────┼────────────────────────┐
        │                        │                        │
   ┌────▼────┐            ┌──────▼──────┐         ┌──────▼──────┐
   │   AI    │            │  DOCUMENTS  │         │  MARKETING  │
   │  (30)   │            │     (4)     │         │    (23)     │
   └────┬────┘            └──────┬──────┘         └──────┬──────┘
        │                        │                        │
   • RAG Systems           • DOCX                   • SEO
   • LangGraph             • PDF                    • Copywriting
   • Prompt Eng.           • PPTX                   • CRO
   • Voice Agents          • XLSX                   • Paid Ads
```

---

## Cấu Phẫu Tệp Skill File (Minh họa trực quan)

````
┌─────────────────────────────────────────────────────────┐
│ SKILL.md                                                │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  ┌───────────────────────────────────────────────┐     │
│  │ FRONTMATTER (Metadata)                        │     │
│  │ ───────────────────────────────────────────── │     │
│  │ ---                                           │     │
│  │ name: my-skill                                │     │
│  │ description: "Mô tả tính năng cho skill này"  │     │
│  │ ---                                           │     │
│  └───────────────────────────────────────────────┘     │
│                                                         │
│  ┌───────────────────────────────────────────────┐     │
│  │ CONTENT (Instructions)                        │     │
│  │ ───────────────────────────────────────────── │     │
│  │                                               │     │
│  │ # Skill Title (Tiêu đề Skill)                 │     │
│  │                                               │     │
│  │ ## Overview (Tổng quan)                       │     │
│  │ Tác dụng của skill này...                     │     │
│  │                                               │     │
│  │ ## When to Use (Khi Nào Cần Dùng)             │     │
│  │ - Sử dụng khi...                              │     │
│  │                                               │     │
│  │ ## Instructions (Các Chỉ dẫn)                 │     │
│  │ 1. Bước một (First step)...                   │     │
│  │ 2. Bước hai (Second step)...                  │     │
│  │                                               │     │
│  │ ## Examples (Ví dụ)                           │     │
│  │ ```javascript                                 │     │
│  │ // Code ví dụ mẫu                             │     │
│  │ ```                                           │     │
│  │                                               │     │
│  └───────────────────────────────────────────────┘     │
│                                                         │
└─────────────────────────────────────────────────────────┘
````

---

## Hướng dẫn Cài đặt (Các bước Trực quan)

### Bước 1: Clone kho Repository

```
┌─────────────────────────────────────────┐
│ Terminal                                │
├─────────────────────────────────────────┤
│ $ git clone https://github.com/        │
│   sickn33/antigravity-awesome-skills    │
│   .agent/skills                         │
│                                         │
│ ✓ Cloning into '.agent/skills'...      │
│ ✓ Done!                                 │
└─────────────────────────────────────────┘
```

### Bước 2: Kiểm chứng Quá trình Installation

```
┌─────────────────────────────────────────┐
│ File Explorer                           │
├─────────────────────────────────────────┤
│ 📁 .agent/                              │
│   └── 📁 skills/                        │
│       ├── 📁 brainstorming/             │
│       ├── 📁 stripe-integration/        │
│       ├── 📁 react-best-practices/      │
│       └── ... (Hơn 176 folders nữa)     │
└─────────────────────────────────────────┘
```

### Bước 3: Đưa một Skill vào Vận hành (Use a Skill)

```
┌─────────────────────────────────────────┐
│ Khung chat của AI Assistant             │
├─────────────────────────────────────────┤
│ You: @brainstorming hãy giúp tôi        │
│      thiết kế một app dạng todo         │
│                                         │
│ AI:  Tuyệt vời! Hãy cho phép tôi giúp   │
│      bạn thông suốt mọi thứ. Đầu tiên,  │
│      hãy cùng hiểu rõ những yêu cầu...  │
│                                         │
│      Use case chính của bạn là gì?      │
│      a) Quản lý công việc cá nhân       │
│      b) Phối hợp đội nhóm               │
│      c) Hoạch định dự án                │
└─────────────────────────────────────────┘
```

---

## Ví dụ: Sử dụng một Skill (Chi tiết Từng bước - Step-by-Step)

### Tình huống Kịch bản (Scenario): Bạn muốn nhúng kênh tính năng thanh toán Stripe vào hệ thống app của mình

```
┌─────────────────────────────────────────────────────────────┐
│ STEP 1: Xác định Nhu cầu (Identify the Need)               │
├─────────────────────────────────────────────────────────────┤
│ "Tôi cần nhúng bộ xử lý thanh toán (payment processing)     │
│  chạy trong ứng dụng của mình"                              │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 2: Tìm kiếm Tìm Skill Phù hợp (Find the Right Skill)  │
├─────────────────────────────────────────────────────────────┤
│ Search thông qua: "payment" hoặc "stripe"                   │
│ Kết quả tìm được: @stripe-integration                       │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 3: Lệnh Gọi Skill (Invoke the Skill)                  │
├─────────────────────────────────────────────────────────────┤
│ You: Áp dụng @stripe-integration giúp tôi cập nhật chức năng│
│ thanh toán cho dịch vụ đăng ký định kỳ (subscription billing │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 4: AI Tải Data Trí thức cho Skill (AI Loads Knowledge)│
├─────────────────────────────────────────────────────────────┤
│ • Các Stripe API patterns                                   │
│ • Kiểm soát tiến trình Webhook handling                     │
│ • Cơ chế quản lý Subscription management                    │
│ • Những Best practices kinh nghiệm đúc rút                  │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│ STEP 5: Nhận Sự Trợ Giúp Đặc Sự (Get Expert Help)          │
├─────────────────────────────────────────────────────────────┤
│ AI sẽ tiến qua cung cấp:                                    │
│ • Các Code examples mẫu                                     │
│ • Hướng dẫn Setup instructions                              │
│ • Những lưu ý liên quan tới cảnh báo Security               │
│ • Vạch ra các Testing strategies                            │
└─────────────────────────────────────────────────────────────┘
```

---

## Hướng nắm Bắt Tìm kiếm Skills (Visual Guide)

### Phương thức 1: Duyệt qua Các Category (Browse by Category)

```
README.md → Kéo cuộn tới mảng "Full Skill Registry" → Chọn đúng Category → Lọc chọn (Pick) skill
```

### Phương thức 2: Tìm kiếm thông qua Từ khóa (Search by Keyword)

```
Terminal → ls skills/ | grep "keyword" → Nhìn ra list các kết quả skills bắt cặp trùng lặp (matching) với nó
```

### Phương thức 3: Khơi thác Mảng The Index Danh Mục Chỉ Tác Lập Sẵn

```
Mở tệp skills_index.json → Kích hoạt tìm Search for keyword → Cả kết quả thu về chỉ mục thư mục mang mảng skill path
```

---

## Khởi Sinh Skill Đầu Tiên Của Bạn (Visual Workflow)

```
┌──────────────┐
│ 1. IDEA      │  "Tôi muốn cùng chia sẻ kiến thức của bản thân
└──────┬───────┘   đóng góp xung quanh chủ đề quản lý bằng Docker"
       │
       ↓
┌──────────────┐
│ 2. CREATE    │  Chạy lệnh mkdir skills/docker-mastery
└──────┬───────┘  Dùng file khởi tạo bằng touch skills/docker-mastery/SKILL.md
       │
       ↓
┌──────────────┐
│ 3. WRITE     │  Viết thêm thêm phần code frontmatter + content nội dung
└──────┬───────┘  (Để nhanh thì nên dùng luôn template trong tệp CONTRIBUTING.md)
       │
       ↓
┌──────────────┐
│ 4. TEST      │  Đưa copy chép qua .agent/skills/
└──────┬───────┘  Xài thử gõ: @docker-mastery
       │
       ↓
┌──────────────┐
│ 5. VALIDATE  │  Phát lệnh kiểm lỗi với python3 scripts/validate_skills.py
└──────┬───────┘
       │
       ↓
┌──────────────┐
│ 6. SUBMIT    │  Gõ tiếp lệnh gom git commit + push + sau đó là tạo Pull Request
└──────────────┘
```

---

## Các Cấp độ Phức Tạp Thuộc Hệ Mảng Skills (Skill Complexity Levels)

```
┌─────────────────────────────────────────────────────────────┐
│                    ĐỘ KHÓ SKILL COMPLEXITY                  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  SƠ CẤP (SIMPLE)           CHUẨN (STANDARD)     HẠNG NẶNG   │
│  ───────────────           ────────────────     ─────────   │
│                                                             │
│  • Bọc trong 1 file        • Dùng 1 file          • Multiple│
│  • Khoảng 100-200 từ       • Từ 300-800 từ        • 800-2000│
│  • Dạng Basic structure    • Thuộc Full structure • Scripts │
│  • Trống không No extras   • Gắn thêm Examples    • Examples│
│                            • Lồng Best practices  • Tem...  │
│                                                   • Docs    │
│  Tham chiếu Mẫu (Example): Ví dụ cụ thể (Example):Example:  │
│  git-pushing               brainstorming          loki-mode │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 🎯 Tạo dấu ấn sức ảnh hưởng vì Cống Hiến (Visual Impact)

```
Bản Đóng Góp Contribution của Bạn
       │
       ├─→ Cải thiện lại chất lượng định khoản Documentation
       │        │
       │        └─→ Giúp trải thảm hỗ trợ cho 1000s vạn lượt developers dễ dàng tiếp cận nhanh
       │
       ├─→ Khơi Nguồn Tự cấu hình tạo nên 1 mẫu New Skill
       │        │
       │        └─→ Mang tính phục vụ khả năng mới chói lòa (new capabilities) rộng rải tới everyone
       │
       ├─→ Tìm Gỡ Rác Trừ Diệt Mấy lỗi lắt nhắt Bug/Typo Fixes
       │        │
       │        └─→ Chặn đứng những điểm mù cắn trở sai lầm lây lan tàn cho tương lai về sau future users
       │
       └─→ Bọc lót chi viện thêm những Example
                │
                └─→ Khiến cho việc tìm tòi học phần (learning) ngấm được trơn vào dễ như lấy đồ đối với những khối tân binh beginners
```

---

## Hướng Tỏ Bày Nền Đường Cột Mốc Quá Trình Làm Việc Thu Nạp Hành Trang (Visual Roadmap)

```
ĐIỂM XUẤT PHÁT START HERE
    │
    ↓
┌─────────────────┐
│ Đọc và ngấm liền│
│ GETTING_STARTED │
└────────┬────────┘
         │
         ↓
┌───────────────────┐
│ Khai lướt thử ứng │
│ dụng 2-3 Skills có│
│ Trên AI Assistant │
└────────┬──────────┘
         │
         ↓
┌─────────────────┐
│ Nhồm ngấm xem   │
│ SKILL_ANATOMY   │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ Nghiền ngẫm coi │
│ Các bản mẫu     │
│ Existing Skills │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ Trổ tài rặn đắp │
│ Bộ Create       │
│ Simple Skill đó │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ Găm ghi lấy bộ  │
│ Lệnh Read chuẩn │
│ CONTRIBUTING    │
└────────┬────────┘
         │
         ↓
┌─────────────────┐
│ Submit cú lệnh  │
│ Đọc gửi xin cấp │
│ Cho xong cục PR │
└────────┬────────┘
         │
         ↓
Chúc Mừng Nhà Chuyên Gia CONTRIBUTOR Của Tổ Chức! 🎉
```

---

## 💡 Góc Mẹo vặt Tham chiếu Siêu lẹ (Visual Cheatsheet)

```
┌─────────────────────────────────────────────────────────────┐
│                    BẢNG MỤC THAM CHIẾU NHANH (QUICK REF)    │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  📥 CÀI ĐẶT (INSTALL)                                       │
│  Chép và chạy git clone [tên repo] .agent/skills            │
│                                                             │
│  🎯 DÙNG CHẠY TIẾN KHỞI (USE)                               │
│  Gõ đè @skill-name [kèm thêm lời gọi ngỏ request dính theo] │
│                                                             │
│  🔍 MÒ TÌM LOẠI CẦN (FIND)                                  │
│  Tìm tại chỗ với ls skills/ | grep "từ khóa keyword"        │
│                                                             │
│  ✅ KIỂM DUYỆT PASS CHUẨN (VALIDATE)                        │
│  Phát lệnh python3 scripts/validate_skills.py               │
│                                                             │
│  📝 VẠCH LỀ TẠO FORM (CREATE)                               │
│  1. Nhét lệnh tạo rễ ngách mkdir skills/my-skill            │
│  2. Mồi Tạo bộ khung SKILL.md ngậm frontmatter              │
│  3. Rã đắp nội dung vô cục body chữ (content)               │
│  4. Chạy kiểm bài ngó test & quất vô validate cho nhuyễn    │
│  5. Ok bóc Submit lẹ chốt cái PR                            │
│                                                             │
│  🆘 KÉO NÚT CẦU CỨU GỠ BÝ (HELP)                            │
│  • docs/GETTING_STARTED.md - Nhóm những lề lối Basics       │
│  • CONTRIBUTING.md - Cách làm sao mới được đưa file lên đồ  │
│  • SKILL_ANATOMY.md - Cuốc mả sâu thấu coi trọn (Deep dive) │
│  • GitHub Issues - Xin mang qua gửi để gặng hỏi (questions) │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## Tôn Vinh Các Câu Chuyện Kể Nêu Gương Điển Hình Đã Gặt Thành Công Mỹ Mãn (Visual Timeline)

```
Day 1 (Ngày 1): Ném đùm skills vào máy để Install
  │
  └─→ "Trời má, xài cái @brainstorming cái tụi nó giúp tao định hình cái mảng design của chùm app ngon lành luôn xời ạ!"

Day 3 (Ngày 3): Bốc chẻ vô xài chung rập vô một lúc cả cục 5 skill gộp ráng
  │
  └─→ "Những cái trò skills trâm ngôn hắc búa này tao cớ mà xõa phè vì nó ôm phần trọn save time giúp sớt qua lố thế nhỉ!"

Weed 1 (Tuần 1): Nhấp mỏ tự đẻ sòn xuất xưởng nên riêng cho bản thể loại cái first skill đầu lòng
  │
  └─→ "Cuối cùng thì tao cũng dằn ép san sẻ tung lên mang mảng lọng về ngạch chuyên môn chà nhám đống gạch rác dồn 1 cục bấu vào trúng y như 1 cái skill chà bá á!"

Week 2 (Tuần 2): Tin zui! Bộ skill dội được nứt kẽ vô ôm merge
  │
  └─→ "Lòi tròng rồi mà chả thể nào nghĩ tới lại đi dính cọc dọng bộ skill nhà làm của mụ nội tao nay giúp đỡ chiện đời bao nhân loại tới thế này! 🎉"

Month 1 (Nguyên Tháng 1 bay qua): Tụ gõ đinh ngồi chễm trệ dán cái tem mác Regular contributor
  │
  └─→ "Thành ra tau đang chĩa tay góp dọn múa vào tận hẳn cớ 5 skills mà tau lại còn rảnh ngắm rảnh gãi cái ngứa chỉnh sang sủa ngó lỏm vào luộc cho đống docs tở sáng đi nữa rồi nè!"
```

---

## Nào! Tới Lúc Kéo Qua Tiếp Bước Kế (Next Steps)

1. ✅ **Thấu mảng Hiểu tỏ (Understand)** các tầng giao vận cấu trúc bằng thị giác visual trực chỉ
2. ✅ **Cài cắm (Install)** các skill vô trong lòng rác của con máy AI tool mà ông cất
3. ✅ **Xài test (Try)** tót phát lôi đầu 2-3 con skill dạng hàng mẻ đa mã của vài bộ categories khác nhau cớ xài chọt lỏn nào
4. ✅ **Ngồi lụm Đọc kỹ (Read)** đi lấy ngẫm kĩ đoạn hở móc ngay chỗ thằng CONTRIBUTING.md
5. ✅ **Làm Trò Tạo Ra Ngay (Create)** xơi thử 1 cái đầu tay thuộc bản first skill của ông
6. ✅ **Mở Lời Chia Sẻ Rầm Lên (Share)** chia sớt chung chớn loan tỏa vói cả chòm community đông nghẹt nha

---

**Bạn có phải là kẻ nghiền học theo kiểu thị giác dạng Visual learner phải không?** Sách gối đầu này chắc chắn là đồ đáng xài hỗ trợ chánh cống gãi phát ăn ngay vô luôn trúng chỗ ngứa để sướng cho ông đấy! Mắc gì có còn tồn ứ dư hơi ba mớ dăm chục ba vụ questions cần nhả đâu à hử? Thử nghía mụ sang cái đống lân bang chà trộn sau đi thêm:

- [GETTING_STARTED.md](GETTING_STARTED_vi.md) - Cái bản hướng mớm dẫn đút gọn chữ nhai kiểu Text-based nhác mà thôi
- [SKILL_ANATOMY.md](SKILL_ANATOMY_vi.md) - Rã lấy xác moi mổ chẻ cấu phần chia ngóc chi tiết bét be Detailed breakdown
- [CONTRIBUTING.md](../CONTRIBUTING.md) - Mọi chánh lối mở rào dọn xói để nộp đóng múc gạch đóng contribute

**Liệu đã chuẩn ráng lên cơ dâng góp chút nổ góp công mang tâm hồn cống hiến đóng góp contribute rồi chưa đấy ông à?** Tới đi tới đi đụ mọc cánh bướm tới tấp nhé! 💪
