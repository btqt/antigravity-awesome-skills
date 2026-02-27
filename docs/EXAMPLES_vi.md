# 🧪 Ví dụ Thực tế ("The Antigravity Cookbook")

Các skills vốn dĩ đã rất mạnh mẽ khi hoạt động độc lập, nhưng sẽ trở nên không thể cản bước khi được kết hợp với nhau.
Dưới đây là ba tình huống phổ biến và cách giải quyết chúng bằng cách sử dụng repository này.

## 🥘 Recipe 1: The "Legacy Code Audit"

_Scenario: Bạn vừa tiếp quản một repo Node.js đã 5 năm tuổi rườm rà. Bạn cần sửa chữa nó một cách an toàn._

**Các Skills được sử dụng:**

1. `concise-planning` (Để vạch ra kiến trúc các phần lộn xộn)
2. `lint-and-validate` (Để tìm ra các lỗi bugs)
3. `security-audit` (Để tìm ra các lỗ hổng bảo mật)

**The Workflow:**

1. **Plan**: "Agent, hãy sử dụng `concise-planning` để tạo một checklist cho việc refactoring `src/legacy-api.js`."
2. **Audit**: "Chạy `security-audit` trên tệp `package.json` để tìm các dependencies có lỗ hổng bảo mật."
3. **Fix**: "Sử dụng các rules của `lint-and-validate` để tự động sửa chữa (auto-fix) các vấn đề formatting trong `src/`."

---

## 🥘 Recipe 2: The "Modern Web App"

_Scenario: Bạn cần xây dựng một Landing Page với hiệu suất cao trong vòng 2 giờ._

**Các Skills được sử dụng:**

1. `frontend-design` (Đảm bảo tính thẩm mỹ)
2. `react-patterns` (Xây dựng cấu trúc)
3. `tailwind-mastery` (Tối ưu tốc độ phát triển)

**The Workflow:**

1. **Design**: "Sử dụng `frontend-design` để tạo bảng màu (color palette) và typography cho một 'Cyberpunk Coffee Shop'."
2. **Scaffold**: "Khởi tạo một project Vite. Sau đó áp dụng `react-patterns` để tạo 'Hero' component."
3. **Style**: "Sử dụng `tailwind-mastery` để làm cho các buttons có hiệu ứng glassmorphic và responsive."

---

## 🥘 Recipe 3: The "Agent Architect"

_Scenario: Bạn muốn xây dựng một AI agent tùy chỉnh có khả năng tự kiểm chứng (verify) code của chính nó._

**Các Skills được sử dụng:**

1. `mcp-builder` (Để xây dựng các tools)
2. `agent-evaluation` (Để kiểm thử độ tin cậy)
3. `prompt-engineering` (Để tinh chỉnh các chỉ dẫn - instructions)

**The Workflow:**

1. **Build**: "Sử dụng `mcp-builder` để tạo ra một tool tên là `verify-file`."
2. **Instruct**: "Áp dụng các patterns của `prompt-engineering` vào System Prompt để agent luôn luôn phải kiểm tra các file paths."
3. **Test**: "Chạy `agent-evaluation` để benchmark tần suất agent thất bại trong việc tìm kiếm file."
