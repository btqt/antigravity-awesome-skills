---
name: cc-skill-project-guidelines-example
description: Kỹ năng Hướng dẫn Dự án (Ví dụ)
author: affaan-m
version: "1.0"
---

# Kỹ năng Hướng dẫn Dự án (Ví dụ)

Đây là một ví dụ về kỹ năng cụ thể cho dự án. Sử dụng cái này làm mẫu cho các dự án của riêng bạn.

Dựa trên một ứng dụng sản xuất thực tế: [Zenith](https://zenith.chat) - Nền tảng khám phá khách hàng được hỗ trợ bởi AI.

---

## Khi nào Sử dụng

Tham khảo kỹ năng này khi làm việc trên dự án cụ thể mà nó được thiết kế cho. Các kỹ năng dự án chứa:

- Tổng quan kiến trúc
- Cấu trúc tệp
- Các mẫu mã (code patterns)
- Yêu cầu kiểm thử
- Quy trình triển khai

---

## Tổng quan Kiến trúc

**Tech Stack:**

- **Frontend**: Next.js 15 (App Router), TypeScript, React
- **Backend**: FastAPI (Python), Pydantic models
- **Cơ sở dữ liệu**: Supabase (PostgreSQL)
- **AI**: Claude API với gọi công cụ và đầu ra có cấu trúc
- **Triển khai**: Google Cloud Run
- **Kiểm thử**: Playwright (E2E), pytest (backend), React Testing Library

**Các Dịch vụ:**

```
┌─────────────────────────────────────────────────────────────┐
│                         Frontend                            │
│  Next.js 15 + TypeScript + TailwindCSS                     │
│  Đã triển khai: Vercel / Cloud Run                         │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│                         Backend                             │
│  FastAPI + Python 3.11 + Pydantic                          │
│  Đã triển khai: Cloud Run                                  │
└─────────────────────────────────────────────────────────────┘
                              │
              ┌───────────────┼───────────────┐
              ▼               ▼               ▼
        ┌──────────┐   ┌──────────┐   ┌──────────┐
        │ Supabase │   │  Claude  │   │  Redis   │
        │ Database │   │   API    │   │  Cache   │
        └──────────┘   └──────────┘   └──────────┘
```

---

## Cấu trúc Tệp

```
project/
├── frontend/
│   └── src/
│       ├── app/              # Next.js app router pages
│       │   ├── api/          # Các route API
│       │   ├── (auth)/       # Các route được bảo vệ bởi auth
│       │   └── workspace/    # Không gian làm việc ứng dụng chính
│       ├── components/       # Các thành phần React
│       │   ├── ui/           # Các thành phần UI cơ bản
│       │   ├── forms/        # Các thành phần Form
│       │   └── layouts/      # Các thành phần Layout
│       ├── hooks/            # Các React hooks tùy chỉnh
│       ├── lib/              # Các tiện ích
│       ├── types/            # Định nghĩa TypeScript
│       └── config/           # Cấu hình
│
├── backend/
│   ├── routers/              # Các trình xử lý route FastAPI
│   ├── models.py             # Các mô hình Pydantic
│   ├── main.py               # Đầu vào ứng dụng FastAPI
│   ├── auth_system.py        # Xác thực
│   ├── database.py           # Các hoạt động cơ sở dữ liệu
│   ├── services/             # Logic nghiệp vụ
│   └── tests/                # Các bài kiểm tra pytest
│
├── deploy/                   # Cấu hình triển khai
├── docs/                     # Tài liệu
└── scripts/                  # Các tập lệnh tiện ích
```

---

## Các Mẫu Mã

### Định dạng Phản hồi API (FastAPI)

```python
from pydantic import BaseModel
from typing import Generic, TypeVar, Optional

T = TypeVar('T')

class ApiResponse(BaseModel, Generic[T]):
    success: bool
    data: Optional[T] = None
    error: Optional[str] = None

    @classmethod
    def ok(cls, data: T) -> "ApiResponse[T]":
        return cls(success=True, data=data)

    @classmethod
    def fail(cls, error: str) -> "ApiResponse[T]":
        return cls(success=False, error=error)
```

### Gọi API Frontend (TypeScript)

```typescript
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
}

async function fetchApi<T>(
  endpoint: string,
  options?: RequestInit,
): Promise<ApiResponse<T>> {
  try {
    const response = await fetch(`/api${endpoint}`, {
      ...options,
      headers: {
        "Content-Type": "application/json",
        ...options?.headers,
      },
    });

    if (!response.ok) {
      return { success: false, error: `HTTP ${response.status}` };
    }

    return await response.json();
  } catch (error) {
    return { success: false, error: String(error) };
  }
}
```

### Tích hợp Claude AI (Đầu ra có cấu trúc)

```python
from anthropic import Anthropic
from pydantic import BaseModel

class AnalysisResult(BaseModel):
    summary: str
    key_points: list[str]
    confidence: float

async def analyze_with_claude(content: str) -> AnalysisResult:
    client = Anthropic()

    response = client.messages.create(
        model="claude-sonnet-4-5-20250514",
        max_tokens=1024,
        messages=[{"role": "user", "content": content}],
        tools=[{
            "name": "provide_analysis",
            "description": "Provide structured analysis",
            "input_schema": AnalysisResult.model_json_schema()
        }],
        tool_choice={"type": "tool", "name": "provide_analysis"}
    )

    # Trích xuất kết quả sử dụng công cụ
    tool_use = next(
        block for block in response.content
        if block.type == "tool_use"
    )

    return AnalysisResult(**tool_use.input)
```

### Custom Hooks (React)

```typescript
import { useState, useCallback } from "react";

interface UseApiState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

export function useApi<T>(fetchFn: () => Promise<ApiResponse<T>>) {
  const [state, setState] = useState<UseApiState<T>>({
    data: null,
    loading: false,
    error: null,
  });

  const execute = useCallback(async () => {
    setState((prev) => ({ ...prev, loading: true, error: null }));

    const result = await fetchFn();

    if (result.success) {
      setState({ data: result.data!, loading: false, error: null });
    } else {
      setState({ data: null, loading: false, error: result.error! });
    }
  }, [fetchFn]);

  return { ...state, execute };
}
```

---

## Yêu cầu Kiểm thử

### Backend (pytest)

```bash
# Chạy tất cả các bài kiểm tra
poetry run pytest tests/

# Chạy với độ bao phủ
poetry run pytest tests/ --cov=. --cov-report=html

# Chạy tệp kiểm tra cụ thể
poetry run pytest tests/test_auth.py -v
```

**Cấu trúc kiểm thử:**

```python
import pytest
from httpx import AsyncClient
from main import app

@pytest.fixture
async def client():
    async with AsyncClient(app=app, base_url="http://test") as ac:
        yield ac

@pytest.mark.asyncio
async def test_health_check(client: AsyncClient):
    response = await client.get("/health")
    assert response.status_code == 200
    assert response.json()["status"] == "healthy"
```

### Frontend (React Testing Library)

```bash
# Chạy các bài kiểm tra
npm run test

# Chạy với độ bao phủ
npm run test -- --coverage

# Chạy các bài kiểm tra E2E
npm run test:e2e
```

**Cấu trúc kiểm thử:**

```typescript
import { render, screen, fireEvent } from '@testing-library/react'
import { WorkspacePanel } from './WorkspacePanel'

describe('WorkspacePanel', () => {
  it('renders workspace correctly', () => {
    render(<WorkspacePanel />)
    expect(screen.getByRole('main')).toBeInTheDocument()
  })

  it('handles session creation', async () => {
    render(<WorkspacePanel />)
    fireEvent.click(screen.getByText('New Session'))
    expect(await screen.findByText('Session created')).toBeInTheDocument()
  })
})
```

---

## Quy trình Triển khai

### Danh sách kiểm tra Trước khi Triển khai

- [ ] Tất cả các bài kiểm tra đều vượt qua cục bộ
- [ ] `npm run build` thành công (frontend)
- [ ] `poetry run pytest` vượt qua (backend)
- [ ] Không có bí mật được mã hóa cứng
- [ ] Các biến môi trường được ghi lại
- [ ] Di chuyển cơ sở dữ liệu đã sẵn sàng

### Các lệnh Triển khai

```bash
# Xây dựng và triển khai frontend
cd frontend && npm run build
gcloud run deploy frontend --source .

# Xây dựng và triển khai backend
cd backend
gcloud run deploy backend --source .
```

### Các biến Môi trường

```bash
# Frontend (.env.local)
NEXT_PUBLIC_API_URL=https://api.example.com
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ...

# Backend (.env)
DATABASE_URL=postgresql://...
ANTHROPIC_API_KEY=sk-ant-...
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_KEY=eyJ...
```

---

## Các Quy tắc Quan trọng

1. **Không có biểu tượng cảm xúc** trong mã, bình luận, hoặc tài liệu
2. **Tính bất biến** - không bao giờ thay đổi các đối tượng hoặc mảng
3. **TDD** - viết kiểm thử trước khi thực hiện
4. **Độ bao phủ tối thiểu 80%**
5. **Nhiều tệp nhỏ** - điển hình 200-400 dòng, tối đa 800
6. **Không console.log** trong mã sản xuất
7. **Xử lý lỗi đúng cách** với try/catch
8. **Xác thực đầu vào** với Pydantic/Zod

---

## Các Kỹ năng Liên quan

- `coding-standards.md` - Thực tiễn tốt nhất về mã hóa chung
- `backend-patterns.md` - Các mẫu API và cơ sở dữ liệu
- `frontend-patterns.md` - Các mẫu React và Next.js
- `tdd-workflow/` - Phương pháp phát triển hướng kiểm thử
