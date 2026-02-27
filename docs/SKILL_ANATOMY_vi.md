# Giải phẫu một Skill - Hiểu rõ Cấu trúc (Anatomy of a Skill)

**Bạn muốn hiểu rõ các skills hoạt động như thế nào ở tầng bên dưới (under the hood)?** Cẩm nang hướng dẫn này sẽ chia nhỏ (breaks down) từng phần cấu tạo nên một file skill.

---

## 📁 Cấu trúc Thư mục Cơ bản (Basic Folder Structure)

```
skills/
└── my-skill-name/
    ├── SKILL.md              ← Bắt buộc (Required): Định nghĩa chính của skill
    ├── examples/             ← Tùy chọn (Optional): Các files ví dụ
    │   ├── example1.js
    │   └── example2.py
    ├── scripts/              ← Tùy chọn (Optional): Các scripts hỗ trợ (Helper scripts)
    │   └── helper.sh
    ├── templates/            ← Tùy chọn (Optional): Các mẫu đoạn mã (Code templates)
    │   └── template.tsx
    ├── references/           ← Tùy chọn (Optional): Tài liệu tham khảo
    │   └── api-docs.md
    └── README.md             ← Tùy chọn (Optional): Tài liệu bổ sung
```

**Quy tắc quan trọng (Key Rule):** Chỉ có file `SKILL.md` là bắt buộc. Mọi thứ khác đều là tùy chọn!

---

## Cấu trúc file SKILL.md

Mỗi file `SKILL.md` có hai phần chính:

### 1. Frontmatter (Metadata)

### 2. Nội dung (Instructions)

Hãy cùng phân tích từng phần:

---

## Phần 1: Frontmatter

Frontmatter luôn nằm ở trên cùng, được đặt ở giữa cặp ký tự `---`:

```markdown
---
name: my-skill-name
description: "Mô tả ngắn gọn về sứ mệnh mà skill này thực hiện"
---
```

### Các Trường Bắt Buộc (Required Fields)

#### `name`

- **Nó là gì:** Định danh của skill (identifier)
- **Định dạng format:** chữ-thường-có-dấu-gạch-ngang (kebab-case)
- **Bắt buộc khớp (Must match):** Tên thư mục một cách chính xác
- **Ví dụ:** `stripe-integration`

#### `description`

- **Nó là gì:** Tóm tắt gói gọn trong duy nhất một câu
- **Định dạng format:** Chuỗi (String) nằm bên trong cặp dấu ngoặc kép
- **Độ dài (Length):** Cố gắng giữ nó không vượt quá 150 ký tự
- **Ví dụ:** `"Stripe payment integration patterns including checkout, subscriptions, and webhooks"`

### Các Trường Tùy Chọn (Optional Fields)

Một số skills cũng thường trang bị đi kèm các lượng metadata dữ liệu bổ sung khác:

```markdown
---
name: my-skill-name
description: "Mô tả ngắn gọn"
risk: "safe" # none | safe | critical | offensive (hãy tham chiếu tại QUALITY_BAR.md)
source: "community"
tags: ["react", "typescript"]
---
```

---

## Phần 2: Nội dung (Content)

Sau phần frontmatter ngay lập tức theo sau chính là nội dung cốt lõi của skill. Dưới đây là cấu trúc được đề xuất khuyên dùng:

### Các Phần Được Đề Xuất (Recommended Sections)

#### 1. Tiêu đề (H1 - Title)

```markdown
# Tiêu đề của Skill (Skill Title)
```

- Sử dụng một tiêu đề rành mạch, có yếu tố nêu bật được vai trò
- Nội dung tiêu đề thường phải khớp với hoặc là bản phân giải mở rộng từ tên của đoạn skill (skill name)

#### 2. Tổng quan (Overview)

```markdown
## Overview

Một phân đoạn giải thích tóm lược ngắn gọn đề cập tác dụng của skill này và lý giải lý do của việc vì sao nó phải được sinh ra.
Sử dụng tầm khoảng từ 2 tới 4 câu văn là sự lựa chọn hợp lý hoàn hảo nhất.
```

#### 3. Bối cảnh Khi nào Sử dụng (When to Use)

```markdown
## When to Use This Skill

- Sử dụng ngay khi bạn cần đến [kịch bản tính huống 1]
- Sử dụng vào trong khi xây dựng với [kịch bản tình huống 2]
- Sử dụng vào thời điểm user có hỏi tới [kịch bản tính huống 3]
```

