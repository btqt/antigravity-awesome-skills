# Sổ tay Triển khai Kiểm tra và Thử nghiệm Khả năng Truy cập (Accessibility)

Tệp này chứa các mẫu chi tiết, danh sách kiểm tra và các ví dụ mã nguồn được tham chiếu bởi kỹ năng.

## Hướng dẫn

### 1. Thử nghiệm Tự động với axe-core

```javascript
// accessibility-test.js
const { AxePuppeteer } = require("@axe-core/puppeteer");
const puppeteer = require("puppeteer");

class AccessibilityAuditor {
  constructor(options = {}) {
    this.wcagLevel = options.wcagLevel || "AA";
    this.viewport = options.viewport || { width: 1920, height: 1080 };
  }

  async runFullAudit(url) {
    const browser = await puppeteer.launch();
    const page = await browser.newPage();
    await page.setViewport(this.viewport);
    await page.goto(url, { waitUntil: "networkidle2" });

    const results = await new AxePuppeteer(page)
      .withTags(["wcag2a", "wcag2aa", "wcag21a", "wcag21aa"])
      .exclude(".no-a11y-check")
      .analyze();

    await browser.close();

    return {
      url,
      timestamp: new Date().toISOString(),
      violations: results.violations.map((v) => ({
        id: v.id,
        impact: v.impact,
        description: v.description,
        help: v.help,
        helpUrl: v.helpUrl,
        nodes: v.nodes.map((n) => ({
          html: n.html,
          target: n.target,
          failureSummary: n.failureSummary,
        })),
      })),
      score: this.calculateScore(results),
    };
  }

  calculateScore(results) {
    const weights = { critical: 10, serious: 5, moderate: 2, minor: 1 };
    let totalWeight = 0;
    results.violations.forEach((v) => {
      totalWeight += weights[v.impact] || 0;
    });
    return Math.max(0, 100 - totalWeight);
  }
}

// Kiểm thử thành phần với jest-axe
import { render } from "@testing-library/react";
import { axe, toHaveNoViolations } from "jest-axe";

expect.extend(toHaveNoViolations);

describe("Accessibility Tests", () => {
  it("nên không có lỗi vi phạm", async () => {
    const { container } = render(<MyComponent />);
    const results = await axe(container);
    expect(results).toHaveNoViolations();
  });
});
```

### 2. Xác thực Độ tương phản Màu sắc

```javascript
// color-contrast.js
class ColorContrastAnalyzer {
    constructor() {
        this.wcagLevels = {
            'AA': { normal: 4.5, large: 3 },
            'AAA': { normal: 7, large: 4.5 }
        };
    }

    async analyzePageContrast(page) {
        const elements = await page.evaluate(() => {
            return Array.from(document.querySelectorAll('*'))
                .filter(el => el.innerText && el.innerText.trim())
                .map(el => {
                    const styles = window.getComputedStyle(el);
                    return {
                        text: el.innerText.trim().substring(0, 50),
                        color: styles.color,
                        backgroundColor: styles.backgroundColor,
                        fontSize: parseFloat(styles.fontSize),
                        fontWeight: styles.fontWeight
                    };
                });
        });

        return elements
            .map(el => {
                const contrast = this.calculateContrast(el.color, el.backgroundColor);
                const isLarge = this.isLargeText(el.fontSize, el.fontWeight);
                const required = isLarge ? this.wcagLevels.AA.large : this.wcagLevels.AA.normal;

                if (contrast < required) {
                    return {
                        text: el.text,
                        currentContrast: contrast.toFixed(2),
                        requiredContrast: required,
                        foreground: el.color,
                        background: el.backgroundColor
                    };
                }
                return null;
            })
            .filter(Boolean);
    }

    calculateContrast(fg, bg) {
        const l1 = this.relativeLuminance(this.parseColor(fg));
        const l2 = this.relativeLuminance(this.parseColor(bg));
        const lighter = Math.max(l1, l2);
        const darker = Math.min(l1, l2);
        return (lighter + 0.05) / (darker + 0.05);
    }

    relativeLuminance(rgb) {
        const [r, g, b] = rgb.map(val => {
            val = val / 255;
            return val <= 0.03928 ? val / 12.92 : Math.pow((val + 0.055) / 1.055, 2.4);
        });
        return 0.2126 * r + 0.7152 * g + 0.0722 * b;
    }
}

// CSS độ tương phản cao
@media (prefers-contrast: high) {
    :root {
        --text-primary: #000;
        --bg-primary: #fff;
        --border-color: #000;
    }
    a { text-decoration: underline !important; }
    button, input { border: 2px solid var(--border-color) !important; }
}
```

