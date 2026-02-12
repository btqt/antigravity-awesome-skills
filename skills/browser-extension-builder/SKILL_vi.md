---
name: browser-extension-builder
description: "Chuyên gia xây dựng các tiện ích mở rộng trình duyệt giải quyết các vấn đề thực tế - Tiện ích mở rộng Chrome, Firefox và đa trình duyệt. Bao gồm kiến trúc tiện ích mở rộng, manifest v3, content scripts, giao diện popup, chiến lược kiếm tiền và xuất bản lên Chrome Web Store. Sử dụng khi: tiện ích mở rộng trình duyệt, tiện ích mở rộng chrome, addon firefox, tiện ích mở rộng, manifest v3."
source: vibeship-spawner-skills (Apache 2.0)
---

# Xây dựng Tiện ích mở rộng Trình duyệt

**Vai trò**: Kiến trúc sư Tiện ích mở rộng Trình duyệt

Bạn mở rộng trình duyệt để cung cấp cho người dùng siêu năng lực. Bạn hiểu những hạn chế độc đáo của việc phát triển tiện ích mở rộng - quyền hạn, bảo mật, chính sách cửa hàng. Bạn xây dựng các tiện ích mở rộng mà mọi người cài đặt và thực sự sử dụng hàng ngày. Bạn biết sự khác biệt giữa một món đồ chơi và một công cụ.

## Khả năng

- Kiến trúc tiện ích mở rộng (Extension architecture)
- Manifest v3 (MV3)
- Content scripts
- Background workers
- Giao diện Popup
- Kiếm tiền từ tiện ích mở rộng
- Xuất bản lên Chrome Web Store
- Hỗ trợ đa trình duyệt

## Các Mẫu (Patterns)

### Kiến trúc Tiện ích mở rộng

Cấu trúc cho các tiện ích mở rộng trình duyệt hiện đại

**Khi nào sử dụng**: Khi bắt đầu một tiện ích mở rộng mới

```javascript
## Extension Architecture

### Project Structure
```

extension/
├── manifest.json # Cấu hình tiện ích mở rộng
├── popup/
│ ├── popup.html # Giao diện Popup
│ ├── popup.css
│ └── popup.js
├── content/
│ └── content.js # Chạy trên các trang web
├── background/
│ └── service-worker.js # Logic nền
├── options/
│ ├── options.html # Trang cài đặt
│ └── options.js
└── icons/
├── icon16.png
├── icon48.png
└── icon128.png

````

### Manifest V3 Template
```json
{
  "manifest_version": 3,
  "name": "My Extension",
  "version": "1.0.0",
  "description": "What it does",
  "permissions": ["storage", "activeTab"],
  "action": {
    "default_popup": "popup/popup.html",
    "default_icon": {
      "16": "icons/icon16.png",
      "48": "icons/icon48.png",
      "128": "icons/icon128.png"
    }
  },
  "content_scripts": [{
    "matches": ["<all_urls>"],
    "js": ["content/content.js"]
  }],
  "background": {
    "service_worker": "background/service-worker.js"
  },
  "options_page": "options/options.html"
}
````

### Mô hình Giao tiếp

```
Popup ←→ Background (Service Worker) ←→ Content Script
              ↓
        chrome.storage
```

````

### Content Scripts

Mã chạy trên các trang web

**Khi nào sử dụng**: Khi sửa đổi hoặc đọc nội dung trang

```javascript
## Content Scripts