**Tại sao phần này lại quan trọng:** Giúp cho AI biết được thời điểm thích hợp và chuẩn xác để kích hoạt (activate) skill này

#### 4. Các Chỉ dẫn Cốt lõi (Core Instructions)

```markdown
## How It Works

### Bước 1 (Step 1): [Hành động]

Các cấu trúc hướng dẫn theo góc độ chi tiết...

### Bước 2 (Step 2): [Hành động]

Lại là một vài chỉ dẫn tiếp bước...
```

**Đây được cấu tạo chính là đầu não và trái tim làm nên skill của bạn** - là những bước thao tác cực kỳ minh bạch và có tính khả thi dễ dàng thực thi ngay (actionable steps)

#### 5. Những Đoạn Mã Ví dụ (Examples)

```markdown
## Examples

### Ví dụ 1: [Trường hợp sử dụng (Use Case)]

\`\`\`javascript
// Đoạn code cho Example mẫu
\`\`\`

### Ví dụ 2: [Một Use Case khác chăng]

\`\`\`javascript
// Viết thêm phần code khác
\`\`\`
```

**Tại sao bộ thiết kế các ví dụ lại có tầm quan trọng chủ chốt:** Cung cấp tư liệu cho con AI mục sở thị chính xác việc định nghĩa ra 1 output "Chất lượng" nó sẽ có thể trông ra sao

#### 6. Các Phép Hiện thực Chuẩn Thiết kế (Best Practices)

```markdown
## Best Practices

- ✅ Hãy thiết kế làm như thế này
- ✅ Ngoài ra vẫn có thể ứng dụng cái trò này nữa
- ❌ Tuyệt nhiên không được tiến hành làm ra cái dạng này
- ❌ Hãy triệt để đề phòng loại thiết kế tránh cái này
```

#### 7. Những Gài bẫy Lỗi ngầm (Common Pitfalls)

```markdown
## Common Pitfalls

- **Trục trặc Vấn đề (Problem):** Liệt kê Description báo lỗi
  **Hướng đi Giải pháp (Solution):** Phương thức dùng để vượt rào (How to fix it)
```

#### 8. Skills Có tương đồng và móc xích (Related Skills)

```markdown
## Related Skills

- `@other-skill` - Quản lý xem thời điểm khi nào cần thế vai đè mảng ứng dụng mới
- `@complementary-skill` - Phô diễn cách thức kết nối làm nền cho nó hoạt động cùng nhau tương thích
```

---

## Kể câu chuyện Viết Lập trình Các Chỉ dẫn (Instructions) Đạt Hiệu suất cao

### Dùng Phương thức Diễn đạt Rõ ràng & Dứt khoát đi thẳng (Direct Language)

**❌ Tệ (Bad):**

```markdown
Bạn có lẽ nên thử cân nhắc xem liệu có nên đánh giá có thể kiểm tra coi user đó đã authentication hay chưa thì sẽ ổn hơn.
```

**✅ Tốt (Good):**

```markdown
Nhất quyết phải kiểm tra xem user kia đã được authenticated trước lúc chuyển sang quy trình kế.
```

### Khai thác Tính năng của Động Từ Hành Động (Action Verbs)

**❌ Tệ (Bad):**

```markdown
Cái file đó buộc nên được tạo ra...
```

**✅ Tốt (Good):**

```markdown
Khởi tạo ngay file đó trực tiếp luôn đi...
```

### Đề cao Tính Rõ ràng Tường minh (Be Specific)

**❌ Tệ (Bad):**

```markdown
Hãy thực thi thiết lập cấu hình Database theo chuẩn đúng cách.
```

**✅ Tốt (Good):**

```markdown
1. Thiết kế và tạo lập một PostgreSQL database chuyên biệt
2. Vận hành quy trình migrations: `npm run migrate`
3. Định danh khởi nguồn cho kho dữ liệu (Seed initial data): `npm run seed`
```

---

## Chi tiết Các Thành phần Tùy chọn (Optional Components)

### Thư mục Scripts

Trong thời gian vận hành điều hướng nếu skill của bạn lại cần đến các công cụ helper scripts:

```
scripts/
├── setup.sh          ← Phục vụ cấu hình tự động thiết lập (Setup automation)
├── validate.py       ← Áp dụng để chạy những tools cho tiến độ Validation
└── generate.js       ← Bộ chuyển mã và tự đồng bộ các đoạn (Code generators)
```