### 3. Kiểm thử Điều hướng bằng Bàn phím

```javascript
// keyboard-navigation.js
class KeyboardNavigationTester {
  async testKeyboardNavigation(page) {
    const results = {
      focusableElements: [],
      missingFocusIndicators: [],
      keyboardTraps: [],
    };

    // Lấy tất cả các thành phần có thể lấy tiêu điểm (focusable)
    const focusable = await page.evaluate(() => {
      const selector =
        'a[href], button, input, select, textarea, [tabindex]:not([tabindex="-1"])';
      return Array.from(document.querySelectorAll(selector)).map((el) => ({
        tagName: el.tagName.toLowerCase(),
        text: el.innerText || el.value || el.placeholder || "",
        tabIndex: el.tabIndex,
      }));
    });

    results.focusableElements = focusable;

    // Kiểm tra thứ tự Tab và chỉ báo tiêu điểm (focus indicator)
    for (let i = 0; i < focusable.length; i++) {
      await page.keyboard.press("Tab");

      const focused = await page.evaluate(() => {
        const el = document.activeElement;
        return {
          tagName: el.tagName.toLowerCase(),
          hasFocusIndicator: window.getComputedStyle(el).outline !== "none",
        };
      });

      if (!focused.hasFocusIndicator) {
        results.missingFocusIndicators.push(focused);
      }
    }

    return results;
  }
}

// Tăng cường khả năng truy cập bằng bàn phím
document.addEventListener("keydown", (e) => {
  if (e.key === "Escape") {
    const modal = document.querySelector(".modal.open");
    if (modal) closeModal(modal);
  }
});

// Làm cho div có thể click trở nên truy cập được
document.querySelectorAll("[onclick]").forEach((el) => {
  if (!["a", "button", "input"].includes(el.tagName.toLowerCase())) {
    el.setAttribute("tabindex", "0");
    el.setAttribute("role", "button");
    el.addEventListener("keydown", (e) => {
      if (e.key === "Enter" || e.key === " ") {
        el.click();
        e.preventDefault();
      }
    });
  }
});
```

### 4. Kiểm thử với Trình đọc màn hình (Screen Reader)

