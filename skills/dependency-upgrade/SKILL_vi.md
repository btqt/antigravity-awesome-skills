---
name: dependency-upgrade
description: Quản lý nâng cấp phiên bản phụ thuộc chính với phân tích tương thích, triển khai theo giai đoạn và kiểm thử toàn diện. Sử dụng khi nâng cấp phiên bản framework, cập nhật các phụ thuộc chính hoặc quản lý các thay đổi phá vỡ trong thư viện.
---

# Nâng Cấp Phụ Thuộc (Dependency Upgrade)

Làm chủ việc nâng cấp phiên bản phụ thuộc chính, phân tích tương thích, chiến lược nâng cấp theo giai đoạn và các cách tiếp cận kiểm thử toàn diện.

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến nâng cấp phụ thuộc
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Sử dụng kỹ năng này khi

- Nâng cấp các phiên bản framework chính
- Cập nhật các phụ thuộc có lỗ hổng bảo mật
- Hiện đại hóa các phụ thuộc cũ
- Giải quyết xung đột phụ thuộc
- Lập kế hoạch lộ trình nâng cấp gia tăng
- Kiểm thử ma trận tương thích
- Tự động hóa cập nhật phụ thuộc

## Đánh giá Phiên bản Ngữ nghĩa (Semantic Versioning)

```
MAJOR.MINOR.PATCH (ví dụ: 2.3.1)

MAJOR: Thay đổi phá vỡ (Breaking changes)
MINOR: Tính năng mới, tương thích lùi
PATCH: Sửa lỗi, tương thích lùi

^2.3.1 = >=2.3.1 <3.0.0 (cập nhật minor)
~2.3.1 = >=2.3.1 <2.4.0 (cập nhật patch)
2.3.1 = phiên bản chính xác
```

## Phân tích Phụ thuộc

### Kiểm toán Phụ thuộc

```bash
# npm
npm outdated
npm audit
npm audit fix

# yarn
yarn outdated
yarn audit

# Kiểm tra cập nhật chính
npx npm-check-updates
npx npm-check-updates -u  # Update package.json
```

### Phân tích Cây Phụ thuộc

```bash
# Xem tại sao một gói được cài đặt
npm ls package-name
yarn why package-name

# Tìm các gói trùng lặp
npm dedupe
yarn dedupe

# Trực quan hóa phụ thuộc
npx madge --image graph.png src/
```

## Ma trận Tương thích

```javascript
// compatibility-matrix.js
const compatibilityMatrix = {
  react: {
    "16.x": {
      "react-dom": "^16.0.0",
      "react-router-dom": "^5.0.0",
      "@testing-library/react": "^11.0.0",
    },
    "17.x": {
      "react-dom": "^17.0.0",
      "react-router-dom": "^5.0.0 || ^6.0.0",
      "@testing-library/react": "^12.0.0",
    },
    "18.x": {
      "react-dom": "^18.0.0",
      "react-router-dom": "^6.0.0",
      "@testing-library/react": "^13.0.0",
    },
  },
};

function checkCompatibility(packages) {
  // Validate package versions against matrix
}
```

## Chiến lược Nâng cấp theo Giai đoạn

### Giai đoạn 1: Lập kế hoạch

```bash
# 1. Xác định phiên bản hiện tại
npm list --depth=0

# 2. Kiểm tra thay đổi phá vỡ
# Đọc CHANGELOG.md và MIGRATION.md

# 3. Tạo kế hoạch nâng cấp
echo "Upgrade order:
1. TypeScript
2. React
3. React Router
4. Testing libraries
5. Build tools" > UPGRADE_PLAN.md
```

### Giai đoạn 2: Cập nhật Gia tăng

```bash
# Đừng nâng cấp mọi thứ cùng một lúc!

# Bước 1: Cập nhật TypeScript
npm install typescript@latest

# Kiểm thử
npm run test
npm run build

# Bước 2: Cập nhật React (mỗi lần một phiên bản chính)
npm install react@17 react-dom@17

# Kiểm thử lại
npm run test

# Bước 3: Tiếp tục với các gói khác
npm install react-router-dom@6

# Và cứ thế tiếp tục...
```

### Giai đoạn 3: Xác thực

```javascript
// tests/compatibility.test.js
describe("Dependency Compatibility", () => {
  it("should have compatible React versions", () => {
    const reactVersion = require("react/package.json").version;
    const reactDomVersion = require("react-dom/package.json").version;

    expect(reactVersion).toBe(reactDomVersion);
  });

  it("should not have peer dependency warnings", () => {
    // Run npm ls and check for warnings
  });
});
```

## Xử lý Thay đổi Phá vỡ

### Xác định Thay đổi Phá vỡ

```bash
# Sử dụng trình phân tích changelog
npx changelog-parser react 16.0.0 17.0.0

# Hoặc kiểm tra thủ công
curl https://raw.githubusercontent.com/facebook/react/main/CHANGELOG.md
```

### Codemod cho Sửa lỗi Tự động

```bash
# Codemod nâng cấp React
npx react-codeshift <transform> <path>

# Ví dụ: Cập nhật phương thức vòng đời
npx react-codeshift \
  --parser tsx \
  --transform react-codeshift/transforms/rename-unsafe-lifecycles.js \
  src/
```

### Tài nguyên

- **references/semver.md**: Hướng dẫn phiên bản ngữ nghĩa
- **references/compatibility-matrix.md**: Các vấn đề tương thích phổ biến
- **references/staged-upgrades.md**: Chiến lược nâng cấp gia tăng
- **references/testing-strategy.md**: Các cách tiếp cận kiểm thử toàn diện
- **assets/upgrade-checklist.md**: Danh sách kiểm tra từng bước
- **assets/compatibility-matrix.csv**: Bảng tương thích phiên bản
- **scripts/audit-dependencies.sh**: Tập lệnh kiểm toán phụ thuộc

## Thực tiễn Tốt nhất

1. **Đọc Changelogs**: Hiểu những gì đã thay đổi
2. **Nâng cấp Gia tăng**: Mỗi lần một phiên bản chính
3. **Kiểm thử Kỹ lưỡng**: Unit, integration, E2E tests
4. **Kiểm tra Peer Dependencies**: Giải quyết xung đột sớm
5. **Sử dụng Lock Files**: Đảm bảo cài đặt có thể tái tạo
6. **Tự động hóa Cập nhật**: Sử dụng Renovate hoặc Dependabot
7. **Giám sát**: Theo dõi lỗi thời gian chạy sau khi nâng cấp
8. **Tài liệu hóa**: Giữ ghi chú nâng cấp

## Các Cạm bẫy Phổ biến

- Nâng cấp tất cả phụ thuộc cùng một lúc
- Không kiểm thử sau mỗi lần nâng cấp
- Bỏ qua cảnh báo peer dependency
- Quên cập nhật lock file
- Không đọc ghi chú thay đổi phá vỡ
- Bỏ qua các phiên bản chính
- Không có kế hoạch rollback