**Gọi tên tham chiếu đến chúng trong file lõi SKILL.md:**

```markdown
Rà quét và chạy tập tinh setup script:
\`\`\`bash
bash scripts/setup.sh
\`\`\`
```

### Thư mục Examples

Tổng hợp những bản ví dụ từ case thực tế ngoài đời nhằm nâng tầm minh họa cho dòng skill:

```
examples/
├── basic-usage.js
├── advanced-pattern.ts
└── full-implementation/
    ├── index.js
    └── config.json
```

### Thư mục Templates

Khu vực chứa Code templates được biên chế sẵn có khả năng dễ dàng phục hồi chạy tái sử dụng (Reusable):

```
templates/
├── component.tsx
├── test.spec.ts
└── config.json
```

**Cấu hình gọi tham chiếu tại file SKILL.md:**

```markdown
Nắm lấy cấu trúc template này và định dạng chúng như một bệ phóng xuất phát điểm (starting point):
\`\`\`typescript
{{#include templates/component.tsx}}
\`\`\`
```

### Thư mục References

Bộ từ điển trỏ đến điểm đích liên kết nguồn tư liệu bên ngoài (External documentation) hoặc dẫn nối kho API references:

```
references/
├── api-docs.md
├── best-practices.md
└── troubleshooting.md
```

---

## Tiêu chuẩn Mức định lượng Kích cỡ cho từng Loại (Skill Size Guidelines)

### Skill Nhanh gọn Khả dụng Tối thiểu (Minimum Viable Skill)

- **Frontmatter:** name + description hạng nhẹ
- **Content:** Loan phỏng 100-200 từ
- **Sections:** Sở hữu Overview + Chút Instructions

### Skill Bản đầy đủ Tiêu chuẩn (Standard Skill)

- **Frontmatter:** name + description
- **Content:** Tối đa đạt mức 300-800 từ
- **Sections:** Phân rã theo Overview + When to Use + Thêm Instructions + Có Examples

### Skill Quy chuẩn Toàn diện Bao hàm Vượt bậc (Comprehensive Skill)

- **Frontmatter:** name + description + Tích hợp cả mấy bản optional fields nếu có
- **Content:** Bao chót dạt mức 800-2000 từ
- **Sections:** Điểm mặt gọi chuẩn toàn bộ các thẻ chuyên ngành (Recommended sections) đã cấp
- **Extras:** Tham khảo có Scripts, kèm examples, tích hợp mảng templates cho đẹp

**Nguyên tắc vàng đo ni đóng giày (Rule of thumb):** Nên triển lúc Khởi nguyên theo các bản kích cỡ thật tinh thể (nhỏ và gọn - start small), để rồi dãn cơ kéo to (expand) bằng việc điều chỉnh bám dựa trực tiếp ở kho lượng feedback thu nạp được

---

## Các Phương pháp Tùy chỉnh Các Format chuẩn (Formatting Best Practices)

### Phương tiện Phô diễn Khả năng Sử dụng Markdown Tối cao (Effectively)

#### Khung khoanh vùng mảng Code (Code Blocks)

Hãy nhất mực nghiêm chỉ tuân thủ việc quy ước bắt khai báo tên (specify) của ngôn ngữ:

```markdown
\`\`\`javascript
const example = "code";
\`\`\`
```

#### Phương thức Dùng Danh sách (Lists)

Ứng dụng cách phô diễn đồng nhất cùng chung một nền định dạng (formatting):

```markdown
- Hạng mục 1 (Item 1)
- Lại là Item 2
  - Chẻ nhỏ mục (Sub-item 2.1)
  - Vẫn tính là sub (Sub-item 2.2)
```

#### Nhấm mạnh Độ quan tâm (Emphasis)

- **In đậm (Bold)** hỗ trợ truyền tải nhóm thuật ngữ quan trọng cốt lõi có lực chú ý đỉnh cao (important terms): `**important**`
- _In nghiêng (Italic)_ chuyên dụng để nhắc léo và xoáy đáp sự nhấn mạnh (emphasis): `*emphasis*`
- Phương thức đánh dác `Code` chỉ xài để bao bọc đờn các lệnh ở Terminal/mã số (commands/code): `` `code` ``

