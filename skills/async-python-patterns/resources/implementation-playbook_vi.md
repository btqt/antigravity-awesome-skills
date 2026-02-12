# Sổ tay Triển khai các Mẫu Python Bất đồng bộ (Async Python Patterns Implementation Playbook)

Tệp này chứa các mẫu chi tiết, danh sách kiểm tra và các mẫu mã được tham chiếu bởi kỹ năng.

## Các Khái niệm Cốt lõi

### 1. Vòng lặp Sự kiện (Event Loop)

Vòng lặp sự kiện là trung tâm của asyncio, quản lý và lập lịch cho các tác vụ không đồng bộ.

**Các đặc điểm chính:**

- Đa nhiệm cộng tác đơn luồng (Single-threaded cooperative multitasking)
- Lập lịch cho các coroutine để thực thi
- Xử lý các hoạt động I/O mà không gây chặn
- Quản lý các callback và futures

### 2. Coroutines

Các hàm được định nghĩa với `async def` có thể được tạm dừng và tiếp tục.

**Cú pháp:**

```python
async def my_coroutine():
    result = await some_async_operation()
    return result
```

### 3. Tác vụ (Tasks)

Các coroutine đã được lập lịch chạy đồng thời trên vòng lặp sự kiện.

### 4. Futures

Các đối tượng cấp thấp đại diện cho kết quả cuối cùng của các hoạt động không đồng bộ.

### 5. Quản lý Ngữ cảnh Bất đồng bộ (Async Context Managers)

Các tài nguyên hỗ trợ `async with` để dọn dẹp đúng cách.

### 6. Trình lặp Bất đồng bộ (Async Iterators)

Các đối tượng hỗ trợ `async for` để lặp qua các nguồn dữ liệu không đồng bộ.

## Khởi đầu nhanh

```python
import asyncio

async def main():
    print("Xin chào")
    await asyncio.sleep(1)
    print("Thế giới")

# Python 3.7+
asyncio.run(main())
```

## Các Mẫu Cơ bản

### Mẫu 1: Async/Await Cơ bản

```python
import asyncio

async def fetch_data(url: str) -> dict:
    """Lấy dữ liệu từ URL một cách bất đồng bộ."""
    await asyncio.sleep(1)  # Giả lập I/O
    return {"url": url, "data": "result"}

async def main():
    result = await fetch_data("https://api.example.com")
    print(result)

asyncio.run(main())
```

### Mẫu 2: Thực thi Đồng thời với gather()

```python
import asyncio
from typing import List

async def fetch_user(user_id: int) -> dict:
    """Lấy dữ liệu người dùng."""
    await asyncio.sleep(0.5)
    return {"id": user_id, "name": f"User {user_id}"}

async def fetch_all_users(user_ids: List[int]) -> List[dict]:
    """Lấy dữ liệu của nhiều người dùng đồng thời."""
    tasks = [fetch_user(uid) for uid in user_ids]
    results = await asyncio.gather(*tasks)
    return results

async def main():
    user_ids = [1, 2, 3, 4, 5]
    users = await fetch_all_users(user_ids)
    print(f"Đã lấy dữ liệu của {len(users)} người dùng")

asyncio.run(main())
```

### Mẫu 3: Tạo và Quản lý Tác vụ

```python
import asyncio

async def background_task(name: str, delay: int):
    """Tác vụ nền chạy lâu dài."""
    print(f"{name} đã bắt đầu")
    await asyncio.sleep(delay)
    print(f"{name} đã hoàn thành")
    return f"Kết quả từ {name}"

async def main():
    # Tạo các tác vụ
    task1 = asyncio.create_task(background_task("Tác vụ 1", 2))
    task2 = asyncio.create_task(background_task("Tác vụ 2", 1))

    # Làm các việc khác
    print("Main: đang làm việc khác")
    await asyncio.sleep(0.5)

    # Đợi các tác vụ hoàn thành
    result1 = await task1
    result2 = await task2

    print(f"Kết quả: {result1}, {result2}")

asyncio.run(main())
```

### Mẫu 4: Xử lý Lỗi trong Mã Async

