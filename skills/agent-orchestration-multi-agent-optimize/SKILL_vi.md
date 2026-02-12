---
name: agent-orchestration-multi-agent-optimize
description: "Tối ưu hóa các hệ thống đa đại lý (multi-agent systems) với việc phân tích hiệu năng (profiling) có phối hợp, phân bổ khối lượng công việc và điều phối có cân nhắc chi phí. Sử dụng khi cải thiện hiệu suất, thông lượng hoặc độ tin cậy của đại lý."
---

# Bộ công cụ Tối ưu hóa Đa đại lý

## Sử dụng kỹ năng này khi

- Cải thiện sự phối hợp, thông lượng hoặc độ trễ của hệ thống đa đại lý
- Phân tích hiệu năng (profiling) quy trình của đại lý để xác định các điểm nghẽn
- Thiết kế các chiến lược điều phối cho các quy trình công việc phức tạp
- Tối ưu hóa chi phí, sử dụng ngữ cảnh (context) hoặc hiệu quả sử dụng công cụ

## Không sử dụng kỹ năng này khi

- Bạn chỉ cần tinh chỉnh prompt cho một đại lý duy nhất
- Không có các chỉ số có thể đo lường hoặc dữ liệu đánh giá
- Nhiệm vụ không liên quan đến việc điều phối đa đại lý

## Hướng dẫn

1. Thiết lập các chỉ số cơ sở và các mục tiêu hiệu suất đích.
2. Phân tích hiệu năng khối lượng công việc của đại lý và xác định các điểm nghẽn trong phối hợp.
3. Áp dụng các thay đổi trong điều phối và kiểm soát chi phí một cách dần dần.
4. Xác thực các cải tiến bằng các bài kiểm thử có thể lặp lại và có khả năng khôi phục.

## An toàn

- Tránh triển khai các thay đổi trong điều phối mà không kiểm thử hồi quy.
- Triển khai các thay đổi dần dần để ngăn chặn các lỗi suy giảm trên toàn hệ thống.

## Vai trò: Chuyên gia Kỹ thuật Hiệu suất Đa đại lý hỗ trợ bởi AI

### Ngữ cảnh

Công cụ Tối ưu hóa Đa đại lý là một khung làm việc tiên tiến dựa trên AI được thiết kế để cải thiện hiệu suất hệ thống một cách toàn diện thông qua việc tối ưu hóa dựa trên đại lý có phối hợp và thông minh. Tận dụng các kỹ thuật điều phối AI tiên tiến, công cụ này cung cấp một phương pháp toàn diện cho kỹ thuật hiệu suất trên nhiều lĩnh vực.

### Năng lực Cốt lõi

- Phối hợp đa đại lý thông minh
- Phân tích hiệu năng và xác định điểm nghẽn
- Các chiến lược tối ưu hóa thích ứng
- Tối ưu hóa hiệu suất chéo lĩnh vực
- Theo dõi chi phí và hiệu quả

## Xử lý Đối số

Công cụ xử lý các đối số tối ưu hóa với các tham số đầu vào linh hoạt:

- `$TARGET`: Hệ thống/ứng dụng chính cần tối ưu hóa
- `$PERFORMANCE_GOALS`: Các chỉ số và mục tiêu hiệu suất cụ thể
- `$OPTIMIZATION_SCOPE`: Độ sâu của việc tối ưu hóa (thắng nhanh - quick-win, hoặc toàn diện)
- `$BUDGET_CONSTRAINTS`: Các giới hạn về chi phí và tài nguyên
- `$QUALITY_METRICS`: Các ngưỡng chất lượng hiệu suất

## 1. Phân tích Hiệu năng (Profiling) Đa đại lý

### Chiến lược Phân tích

- Giám sát hiệu suất phân tán trên các lớp hệ thống
- Thu thập và phân tích các chỉ số trong thời gian thực
- Theo dõi chữ ký hiệu suất liên tục

#### Các đại lý Phân tích (Profiling Agents)

1. **Đại lý Hiệu suất Cơ sở dữ liệu**
   - Phân tích thời gian thực thi truy vấn
   - Theo dõi việc sử dụng chỉ mục (index)
   - Giám sát tiêu thụ tài nguyên

2. **Đại lý Hiệu suất Ứng dụng**
   - Phân tích CPU và bộ nhớ
   - Đánh giá độ phức tạp của thuật toán
   - Phân tích các hoạt động đồng thời và bất đồng bộ

3. **Đại lý Hiệu suất Frontend**
   - Các chỉ số hiệu suất kết xuất (rendering)
   - Tối ưu hóa yêu cầu mạng
   - Giám sát Core Web Vitals

### Ví dụ mã phân tích hiệu năng

```python
def multi_agent_profiler(target_system):
    agents = [
        DatabasePerformanceAgent(target_system),
        ApplicationPerformanceAgent(target_system),
        FrontendPerformanceAgent(target_system)
    ]

    performance_profile = {}
    for agent in agents:
        performance_profile[agent.__class__.__name__] = agent.profile()

    return aggregate_performance_metrics(performance_profile)
```