#### Phương thức Gọi Các Đường dẫn Liên đới (Links)

```markdown
[Đoạn văn bản sẽ bọc lấy link (Link text)](https://example.com)
```

---

## ✅ Bảng Check đối soát Quality (Quality Checklist)

Dùng trước giờ G khoá bài chốt lại bước định hình rà soát cho skill của riêng bạn trước lúc đưa ra công chúng (finalizing):

### Theo dõi Chỉ tiêu Chất lượng Nội dung (Content Quality)

- [ ] Các thông điệp hướng (instructions) phải thật rõ ràng đi kèm cả sức nặng có thể khích ứng thực thi xử lý sự vụ liền (actionable)
- [ ] Các loại cases ví dụ (examples) đều đạt chuẩn mang tính ứng dụng ngoài thực tế đời sống kèm luôn sự hữu hiệu và độ khả dùng (helpful)
- [ ] Khắc chế lỗi do làm ẩu bị sai ngớ ngẩn lúc gõ (typos) hoặc đánh nhầm sai bét lỗi ngữ pháp hành văn (grammar errors)
- [ ] Sự độ xác thực đến từ chuyên ngành (Technical accuracy) bắt buộc phải do 1 bên có đủ thẩm quyền ấn xác nhận bảo hành (verified)

### Phần Mảng miếng Cấu trúc (Structure)

- [ ] Lõi code Frontmatter dứt điểm phải hoàn toàn là 1 cơ cấu kiến trúc chuỗi YAML chuẩn đúng định dạng rành mạch cực kỳ hợp lệ
- [ ] Khớp tên nhãn tag phân vùng (name) với nhãn tên ở ngoài khu folder (folder name) chưa chệch 1 milimet
- [ ] Từng khối tách nhãn chia lớp phân (sections) buộc phải chốt kẽ sắp đồ (organized) bằng một lối mòn logic cực kỳ trật tự và ngăn nắp
- [ ] Cỡ kiểu định dạng trên dãy Headings đều nghiêm cẩn tuân thủ một trật tự cấp bậc có phân vai (hierarchy) từ lớn đến nhỏ (H1 → H2 → H3)

### Quan lại Khâu Tính năng Hoàn chỉnh của nó (Completeness)

- [ ] Bề nổi phân khúc đoạn Tổng Quan (Overview) được ưu thế là đã giải thích rốt ráo chữ "vì sao" ("왜 - why")
- [ ] Mảng Instructions đã phơi bày cặn kẽ và giải quyết ngọn ngành chi tiết của phần "làm như thế nào" ("어떻게 - how")
- [ ] Phần Examples làm mẫu sẵn các đường nét giúp bộc bạch mô tả cho rõ "là loại cái gì đây" ("무엇 - what")
- [ ] Tần tật nhóm ngoại lệ gây khó cho quy luật xử lý (edge cases) đều được truy vết và đưa ra bộ hướng xử lý bài bản (addressed)

### Quan tâm Phần Đánh giá Tính khả dụng hiệu năng (Usability)

- [ ] Hãy cứ chắc nịch là đến cấp cỡ 1 tay tập tễnh vỡ lòng học nghề (beginner) khi theo dõi là đủ tầm học khôn bắt theo liền
- [ ] Đến cả giới cao thủ thành tinh cộm cáng (an expert) khi ghé tham khảo vẫn cứ thấy nó thiết thực hệt như là kim ngọc lương ngôn (useful)
- [ ] Có thể để tự 1 cỗ máy AI bọc vỏ nhai (parse/phân tích cú pháp sinh từ) mớ kiến trúc này một cách đúng nghĩa (correctly) trơn tru
- [ ] Bám rễ giá trị đi cùng hành trang tìm hướng giải cứu một nghịch lỗi đầy nan nguy của thực tại trong thực tiễn (real problem)

---

## 🔍 Chuyên mục Phân tích các Ví dụ Thực tế của cuộc sống (Real-World Example Analysis)

Chúng ta hãy cùng nhau thử mổ xẻ phân tích một skill trong thực tế ở ngay hiện tiền: quy vào cái gọi là `brainstorming`

```markdown
---
name: brainstorming
description: "Xin BẮT BUỘC chốt dùng điều kiện quy chuẩn này đi trước đã hẵng lao vô mấy vụ sáng tác cho công trình sáng tạo (creative work)..."
---
```

**Bảng Phân tích (Analysis):**