```python
import asyncio
from typing import List, Optional

async def risky_operation(item_id: int) -> dict:
    """Hoạt động có thể thất bại."""
    await asyncio.sleep(0.1)
    if item_id % 3 == 0:
        raise ValueError(f"Mục {item_id} bị lỗi")
    return {"id": item_id, "status": "success"}

async def safe_operation(item_id: int) -> Optional[dict]:
    """Wrapper với xử lý lỗi."""
    try:
        return await risky_operation(item_id)
    except ValueError as e:
        print(f"Lỗi: {e}")
        return None

async def process_items(item_ids: List[int]):
    """Xử lý nhiều mục với xử lý lỗi."""
    tasks = [safe_operation(iid) for iid in item_ids]
    results = await asyncio.gather(*tasks, return_exceptions=True)

    # Lọc ra các thành công
    successful = [r for r in results if r is not None and not isinstance(r, Exception)]
    failed = [r for r in results if isinstance(r, Exception)]

    print(f"Thành công: {len(successful)}, Thất bại: {len(failed)}")
    return successful

asyncio.run(process_items([1, 2, 3, 4, 5, 6]))
```

### Mẫu 5: Xử lý Thời gian chờ (Timeout)

```python
import asyncio

async def slow_operation(delay: int) -> str:
    """Hoạt động tốn thời gian."""
    await asyncio.sleep(delay)
    return f"Đã hoàn thành sau {delay} giây"

async def with_timeout():
    """Thực thi hoạt động với thời gian chờ."""
    try:
        result = await asyncio.wait_for(slow_operation(5), timeout=2.0)
        print(result)
    except asyncio.TimeoutError:
        print("Hoạt động đã quá thời gian chờ")

asyncio.run(with_timeout())
```

## Các Mẫu Nâng cao

### Mẫu 6: Trình Quản lý Ngữ cảnh Bất đồng bộ (Async Context Managers)

```python
import asyncio
from typing import Optional

class AsyncDatabaseConnection:
    """Trình quản lý ngữ cảnh kết nối cơ sở dữ liệu bất đồng bộ."""

    def __init__(self, dsn: str):
        self.dsn = dsn
        self.connection: Optional[object] = None

    async def __aenter__(self):
        print("Đang mở kết nối")
        await asyncio.sleep(0.1)  # Giả lập kết nối
        self.connection = {"dsn": self.dsn, "connected": True}
        return self.connection

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        print("Đang đóng kết nối")
        await asyncio.sleep(0.1)  # Giả lập dọn dẹp
        self.connection = None

async def query_database():
    """Sử dụng trình quản lý ngữ cảnh bất đồng bộ."""
    async with AsyncDatabaseConnection("postgresql://localhost") as conn:
        print(f"Đang sử dụng kết nối: {conn}")
        await asyncio.sleep(0.2)  # Giả lập truy vấn
        return {"rows": 10}

asyncio.run(query_database())
```

### Mẫu 7: Trình lặp và Trình tạo Bất đồng bộ (Async Iterators and Generators)

```python
import asyncio
from typing import AsyncIterator

async def async_range(start: int, end: int, delay: float = 0.1) -> AsyncIterator[int]:
    """Trình tạo bất đồng bộ trả về các số sau một khoảng trễ."""
    for i in range(start, end):
        await asyncio.sleep(delay)
        yield i

async def fetch_pages(url: str, max_pages: int) -> AsyncIterator[dict]:
    """Lấy dữ liệu phân trang một cách bất đồng bộ."""
    for page in range(1, max_pages + 1):
        await asyncio.sleep(0.2)  # Giả lập gọi API
        yield {
            "page": page,
            "url": f"{url}?page={page}",
            "data": [f"item_{page}_{i}" for i in range(5)]
        }

async def consume_async_iterator():
    """Tiêu thụ trình lặp bất đồng bộ."""
    async for number in async_range(1, 5):
        print(f"Số: {number}")

    print("\nĐang lấy các trang:")
    async for page_data in fetch_pages("https://api.example.com/items", 3):
        print(f"Trang {page_data['page']}: {len(page_data['data'])} mục")

asyncio.run(consume_async_iterator())
```

### Mẫu 8: Mẫu Người sản xuất - Người tiêu thụ (Producer-Consumer Pattern)