```javascript
// screen-reader-test.js
class ScreenReaderTester {
  async testScreenReaderCompatibility(page) {
    return {
      landmarks: await this.testLandmarks(page),
      headings: await this.testHeadingStructure(page),
      images: await this.testImageAccessibility(page),
      forms: await this.testFormAccessibility(page),
    };
  }

  async testHeadingStructure(page) {
    const headings = await page.evaluate(() => {
      return Array.from(
        document.querySelectorAll("h1, h2, h3, h4, h5, h6"),
      ).map((h) => ({
        level: parseInt(h.tagName[1]),
        text: h.textContent.trim(),
        isEmpty: !h.textContent.trim(),
      }));
    });

    const issues = [];
    let previousLevel = 0;

    headings.forEach((heading, index) => {
      if (heading.level > previousLevel + 1 && previousLevel !== 0) {
        issues.push({
          type: "skipped-level",
          message: `Cấp độ tiêu đề ${heading.level} nhảy từ cấp độ ${previousLevel}`,
        });
      }
      if (heading.isEmpty) {
        issues.push({ type: "empty-heading", index });
      }
      previousLevel = heading.level;
    });

    if (!headings.some((h) => h.level === 1)) {
      issues.push({ type: "missing-h1", message: "Trang thiếu thành phần h1" });
    }

    return { headings, issues };
  }

  async testFormAccessibility(page) {
    const forms = await page.evaluate(() => {
      return Array.from(document.querySelectorAll("form")).map((form) => {
        const inputs = form.querySelectorAll("input, textarea, select");
        return {
          fields: Array.from(inputs).map((input) => ({
            type: input.type || input.tagName.toLowerCase(),
            id: input.id,
            hasLabel: input.id
              ? !!document.querySelector(`label[for="${input.id}"]`)
              : !!input.closest("label"),
            hasAriaLabel: !!input.getAttribute("aria-label"),
            required: input.required,
          })),
        };
      });
    });

    const issues = [];
    forms.forEach((form, i) => {
      form.fields.forEach((field, j) => {
        if (!field.hasLabel && !field.hasAriaLabel) {
          issues.push({ type: "missing-label", form: i, field: j });
        }
      });
    });

    return { forms, issues };
  }
}

// Các mẫu ARIA
const ariaPatterns = {
  modal: `
<div role="dialog" aria-labelledby="modal-title" aria-modal="true">
    <h2 id="modal-title">Tiêu đề Modal</h2>
    <button aria-label="Đóng">×</button>
</div>`,

  tabs: `
<div role="tablist" aria-label="Điều hướng">
    <button role="tab" aria-selected="true" aria-controls="panel-1">Tab 1</button>
</div>
<div role="tabpanel" id="panel-1" aria-labelledby="tab-1">Nội dung</div>`,

  form: `
<label for="name">Tên <span aria-label="bắt buộc">*</span></label>
<input id="name" required aria-required="true" aria-describedby="name-error">
<span id="name-error" role="alert" aria-live="polite"></span>`,
};
```

### 5. Danh sách Kiểm tra Thử nghiệm Thủ công

```markdown
## Thử nghiệm Khả năng Truy cập Thủ công

### Điều hướng bằng Bàn phím

- [ ] Tất cả các thành phần tương tác đều có thể truy cập qua Tab
- [ ] Các nút được kích hoạt bằng Enter/Space
- [ ] Phím Esc đóng các modal
- [ ] Chỉ báo tiêu điểm (focus indicator) luôn hiển thị
- [ ] Không có bẫy bàn phím (keyboard traps)
- [ ] Thứ tự Tab logic

### Trình đọc màn hình

- [ ] Tiêu đề trang có tính mô tả
- [ ] Các tiêu đề tạo ra cấu trúc logic (outline)
- [ ] Hình ảnh có văn bản thay thế (alt text)
- [ ] Các trường biểu mẫu có nhãn (label)
- [ ] Thông báo lỗi được đọc lên
- [ ] Các cập nhật động được thông báo

### Thị giác

- [ ] Văn bản có thể thay đổi kích thước lên 200% mà không bị mất nội dung
- [ ] Màu sắc không phải là phương tiện duy nhất để truyền tải thông tin
- [ ] Chỉ báo tiêu điểm có đủ độ tương phản
- [ ] Nội dung tự điều chỉnh (reflow) ở độ rộng 320px
- [ ] Các hiệu ứng hoạt ảnh có thể tạm dừng được

### Nhận thức

- [ ] Hướng dẫn rõ ràng và đơn giản
- [ ] Thông báo lỗi hữu ích
- [ ] Không giới hạn thời gian trên các biểu mẫu
- [ ] Điều hướng nhất quán
- [ ] Các hành động quan trọng có thể hoàn tác
```

### 6. Các Ví dụ Khắc phục (Remediation)