- ✅ Đặt loại Tên (name) rõ mười mươi không mù mờ
- ✅ Có kèm chi tiết mảng tính năng (description) có yếu tố mang tính thúc ép áp lệnh để sinh ra sự khẩn cấp và không được quyền chống cự (urgency) (Dứt lời là "BẮT BUỘC SÀI LIỀN - MUST use")
- ✅ Trình bày rõ bối cảnh được áp chế vào thời điểm khi nào thì nên dùng cái skill vớ vẩn này

```markdown
# Brainstorming Ideas Into Designs

## Overview

Help turn ideas into fully formed designs...
```

**Bảng Phân tích:**

- ✅ Điểm của quả Tiêu đề rành mạch chẳng lẫn đi đâu được
- ✅ Lời thuyết giảng tóm gọn phần Overview lại cứ rành rành cô đặc thật là súc tích
- ✅ Đã giải thích cho ngộ ra một cái giá trị truyền đạt định hướng siêu khủng gọi là mệnh đề đưa hướng Value proposition

```markdown
## The Process

**Understanding the idea:**

- Check out the current project state first
- Ask questions one at a time
```

**Bảng Phân tích:**

- ✅ Bể gãy chẽ khúc nhỏ vụn theo dạng từng ranh mục có giới hạn định kỳ rạch ròi (phases) làm chuẩn
- ✅ Rất đặc thù bám sát hoàn cảnh (Specific), làm là thấy rõ kết quả theo chuỗi thao tác luôn (actionable steps)
- ✅ Cực kỳ thân thiện cho quá trình ngấm và rập khuôn thao tác thực chiến mô phỏng (Easy to follow)

---

## Dạo đầu qua số mẫu Các Pattern Nâng cao (Advanced Patterns)

### Trò cấu kết Logic Có chứa thuật điều kiện ở trỏng (Conditional Logic)

```markdown
## Instructions

Lỡ mai mà phát sinh sự cố user của mảng đó lại hiện đi đang làm việc xử lý code với nền React:

- Lão nên xài cái định dạng functional components
- Gắn lệnh đưa chuẩn theo hệ ưu tiên lấy của (Prefer) mảng kho hooks chứ tránh xài qua thứ hệ hàng cổ là class components nha

Nếu user gặp mặt đang làm việc dính chấu với Vue:

- Xài vô mảng Composition API đê
- Bó dính thao tác đó mà hẵng bám đít tuân theo quy chuẩn dành cho các nhóm pattern thuộc dòng Vue 3 nhá
```

### Bộ nghệ thuật Điểm tin hé lộ Lũy tiến Tịnh độ phân thân (Progressive Disclosure)

```markdown
## Basic Usage

[Cứ cho cung cấp ba số các món instructions và chỉ dẫn ở mức căn bản đơn giản đủ năng lực phục vụ sài ngon tốt bét nhè cho đa số các dòng common cases]

## Advanced Usage

[Sắp bài thiết lập riêng tư các chuẩn hạng dạng mô hình pattern cực kỳ đan xen đa mốc phức tạp để chừa sân chơi nhằm để dành đưa dùng cho hạng người dùng hạng nặng - power users]
```

### Đi săn Tham chiếu qua Chéo qua về vòng (Cross-References)

```markdown
## Related Workflows

1. Vô đề số mọt, vươn tay chộp hãy dùng ngay món `@brainstorming` cất công vô để mài design (thiết kế)
2. Đi nước 2, luồn lách lại dùng thử `@writing-plans` cốt để mà bấu víu plan (lên kế hoạch kỹ xào)
3. Điểm dừng 3 cuối cùng kia, ôm đồm xài phát cái món `@test-driven-development` để đóng cọc chốt hạ cái vụ implement (tiến hành xuất phát điểm cài đặt nạo code)
```

---

## Chỉ số hiệu quả dành đánh giá khả năng của Skill (Skill Effectiveness Metrics)

Vậy để làm sao nhằm biết rằng bộ định nghĩa skill của phe bạn thiết kế có thật sự đạt điểm tiêu chuẩn cực kỳ tuyệt đỉnh (good) chưa đó nghe:

### Bài kiểm tra Sự gãy gọn rõ nghĩa cực mượt (Clarity Test)