```python
import asyncio
from asyncio import Queue
from typing import Optional

async def producer(queue: Queue, producer_id: int, num_items: int):
    """Sản xuất các mục và đưa vào hàng đợi."""
    for i in range(num_items):
        item = f"Mục-{producer_id}-{i}"
        await queue.put(item)
        print(f"Người sản xuất {producer_id} đã tạo: {item}")
        await asyncio.sleep(0.1)
    await queue.put(None)  # Thông báo hoàn thành

async def consumer(queue: Queue, consumer_id: int):
    """Tiêu thụ các mục từ hàng đợi."""
    while True:
        item = await queue.get()
        if item is None:
            queue.task_done()
            break

        print(f"Người tiêu thụ {consumer_id} đang xử lý: {item}")
        await asyncio.sleep(0.2)  # Giả lập công việc
        queue.task_done()

async def producer_consumer_example():
    """Chạy mẫu người sản xuất - người tiêu thụ."""
    queue = Queue(maxsize=10)

    # Tạo các tác vụ
    producers = [
        asyncio.create_task(producer(queue, i, 5))
        for i in range(2)
    ]

    consumers = [
        asyncio.create_task(consumer(queue, i))
        for i in range(3)
    ]

    # Đợi những người sản xuất
    await asyncio.gather(*producers)

    # Đợi hàng đợi trống
    await queue.join()

    # Hủy những người tiêu thụ
    for c in consumers:
        c.cancel()

asyncio.run(producer_consumer_example())
```

### Mẫu 9: Semaphore để Giới hạn Tốc độ (Rate Limiting)

```python
import asyncio
from typing import List

async def api_call(url: str, semaphore: asyncio.Semaphore) -> dict:
    """Gọi API với giới hạn tốc độ."""
    async with semaphore:
        print(f"Đang gọi {url}")
        await asyncio.sleep(0.5)  # Giả lập gọi API
        return {"url": url, "status": 200}

async def rate_limited_requests(urls: List[str], max_concurrent: int = 5):
    """Thực hiện nhiều yêu cầu với giới hạn tốc độ."""
    semaphore = asyncio.Semaphore(max_concurrent)
    tasks = [api_call(url, semaphore) for url in urls]
    results = await asyncio.gather(*tasks)
    return results

async def main():
    urls = [f"https://api.example.com/item/{i}" for i in range(20)]
    results = await rate_limited_requests(urls, max_concurrent=3)
    print(f"Đã hoàn thành {len(results)} yêu cầu")

asyncio.run(main())
```

### Mẫu 10: Khóa Async và Đồng bộ hóa (Async Locks and Synchronization)

```python
import asyncio

class AsyncCounter:
    """Bộ đếm async an toàn (Thread-safe)."""

    def __init__(self):
        self.value = 0
        self.lock = asyncio.Lock()

    async def increment(self):
        """Tăng bộ đếm một cách an toàn."""
        async with self.lock:
            current = self.value
            await asyncio.sleep(0.01)  # Giả lập công việc
            self.value = current + 1

    async def get_value(self) -> int:
        """Lấy giá trị hiện tại."""
        async with self.lock:
            return self.value

async def worker(counter: AsyncCounter, worker_id: int):
    """Worker tăng bộ đếm."""
    for _ in range(10):
        await counter.increment()
        print(f"Worker {worker_id} đã tăng")

async def test_counter():
    """Kiểm thử bộ đếm đồng thời."""
    counter = AsyncCounter()

    workers = [asyncio.create_task(worker(counter, i)) for i in range(5)]
    await asyncio.gather(*workers)

    final_value = await counter.get_value()
    print(f"Giá trị cuối cùng của bộ đếm: {final_value}")

asyncio.run(test_counter())
```

## Các Ứng dụng Thực tế

### Thu thập Web (Web Scraping) với aiohttp

```python
import asyncio
import aiohttp
from typing import List, Dict

async def fetch_url(session: aiohttp.ClientSession, url: str) -> Dict:
    """Lấy dữ liệu từ một URL duy nhất."""
    try:
        async with session.get(url, timeout=aiohttp.ClientTimeout(total=10)) as response:
            text = await response.text()
            return {
                "url": url,
                "status": response.status,
                "length": len(text)
            }
    except Exception as e:
        return {"url": url, "error": str(e)}

async def scrape_urls(urls: List[str]) -> List[Dict]:
    """Thu thập nhiều URL đồng thời."""
    async with aiohttp.ClientSession() as session:
        tasks = [fetch_url(session, url) for url in urls]
        results = await asyncio.gather(*tasks)
        return results

async def main():
    urls = [
        "https://httpbin.org/delay/1",
        "https://httpbin.org/delay/2",
        "https://httpbin.org/status/404",
    ]

    results = await scrape_urls(urls)
    for result in results:
        print(result)

asyncio.run(main())
```