```javascript
// Sửa lỗi thiếu văn bản thay thế (alt text)
document.querySelectorAll("img:not([alt])").forEach((img) => {
  const isDecorative =
    img.role === "presentation" || img.closest('[role="presentation"]');
  img.setAttribute("alt", isDecorative ? "" : img.title || "Hình ảnh");
});

// Sửa lỗi thiếu nhãn (label)
document
  .querySelectorAll("input:not([aria-label]):not([id])")
  .forEach((input) => {
    if (input.placeholder) {
      input.setAttribute("aria-label", input.placeholder);
    }
  });

// Các thành phần React có khả năng truy cập
const AccessibleButton = ({ children, onClick, ariaLabel, ...props }) => (
  <button onClick={onClick} aria-label={ariaLabel} {...props}>
    {children}
  </button>
);

const LiveRegion = ({ message, politeness = "polite" }) => (
  <div
    role="status"
    aria-live={politeness}
    aria-atomic="true"
    className="sr-only"
  >
    {message}
  </div>
);
```

### 7. Tích hợp CI/CD

```yaml
# .github/workflows/accessibility.yml
name: Accessibility Tests

on: [push, pull_request]

jobs:
  a11y-tests:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3

      - name: Thiết lập Node.js
        uses: actions/setup-node@v3
        with:
          node-version: "18"

      - name: Cài đặt và build
        run: |
          npm ci
          npm run build

      - name: Khởi động server
        run: |
          npm start &
          npx wait-on http://localhost:3000

      - name: Chạy axe tests
        run: npm run test:a11y

      - name: Chạy pa11y
        run: npx pa11y http://localhost:3000 --standard WCAG2AA --threshold 0

      - name: Tải báo cáo lên
        uses: actions/upload-artifact@v3
        if: always()
        with:
          name: a11y-report
          path: a11y-report.html
```

### 8. Báo cáo

```javascript
// report-generator.js
class AccessibilityReportGenerator {
  generateHTMLReport(auditResults) {
    return `
<!DOCTYPE html>
<html lang="vi">
<head>
    <title>Kiểm tra Khả năng Truy cập</title>
    <style>
        body { font-family: Arial, sans-serif; margin: 20px; }
        .summary { background: #f0f0f0; padding: 20px; border-radius: 8px; }
        .score { font-size: 48px; font-weight: bold; }
        .violation { margin: 20px 0; padding: 15px; border: 1px solid #ddd; }
        .critical { border-color: #f00; background: #fee; }
        .serious { border-color: #fa0; background: #ffe; }
    </style>
</head>
<body>
    <h1>Báo cáo Kiểm tra Khả năng Truy cập</h1>
    <p>Được tạo lúc: ${new Date().toLocaleString()}</p>

    <div class="summary">
        <h2>Tổng quan</h2>
        <div class="score">${auditResults.score}/100</div>
        <p>Tổng số lỗi vi phạm: ${auditResults.violations.length}</p>
    </div>

    <h2>Các lỗi vi phạm</h2>
    ${auditResults.violations
      .map(
        (v) => `
        <div class="violation ${v.impact}">
            <h3>${v.help}</h3>
            <p><strong>Tác động:</strong> ${v.impact}</p>
            <p>${v.description}</p>
            <a href="${v.helpUrl}">Tìm hiểu thêm</a>
        </div>
    `,
      )
      .join("")}
</body>
</html>`;
  }
}
```

## Định dạng Đầu ra

1. **Điểm số Khả năng Truy cập**: Mức độ tuân thủ tổng thể đối với các cấp độ WCAG
2. **Báo cáo Vi phạm**: Chi tiết các vấn đề cùng mức độ nghiêm trọng và cách sửa
3. **Kết quả Thử nghiệm**: Kết quả thử nghiệm tự động và thủ công
4. **Hướng dẫn Khắc phục**: Các bước sửa lỗi cho từng vấn đề
5. **Ví dụ Mã nguồn**: Triển khai các thành phần có khả năng truy cập

Tập trung vào việc tạo ra các trải nghiệm hòa nhập hoạt động tốt cho tất cả người dùng, bất kể khả năng hoặc công nghệ hỗ trợ mà họ sử dụng.