- Sống mà lôi cái thể loại cho 1 gã lạ hoắc (unfamiliar) ngoại đạo đi lướt qua xem với ngóc ngách đó thì liệu có nhìn thấu được nội dung thông điệp đó rồi làm tới hay là lại bị hụt mất (follow it)?
- Ngẫm coi đó xem đang có hiện tồn tại thêm rác cho có bất kỳ mớ thông điệp cùi lũi theo dạng thông cáo báo lỗi mà mơ màng ú ớ (ambiguous instructions) gây ngớ ngẩn rồi chưa có xác định là nào?

### Bài kiểm tra tính năng hoàn tất quy chuẩn chốt hạ (Completeness Test)

- Rốt lại liệu là nó có kịp chuẩn cấp được khoác thêm mảnh phủ cover tới tuốt cho những bản đường bay (happy path) ở mức tiêu chuẩn tối cực kỳ nhất cõi?
- Khâu bao sân hỗ trợ có đủ tầm để lo quản xử ngọn mấy vụ trục giật cặn cựa của cái bến bãi chuyên quản về lỗi phát sinh ngoại tuyến lề mẻ (edge cases) kia luôn chưa vậy?
- Thử xem có chệch hướng mường tượng xem ở mức nào cho giải pháp ngộ nhỡ phát vấp phải vấn đề hễ có sự cố rủi ro theo diện vệt kịch bản sai quấy ngáo ngơ (error scenarios) này lại giải quyết sao đây thì coi cho tròn?

### Bài kiểm tra Phép thử cho sự Hữu ích công đắc dĩ (Usefulness Test)

- Xem kỹ nó có công phu ra được sự tích đủ lực để khều móc rồi mang lại khả năng xử đẹp thầu tốt đi đến tận thu cả một rắc rối phát sinh thực tại do hoàn cảnh gán nợ vô đó (real problem) vậy à ta?
- Chỗ sâu xa bạn có nín thở mà xác quyết nhận định lại tính bằng cách chêm dùng tới thứ bửu bối bộ skill đấy phục vụ trực tiếp ngay trên chính người xài chính của nó trên chính đôi bàn cớ bạn chăng (use this yourself)?
- Rồi vụ đưa chém bằng phương thế cách công lực như này ngó coi thử giúp giảm hao hụt cái chi phí bảo quản cho vụ rãnh (save time) dư dả ra hơn kèm thăng ngút luôn tầm chất độ hoàn kim (improve quality) hỉ?

---

## Truy Tầm và Học hỏi từ các Nhóm Skills có Hàng dạt dào

### Yêu cầu Bạn cất bút và Ngâm Cứu Ngay Các Bộ Mẫu này đê (Study These Examples)

**Phân bổ chuyên trách lo phục Dành Cho khối Người Mới Nâng Tầm (Beginners):**

- `skills/brainstorming/SKILL.md` - Kiến thiết cho ra một Cấu trúc cực định hình theo phom (Clear structure) không mảy may gợn bợn
- `skills/git-pushing/SKILL.md` - Đơn giản như đang giỡn thôi gỡ bện (Simple) mà đập mạnh điểm mù đúng yếu điểm đánh lõi vỡ tung cái khung (focused)
- `skills/copywriting/SKILL.md` - Điểm nhấn cho bạn trọn bộ món Good examples

**Hướng đích cự li chuyên lo Phục Mạng tới Dành hẳn Cho Khối tinh binh (Advanced):**

- `skills/systematic-debugging/SKILL.md` - Món này thì toàn trị Toàn cảnh thầu cực kỳ Siêu đa nhiệm bủa vây mọi phía (Comprehensive)
- `skills/react-best-practices/SKILL.md` - Khai mở trò xóc lồng liên thông đa dòng đa tuyến nhuyễn nhừ thao tác Vận dụng qua rất cực nhiều nhị nguyên kết tủa (Multiple files) phối nhau
- `skills/loki-mode/SKILL.md` - Loại này là trùm Phân mảng bẻ nhọn có cái nọc độc quá hể siêu cấp rồi thuộc vô hạng mục quá chật vật cực đành hanh siêu phàm cấp độ cợt nhả một loại Complex workflows

---

## 💡 Ngắn mớm vài mánh hay cho Pro nè (Pro Tips)