### Basic Content Script
```javascript
// content.js - Chạy trên mọi trang khớp

// Chờ trang tải
document.addEventListener('DOMContentLoaded', () => {
  // Sửa đổi trang
  const element = document.querySelector('.target');
  if (element) {
    element.style.backgroundColor = 'yellow';
  }
});

// Lắng nghe tin nhắn từ popup/background
chrome.runtime.onMessage.addListener((message, sender, sendResponse) => {
  if (message.action === 'getData') {
    const data = document.querySelector('.data')?.textContent;
    sendResponse({ data });
  }
  return true; // Giữ kênh mở cho async
});
````

### Injecting UI

```javascript
// Tạo UI nổi trên trang
function injectUI() {
  const container = document.createElement("div");
  container.id = "my-extension-ui";
  container.innerHTML = `
    <div style="position: fixed; bottom: 20px; right: 20px;
                background: white; padding: 16px; border-radius: 8px;
                box-shadow: 0 4px 12px rgba(0,0,0,0.15); z-index: 10000;">
      <h3>My Extension</h3>
      <button id="my-extension-btn">Click me</button>
    </div>
  `;
  document.body.appendChild(container);

  document.getElementById("my-extension-btn").addEventListener("click", () => {
    // Xử lý click
  });
}

injectUI();
```

### Permissions for Content Scripts

```json
{
  "content_scripts": [
    {
      "matches": ["https://specific-site.com/*"],
      "js": ["content.js"],
      "run_at": "document_end"
    }
  ]
}
```

````

### Lưu trữ và Trạng thái

Lưu giữ dữ liệu tiện ích mở rộng

**Khi nào sử dụng**: Khi lưu cài đặt người dùng hoặc dữ liệu

```javascript
## Storage and State

### Chrome Storage API
```javascript
// Lưu dữ liệu
chrome.storage.local.set({ key: 'value' }, () => {
  console.log('Saved');
});

// Lấy dữ liệu
chrome.storage.local.get(['key'], (result) => {
  console.log(result.key);
});

// Đồng bộ lưu trữ (đồng bộ qua các thiết bị)
chrome.storage.sync.set({ setting: true });

// Theo dõi thay đổi
chrome.storage.onChanged.addListener((changes, area) => {
  if (changes.key) {
    console.log('key changed:', changes.key.newValue);
  }
});
````

### Storage Limits

| Loại  | Giới hạn                |
| ----- | ----------------------- |
| local | 5MB                     |
| sync  | 100KB tổng, 8KB mỗi mục |

### Async/Await Pattern

```javascript
// Modern async wrapper
async function getStorage(keys) {
  return new Promise((resolve) => {
    chrome.storage.local.get(keys, resolve);
  });
}

async function setStorage(data) {
  return new Promise((resolve) => {
    chrome.storage.local.set(data, resolve);
  });
}

// Sử dụng
const { settings } = await getStorage(["settings"]);
await setStorage({ settings: { ...settings, theme: "dark" } });
```

```

## Các Chống mẫu (Anti-Patterns)

### ❌ Yêu cầu Tất cả Quyền (Requesting All Permissions)

**Tại sao tệ**: Người dùng sẽ không cài đặt.
Cửa hàng có thể từ chối.
Rủi ro bảo mật.
Đánh giá xấu.

**Thay vào đó**: Yêu cầu tối thiểu cần thiết.
Sử dụng quyền tùy chọn.
Giải thích tại sao trong mô tả.
Yêu cầu tại thời điểm sử dụng.

### ❌ Xử lý Nền nặng nề (Heavy Background Processing)

**Tại sao tệ**: MV3 chấm dứt các worker nhàn rỗi.
Hao pin.
Trình duyệt chậm lại.
Người dùng gỡ cài đặt.

**Thay vào đó**: Giữ nền tối thiểu.
Sử dụng alarms cho các tác vụ định kỳ.
Chuyển tải sang content scripts.
Cache tích cực.

### ❌ Hỏng khi Cập nhật (Breaking on Updates)

**Tại sao tệ**: Selectors thay đổi.
API thay đổi.
Người dùng tức giận.
Đánh giá xấu.

**Thay vào đó**: Sử dụng selectors ổn định.
Thêm xử lý lỗi.
Giám sát sự cố.
Cập nhật nhanh chóng khi bị hỏng.

## Các Kỹ năng Liên quan

Hoạt động tốt với: `frontend`, `micro-saas-launcher`, `personal-tool-builder`
```
