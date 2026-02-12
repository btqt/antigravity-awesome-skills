---
name: python-fastapi
description: Các nguyên tắc mẫu FastAPI REST API. SQLAlchemy, Pydantic, Alembic.
---

# Mẫu FastAPI API

## Tech Stack

| Thành phần          | Công nghệ      |
| ------------------- | -------------- |
| Framework           | FastAPI        |
| Ngôn ngữ            | Python 3.11+   |
| ORM                 | SQLAlchemy 2.0 |
| Xác thực dữ liệu    | Pydantic v2    |
| Migrations          | Alembic        |
| Xác thực người dùng | JWT + passlib  |

---

## Cấu trúc Thư mục

```
project-name/
├── alembic/             # Migrations
├── app/
│   ├── main.py          # Ứng dụng FastAPI
│   ├── config.py        # Cài đặt
│   ├── database.py      # Kết nối DB
│   ├── models/          # SQLAlchemy models
│   ├── schemas/         # Pydantic schemas
│   ├── routers/         # Các tuyến API
│   ├── services/        # Logic nghiệp vụ
│   ├── dependencies/    # Tiêm phụ thuộc (DI)
│   └── utils/
├── tests/
├── .env.example
└── requirements.txt
```

---

## Các Khái niệm Chính

| Khái niệm                             | Mô tả                            |
| ------------------------------------- | -------------------------------- |
| Bất đồng bộ (Async)                   | async/await xuyên suốt           |
| Tiêm phụ thuộc (Dependency Injection) | FastAPI Depends                  |
| Pydantic v2                           | Xác thực + tuần tự hóa           |
| SQLAlchemy 2.0                        | Các phiên (sessions) bất đồng bộ |

---

## Cấu trúc API

| Lớp (Layer)  | Trách nhiệm                |
| ------------ | -------------------------- |
| Routers      | Xử lý HTTP                 |
| Dependencies | Auth, xác thực             |
| Services     | Logic nghiệp vụ            |
| Models       | Các thực thể cơ sở dữ liệu |
| Schemas      | Yêu cầu/phản hồi           |

---

## Các bước Thiết lập

1. `python -m venv venv`
2. `source venv/bin/activate`
3. `pip install fastapi uvicorn sqlalchemy alembic pydantic`
4. Tạo `.env`
5. `alembic upgrade head`
6. `uvicorn app.main:app --reload`

---

## Thực hành Tốt nhất

- Sử dụng async ở mọi nơi
- Pydantic v2 để xác thực
- SQLAlchemy 2.0 với các phiên async
- Alembic cho migrations
- pytest-asyncio cho kiểm thử