1. **Hô xung phong đánh tiên phong đánh tréo vào phần "When to Use"** - Chỗ này thì đăm đăm mài một dao cho trọn cái vụ mục đích chí cốt mang theo cho kỳ ra hồn gắt cực việt purpose của cái mã skill nằm đâu
2. **Lo thả thính Viết các mớ bộ examples chen ngay vào trúng trước tiên thì kệ ráo nhe (Write examples first)** - Phải thừa là trò này mới trổ ra được sự minh chững đúng cái định vị hướng theo lối đi cho não của bạn tỏ ra coi bạn thực sự gài đặt vô cái thứ để làm rõ cái đích điều mình đang cống tâm nhằm vô truyền giảng có nhắm là gì rồi đấy mới chắc
3. **Phải luôn Test tay chạy đối ứng cho máy coi rồi ấp vô test trên một công cụ AI trợ sức có sẵn để coi** - Phải tận tay nhắm mở dán mặt cho kỹ coi con này nó có thực sự kích chạy cho mình (actually works) trước thềm khi hạ bút ký rụp giao phó mảng đem submit chứ
4. **Lo xin ban phát cho vài miếng ngắm xem đánh giá từ giời đất ạ (Get feedback)** - Nhờ rê tụi anh em ra đường cho tụi nó đi ngó soi mói lại rồi cớ coi cho trót mấy cái bài phê mặn nhạt dồn dập vào đọc để review kỹ bộ rễ skill của ranh mình đan đó thử nè
5. **Biết Cải biến theo cơ vẫy và không nãn lòng Lặp đi Lặp lại cái trò mài dao vát kiếm này nghe (Iterate)** - Cần xác lập não hiểu rằng đồ thị cho cấu tứ mảng dính với Skills vốn thuộc dòng dõi sinh sự rồi lại còn hay tự động dần dần cải tà quy chánh định lượng tăng điểm độ chất lượng hoàn thiện (improve) lột xác thành khủng qua từng ngóc thời gian phũ vờn dập của số thông độ bám qua chỉ tiêu xài hoang (usage) thì mới khấm khá thành danh lên nghen mầy!

---

## Có những cái Vũng mà nó Ngáo ngáo hay Phá mình thì phải lo Né gấp nha (Common Mistakes to Avoid)

### ❌ Có Ngơ số 1 (Mistake 1): Cứ làm mơ mơ màng màng mờ mờ mịt mịt đui con mắt (Too Vague)

```markdown
## Instructions

Quắn léo nào móp vô đánh giúp cho cho cái đám bộ code chỗ kia kia ngon hơn tí đi.
```

**✅ Sửa ngay rụp cho đúng thành (Fix):**

```markdown
## Instructions

1. Dùng búa gõ ép đẩy chẻ luôn xuất tinh cái phần logic bôi nhây (repeated) bóc lột nó qua thành những hàm củ cấu trúc là functions phân chia cục chịch
2. Nhồi nắn ghim đập bồi thêm cơ chế hệ thống che lưng vạ lấy những quả nhắc vạ ngớ báo lỗi chặn tay chực (error handling) chặn cả mọi nấm mồ xui rủi của tụi trâu nhọ (edge cases)
3. Làm trò trói và bắt gộp lại đi tiến vào hành quy trình để viết bộ kịch bản test bằng unit tests vây hãm quét cho nham nháp toàn vùng cho đám thuộc khối hạt năng lượng hệ core functionality lòi tói kia luôn một luồng nhe
```

### ❌ Cục Lỗi Chình ình số 2 (Mistake 2): Tham mưu Quá trớn lố cỡ tới tầm Phức tạp hắc não (Too Complex)

```markdown
## Instructions

[Bưng thúng đổ 5000 ký tự rặc một dòng rặt đặc chữ phồn hoa mang tính từ ngứa mồm nói trúng thuật đặc sệt tiếng rên rỉ rêu rạo tiếng chuyên ngành (dense technical jargon) sặc gạch như ông sư đang tụng]
```

**✅ Sửa trúng não lấp thành vầy nhe (Fix):**
Mất gì khỏi phải nhúng xỉn cái dao chặt Break ra nát nhỏ xài những cú miết chi ra làm nùi nùi vài trò vụn thành cấu kiện đồ chẻ ra kiểu Multiple skills hay cứ ngóc ngỏi lên xài thủ cái trò nhá nhá hàng chiêu ngấm từ chập từ cấp bóc phốt mẩu vụn cho nó progressive disclosure xào qua nấu lạt gì cũng tiện xớ hơn không.