### Các hoạt động Cơ sở dữ liệu Bất đồng bộ

```python
import asyncio
from typing import List, Optional

# Giả lập client cơ sở dữ liệu bất đồng bộ
class AsyncDB:
    """Cơ sở dữ liệu bất đồng bộ giả lập."""

    async def execute(self, query: str) -> List[dict]:
        """Thực thi truy vấn."""
        await asyncio.sleep(0.1)
        return [{"id": 1, "name": "Example"}]

    async def fetch_one(self, query: str) -> Optional[dict]:
        """Lấy một hàng duy nhất."""
        await asyncio.sleep(0.1)
        return {"id": 1, "name": "Example"}

async def get_user_data(db: AsyncDB, user_id: int) -> dict:
    """Lấy dữ liệu người dùng và các dữ liệu liên quan đồng thời."""
    user_task = db.fetch_one(f"SELECT * FROM users WHERE id = {user_id}")
    orders_task = db.execute(f"SELECT * FROM orders WHERE user_id = {user_id}")
    profile_task = db.fetch_one(f"SELECT * FROM profiles WHERE user_id = {user_id}")

    user, orders, profile = await asyncio.gather(user_task, orders_task, profile_task)

    return {
        "user": user,
        "orders": orders,
        "profile": profile
    }

async def main():
    db = AsyncDB()
    user_data = await get_user_data(db, 1)
    print(user_data)

asyncio.run(main())
```

### Máy chủ WebSocket

```python
import asyncio
from typing import Set

# Giả lập kết nối WebSocket
class WebSocket:
    """WebSocket giả lập."""

    def __init__(self, client_id: str):
        self.client_id = client_id

    async def send(self, message: str):
        """Gửi tin nhắn."""
        print(f"Đang gửi đến {self.client_id}: {message}")
        await asyncio.sleep(0.01)

    async def recv(self) -> str:
        """Nhận tin nhắn."""
        await asyncio.sleep(1)
        return f"Tin nhắn từ {self.client_id}"

class WebSocketServer:
    """Máy chủ WebSocket đơn giản."""

    def __init__(self):
        self.clients: Set[WebSocket] = set()

    async def register(self, websocket: WebSocket):
        """Đăng ký client mới."""
        self.clients.add(websocket)
        print(f"Client {websocket.client_id} đã kết nối")

    async def unregister(self, websocket: WebSocket):
        """Hủy đăng ký client."""
        self.clients.remove(websocket)
        print(f"Client {websocket.client_id} đã ngắt kết nối")

    async def broadcast(self, message: str):
        """Phát tin nhắn đến tất cả các client."""
        if self.clients:
            tasks = [client.send(message) for client in self.clients]
            await asyncio.gather(*tasks)

    async def handle_client(self, websocket: WebSocket):
        """Xử lý kết nối của từng client riêng lẻ."""
        await self.register(websocket)
        try:
            async for message in self.message_iterator(websocket):
                await self.broadcast(f"{websocket.client_id}: {message}")
        finally:
            await self.unregister(websocket)

    async def message_iterator(self, websocket: WebSocket):
        """Lặp qua các tin nhắn từ client."""
        for _ in range(3):  # Giả lập 3 tin nhắn
            yield await websocket.recv()
```

## Các Thực hành Tốt nhất về Hiệu suất

### 1. Sử dụng Connection Pools (Nhóm kết nối)

```python
import asyncio
import aiohttp

async def with_connection_pool():
    """Sử dụng nhóm kết nối để đạt hiệu quả cao."""
    connector = aiohttp.TCPConnector(limit=100, limit_per_host=10)

    async with aiohttp.ClientSession(connector=connector) as session:
        tasks = [session.get(f"https://api.example.com/item/{i}") for i in range(50)]
        responses = await asyncio.gather(*tasks)
        return responses
```

### 2. Hoạt động theo Lô (Batch Operations)