## 2. Tối ưu hóa Cửa sổ Ngữ cảnh (Context Window)

### Kỹ thuật Tối ưu hóa

- Nén ngữ cảnh thông minh
- Lọc mức độ liên quan về ngữ nghĩa
- Thay đổi kích thước cửa sổ ngữ cảnh động
- Quản lý ngân sách token

### Thuật toán Nén Ngữ cảnh

```python
def compress_context(context, max_tokens=4000):
    # Nén ngữ nghĩa bằng cách cắt bớt dựa trên embedding
    compressed_context = semantic_truncate(
        context,
        max_tokens=max_tokens,
        importance_threshold=0.7
    )
    return compressed_context
```

## 3. Hiệu quả Phối hợp Đại lý

### Nguyên tắc Phối hợp

- Thiết kế thực thi song song
- Giảm thiểu chi phí giao tiếp giữa các đại lý
- Phân bổ khối lượng công việc động
- Tương tác đại lý có khả năng chịu lỗi

### Khung Điều phối (Orchestration Framework)

```python
class MultiAgentOrchestrator:
    def __init__(self, agents):
        self.agents = agents
        self.execution_queue = PriorityQueue()
        self.performance_tracker = PerformanceTracker()

    def optimize(self, target_system):
        # Thực thi đại lý song song với tối ưu hóa có phối hợp
        with concurrent.futures.ThreadPoolExecutor() as executor:
            futures = {
                executor.submit(agent.optimize, target_system): agent
                for agent in self.agents
            }

            for future in concurrent.futures.as_completed(futures):
                agent = futures[future]
                result = future.result()
                self.performance_tracker.log(agent, result)
```

## 4. Tối ưu hóa Thực thi Song song

### Các chiến lược chính

- Xử lý đại lý bất đồng bộ
- Phân chia khối lượng công việc
- Phân bổ tài nguyên động
- Giảm thiểu tối đa các hoạt động gây nghẽn (blocking operations)

## 5. Chiến lược Tối ưu hóa Chi phí

### Quản lý Chi phí LLM

- Theo dõi sử dụng token
- Lựa chọn mô hình thích ứng
- Lưu đệm (caching) và tái sử dụng kết quả
- Prompt engineering hiệu quả

### Ví dụ Theo dõi Chi phí

```python
class CostOptimizer:
    def __init__(self):
        self.token_budget = 100000  # Ngân sách hàng tháng
        self.token_usage = 0
        self.model_costs = {
            'gpt-4o': 0.005,
            'claude-3-5-sonnet': 0.003,
            'claude-3-haiku': 0.00025
        }

    def select_optimal_model(self, complexity):
        # Lựa chọn mô hình động dựa trên độ phức tạp của nhiệm vụ và ngân sách
        pass
```

## 6. Kỹ thuật Giảm Độ trễ

### Tăng tốc Hiệu suất

- Lưu đệm dự đoán (Predictive caching)
- Làm nóng (Pre-warming) ngữ cảnh đại lý
- Ghi nhớ (memoization) kết quả thông minh
- Giảm thiểu giao tiếp khứ hồi (round-trip)

## 7. Sự đánh đổi giữa Chất lượng và Tốc độ

### Phổ Tối ưu hóa

- Các ngưỡng hiệu suất
- Các biên độ suy giảm có thể chấp nhận được
- Tối ưu hóa có cân nhắc chất lượng
- Lựa chọn sự thỏa hiệp thông minh

## 8. Giám sát và Cải tiến Liên tục

### Khung Khả năng Quan sát (Observability Framework)

- Bảng điều khiển hiệu suất thời gian thực
- Vòng lặp phản hồi tối ưu hóa tự động
- Cải tiến dựa trên học máy
- Các chiến lược tối ưu hóa thích ứng

## Quy trình Tham chiếu

### Quy trình 1: Tối ưu hóa nền tảng thương mại điện tử

1. Phân tích hiệu năng ban đầu
2. Tối ưu hóa dựa trên đại lý
3. Theo dõi chi phí và hiệu suất
4. Chu kỳ cải tiến liên tục

### Quy trình 2: Tăng cường Hiệu suất API Doanh nghiệp

1. Phân tích hệ thống toàn diện
2. Tối ưu hóa đại lý đa lớp
3. Tinh chỉnh hiệu suất lặp lại
4. Chiến lược mở rộng quy mô tiết kiệm chi phí

## Các cân nhắc chính

- Luôn đo lường trước và sau khi tối ưu hóa
- Duy trì sự ổn định của hệ thống trong quá trình tối ưu hóa
- Cân bằng giữa mức tăng hiệu suất với mức tiêu thụ tài nguyên
- Triển khai các thay đổi dần dần và có thể đảo ngược

Tối ưu hóa Mục tiêu: $ARGUMENTS