### ❌ Cục rụng rốn phắn ôi Mọi thứ 3 (Mistake 3): Tịt Nóc Đếch Chừa Cho Rớt Mảnh Ví dụ Cho 1 Góc Cụ thể Gì ráo Chọi Để Ăn Cho Biết Nhập Sóng (No Examples)

```markdown
## Instructions

[Hù u một nùi cái bộ kẹt bẫy bao nhiêu cũng không thoát chỉ dẫn hướng này kia làm sao búa léo rào cho sôm xong rồi nhưng quái quỷ đéo đưa cho cái miếng gì lấy 1 đoạn code mẫu thì ăn khói cặc à]
```

**✅ Đập rồi xây lại bằng này coi sao nghe mày (Fix):**
Không mớ gì bét gã nhét 1 đống xài đại quăng mẹ cho tầm nổ trên cỡ tầm 2-3 phần ví vụ đính cái tag hạng real-world examples chân thực ra cho ngỏ sáng mâm nhe

### ❌ Cái lỗi đốn mạt cuối cùng thứ 4 (Mistake 4): Không Đi Kèm Gì Thì Coi Như Hàng Ế Nhảm Chuyện Rồi Đồ Quê Có Thiếu Cập Nhật Bộ Não Liệu Thông Sự Trì Cho Phé Mùi Khắm (Outdated Information)

```markdown
Chỗ lủng đống React class components cổ đú chán này tao móc đít...
```

**✅ Gõ lủng chạc vô đây lấy lại (Fix):**
Đừng khờ đôn nhầm qua ba cái rác mà phai chịu bảo cho dòng đời duy trì bộ dáng cập nhật đàng hoàng ráo hoảnh cái chuẩn Keep skills updated bấu sát sao rọi theo ba nhóm tiêu chuẩn lề thói mới tanh thời hiện hãn dành cho trọn đám Best practices cho lòi mắt vô nhá

---

## 🎯 Chỉ Điểm Các Tọa Đoạn Đặng Để Lết Lết Bước Ra Nước Kế Đây (Next Steps)

1. **Rán nuốt đi đọc dăm chừng vào độ 3-5 existing skills** rà soát ngó cho nẹt ngứa tròng để bóc lột cho rõ hình khối định dạng ở theo thể điệu đa phong cách (style) nhuyễn 1 phát
2. **Lo quào chà vô mò lổm ngổm vọc bộ vạch cho chửa bung ra có được cái skill template** thó ngó mượn lấp liếm tạm ngoài chỗ thư viện sảnh cái file bóc lột nó lấy tên là CONTRIBUTING.md coi chừng bị chập
3. **Thăm ngó dồn tụ trí não xoay vần dồn hết lên gân bằng tìm cái cách Create tạo hình lên lựng 1 nhãn thành mẩu một simple skill** chọc 1 nùi vào có xới cho đúng được 1 cái ngách dạt mà cái hệ sở trường cái gì ông nẳm thóp nhẵn sành của chính mặt của ông ấy à nha
4. **Vây bắt rải trận Chuyển gấp rút nhồi ngay vào tiến vào quá trình Test đồ** đi trơn cặp đôi không thoát để mà thông vút với con nhỏ đệ của ngài là gã anh em AI assistant nhắm cho đã
5. **Nhanh nhẹn quất tung cái món đấy Lan toả dồn dồn luồn sang tới đây cho tui chụm tụ cho nha (Share it)** nhờ ba cớ xài ba cái trò Pull Request vát ra cho coi được 1 tí héng ông

---

**Nhớ cho một Điều khắc nhốt Này (Remember):** Lũ chóp bu expert múa rều khua mõ tới lui rốt cục cái củi mả mẹ nguyên căn cũng vạn sự bắt nguồn cho dính gốc xuất sinh do chỉ một mình tự ông tổ từ thuở anh lính te tò lò đò cho là hạng tép riu sơ sinh (beginner) ráo trụi phàm đi cả. Bước trườn từ tốn chậm thật chậm Start simple, học hỏi mút trọn cả núi cống hiến cho xài ké nhờ qua mớ feedback hòng khuyên lơn, rồi gồng ỉa phân cứt nhả cống hiến cái sự nỗ lực làm sao nhằm cày cuốc cải thiện năng lực (improve) thầm bón đúc nhích đi tươm dần qua số vần con tính vạch dài của quỹ một thời gian đó ông kẹ ả! 🚀