```python
async def batch_process(items: List[str], batch_size: int = 10):
    """Xử lý các mục theo từng lô."""
    for i in range(0, len(items), batch_size):
        batch = items[i:i + batch_size]
        tasks = [process_item(item) for item in batch]
        await asyncio.gather(*tasks)
        print(f"Đã xử lý xong lô thứ {i // batch_size + 1}")

async def process_item(item: str):
    """Xử lý một mục duy nhất."""
    await asyncio.sleep(0.1)
    return f"Đã xử lý: {item}"
```

### 3. Tránh các Hoạt động gây Chặn (Blocking Operations)

```python
import asyncio
import concurrent.futures
from typing import Any

def blocking_operation(data: Any) -> Any:
    """Hoạt động gây chặn thâm dụng CPU."""
    import time
    time.sleep(1)
    return data * 2

async def run_in_executor(data: Any) -> Any:
    """Chạy hoạt động gây chặn trong một thread pool."""
    loop = asyncio.get_event_loop()
    with concurrent.futures.ThreadPoolExecutor() as pool:
        result = await loop.run_in_executor(pool, blocking_operation, data)
        return result

async def main():
    results = await asyncio.gather(*[run_in_executor(i) for i in range(5)])
    print(results)

asyncio.run(main())
```

## Các Bẫy phổ biến (Common Pitfalls)

### 1. Quên `await`

```python
# Sai - trả về đối tượng coroutine, không thực thi
result = async_function()

# Đúng
result = await async_function()
```

### 2. Gây chặn Vòng lặp Sự kiện (Blocking the Event Loop)

```python
# Sai - chặn vòng lặp sự kiện
import time
async def bad():
    time.sleep(1)  # Chặn!

# Đúng
async def good():
    await asyncio.sleep(1)  # Không chặn
```

### 3. Không xử lý việc Hủy bỏ Tác vụ (Cancellation)

```python
async def cancelable_task():
    """Tác vụ có xử lý việc hủy bỏ."""
    try:
        while True:
            await asyncio.sleep(1)
            print("Đang làm việc...")
    except asyncio.CancelledError:
        print("Tác vụ đã bị hủy, đang dọn dẹp...")
        # Thực hiện dọn dẹp
        raise  # Ném lại lỗi để lan truyền việc hủy bỏ
```

### 4. Trộn lẫn mã Đồng bộ và Bất đồng bộ

```python
# Sai - không thể gọi trực tiếp async từ sync
def sync_function():
    result = await async_function()  # Lỗi cú pháp (SyntaxError)!

# Đúng
def sync_function():
    result = asyncio.run(async_function())
```

## Kiểm thử Mã Async

```python
import asyncio
import pytest

# Sử dụng pytest-asyncio
@pytest.mark.asyncio
async def test_async_function():
    """Kiểm thử hàm async."""
    result = await fetch_data("https://api.example.com")
    assert result is not None

@pytest.mark.asyncio
async def test_with_timeout():
    """Kiểm thử với thời gian chờ."""
    with pytest.raises(asyncio.TimeoutError):
        await asyncio.wait_for(slow_operation(5), timeout=1.0)
```

## Tài nguyên

- **Tài liệu Python asyncio**: https://docs.python.org/3/library/asyncio.html
- **aiohttp**: Client/server HTTP bất đồng bộ
- **FastAPI**: Web framework hiện đại hỗ trợ async
- **asyncpg**: Trình điều khiển PostgreSQL bất đồng bộ
- **motor**: Trình điều khiển MongoDB bất đồng bộ

## Tóm tắt Thực hành Tốt nhất

1. **Sử dụng `asyncio.run()`** cho điểm bắt đầu (Python 3.7+)
2. **Luôn `await` các coroutine** để thực thi chúng
3. **Sử dụng `gather()` để thực thi đồng thời** nhiều tác vụ
4. **Triển khai xử lý lỗi đúng cách** với try/except
5. **Sử dụng thời gian chờ (timeouts)** để ngăn các hoạt động bị treo
6. **Sử dụng nhóm kết nối (connection pools)** để có hiệu suất tốt hơn
7. **Tránh các hoạt động gây chặn** trong mã async
8. **Sử dụng semaphore** để giới hạn tốc độ (rate limiting)
9. **Xử lý việc hủy bỏ tác vụ** một cách đúng đắn
10. **Kiểm thử mã async** với pytest-asyncio
