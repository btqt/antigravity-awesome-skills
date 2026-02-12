---
name: computer-use-agents
description: "Xây dựng các tác nhân AI tương tác với máy tính như con người - xem màn hình, di chuyển con trỏ, nhấp nút và gõ văn bản. Bao gồm Computer Use của Anthropic, Operator/CUA của OpenAI và các lựa chọn thay thế nguồn mở. Tập trung quan trọng vào sandbox, bảo mật và xử lý các thách thức độc đáo của điều khiển dựa trên tầm nhìn. Sử dụng khi: sử dụng máy tính, tác nhân tự động hóa máy tính để bàn, AI điều khiển màn hình, tác nhân dựa trên tầm nhìn, tự động hóa GUI."
source: vibeship-spawner-skills (Apache 2.0)
---

# Tác nhân Sử dụng Máy tính (Computer Use Agents)

## Các mô hình

### Vòng lặp Nhận thức-Suy luận-Hành động

Kiến trúc cơ bản của các tác nhân sử dụng máy tính: quan sát màn hình, suy luận về hành động tiếp theo, thực hiện hành động, lặp lại. Vòng lặp này tích hợp các mô hình tầm nhìn với việc thực hiện hành động thông qua một quy trình lặp đi lặp lại.

Các thành phần chính:

1. NHẬN THỨC (PERCEPTION): Ảnh chụp màn hình ghi lại trạng thái màn hình hiện tại
2. SUY LUẬN (REASONING): Mô hình ngôn ngữ-tầm nhìn phân tích và lập kế hoạch
3. HÀNH ĐỘNG (ACTION): Thực hiện các thao tác chuột/bàn phím
4. PHẢN HỒI (FEEDBACK): Quan sát kết quả, tiếp tục hoặc sửa chữa

Thông tin chi tiết quan trọng: Các tác nhân tầm nhìn hoàn toàn đứng yên trong giai đoạn "suy nghĩ" (1-5 giây), tạo ra một mô hình tạm dừng có thể phát hiện được.

**Khi nào sử dụng**: ['Xây dựng bất kỳ tác nhân sử dụng máy tính nào từ đầu', 'Tích hợp các mô hình tầm nhìn với điều khiển máy tính để bàn', 'Hiểu các mẫu hành vi của tác nhân']

```python
from anthropic import Anthropic
from PIL import Image
import base64
import pyautogui
import time

class ComputerUseAgent:
    """
    Triển khai vòng lặp Nhận thức-Suy luận-Hành động.
    Dựa trên các mẫu Computer Use của Anthropic.
    """

    def __init__(self, client: Anthropic, model: str = "claude-sonnet-4-20250514"):
        self.client = client
        self.model = model
        self.max_steps = 50  # Ngăn chặn các vòng lặp chạy quá đà
        self.action_delay = 0.5  # Giây giữa các hành động

    def capture_screenshot(self) -> str:
        """Chụp màn hình và trả về hình ảnh được mã hóa base64."""
        screenshot = pyautogui.screenshot()
        # Thay đổi kích thước để hiệu quả token (1280x800 là cân bằng tốt)
        screenshot = screenshot.resize((1280, 800), Image.LANCZOS)

        import io
        buffer = io.BytesIO()
        screenshot.save(buffer, format="PNG")
        return base64.b64encode(buffer.getvalue()).decode()

    def execute_action(self, action: dict) -> dict:
        """Thực hiện hành động chuột/bàn phím trên máy tính."""
        action_type = action.get("type")

        if action_type == "click":
            x, y = action["x"], action["y"]
            button = action.get("button", "left")
            pyautogui.click(x, y, button=button)
            return {"success": True, "action": f"clicked at ({x}, {y})"}

        elif action_type == "type":
            text = action["text"]
            pyautogui.typewrite(text, interval=0.02)
            return {"success": True, "action": f"typed {len(text)} chars"}

        elif action_type == "key":
            key = action["key"]
            pyautogui.press(key)
            return {"success": True, "action": f"pressed {key}"}

        elif action_type == "scroll":
            direction = action.get("direction", "down")
            amount = action.get("amount", 3)
            scroll = -amount if direction == "down" else amount
            pyautogui.scroll(scroll)
            return {"success": True, "action": f"scrolled {dir
```

### Mô hình Môi trường Sandbox

Các tác nhân sử dụng máy tính PHẢI chạy trong các môi trường biệt lập, được sandbox. Không bao giờ cung cấp cho các tác nhân quyền truy cập trực tiếp vào hệ thống chính của bạn - rủi ro bảo mật quá cao. Sử dụng các container Docker với máy tính để bàn ảo.

Các yêu cầu cách ly chính:

1. MẠNG: Hạn chế chỉ các điểm cuối cần thiết
2. HỆ THỐNG TỆP: Chỉ đọc hoặc phạm vi trong các thư mục tạm thời
3. THÔNG TIN XÁC THỰC: Không truy cập vào thông tin xác thực máy chủ
4. SYSCALLS: Lọc các cuộc gọi hệ thống nguy hiểm
5. TÀI NGUYÊN: Giới hạn CPU, bộ nhớ, thời gian

Mục tiêu là "giảm thiểu bán kính nổ" - nếu tác nhân gặp sự cố, thiệt hại chỉ nằm trong sandbox.

**Khi nào sử dụng**: ['Triển khai bất kỳ tác nhân sử dụng máy tính nào', 'Kiểm tra hành vi tác nhân một cách an toàn', 'Chạy các tác vụ tự động hóa không tin cậy']

```python
# Dockerfile cho môi trường sử dụng máy tính được sandbox
# Dựa trên mẫu triển khai tham chiếu của Anthropic

FROM ubuntu:22.04

# Cài đặt môi trường máy tính để bàn
RUN apt-get update && apt-get install -y \
    xvfb \
    x11vnc \
    fluxbox \
    xterm \
    firefox \
    python3 \
    python3-pip \
    supervisor

# Bảo mật: Tạo người dùng không phải root
RUN useradd -m -s /bin/bash agent && \
    mkdir -p /home/agent/.vnc

# Cài đặt các phụ thuộc Python
COPY requirements.txt /tmp/
RUN pip3 install -r /tmp/requirements.txt

# Bảo mật: Loại bỏ khả năng (capabilities)
RUN apt-get install -y --no-install-recommends libcap2-bin && \
    setcap -r /usr/bin/python3 || true

# Sao chép mã tác nhân
COPY --chown=agent:agent . /app
WORKDIR /app

# Cấu hình Supervisor cho màn hình ảo + VNC
COPY supervisord.conf /etc/supervisor/conf.d/

# Chỉ hiển thị cổng VNC (không phải máy tính để bàn trực tiếp)
EXPOSE 5900

# Chạy như không phải root
USER agent

CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/conf.d/supervisord.conf"]

---

# docker-compose.yml với các ràng buộc bảo mật
version: '3.8'

services:
  computer-use-agent:
    build: .
    ports:
      - "5900:5900"  # VNC để quan sát
      - "8080:8080"  # API để điều khiển

    # Các ràng buộc bảo mật
    security_opt:
      - no-new-privileges:true
      - seccomp:seccomp-profile.json

    # Giới hạn tài nguyên
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 4G
        reservations:
          cpus: '0.5'
          memory: 1G

    # Cách ly mạng
    networks:
      - agent-network

    # Không truy cập vào hệ thống tệp máy chủ
    volumes:
      - agent-tmp:/tmp

    # Hệ thống tệp gốc chỉ đọc
    read_only: true
    tmpfs:
      - /run
      - /var/run

    # Môi trường
    environment:
      - DISPLAY=:99
      - NO_PROXY=localhost

networks:
  agent-network:
    driver: bridge
    internal: true  # Mặc định không có internet

volumes:
  agent-tmp:

---

# Trình bao bọc Python với sandbox thời gian chạy bổ sung
import subprocess
import os
from dataclasses im
```

### Triển khai Sử dụng Máy tính của Anthropic

Mẫu triển khai chính thức sử dụng khả năng sử dụng máy tính của Claude. Claude 3.5 Sonnet là mô hình biên giới đầu tiên cung cấp khả năng sử dụng máy tính. Claude Opus 4.5 hiện là "mô hình tốt nhất thế giới về sử dụng máy tính."

Các khả năng chính:

- screenshot: Chụp trạng thái màn hình hiện tại
- mouse: Nhấp, di chuyển, kéo thả
- keyboard: Gõ văn bản, nhấn phím
- bash: Chạy các lệnh shell
- text_editor: Xem và chỉnh sửa tệp

Các phiên bản công cụ:

- computer_20251124 (Opus 4.5): Thêm hành động thu phóng để kiểm tra chi tiết
- computer_20250124 (Tất cả các mô hình khác): Khả năng tiêu chuẩn

Hạn chế quan trọng: "Một số thành phần UI (như menu thả xuống và thanh cuộn) có thể khó khăn cho Claude thao tác" - Tài liệu Anthropic

**Khi nào sử dụng**: ['Xây dựng các tác nhân sử dụng máy tính sản xuất', 'Cần hiểu biết tầm nhìn chất lượng cao nhất', 'Điều khiển máy tính để bàn đầy đủ (không chỉ trình duyệt)']

```python
from anthropic import Anthropic
from anthropic.types.beta import (
    BetaToolComputerUse20241022,
    BetaToolBash20241022,
    BetaToolTextEditor20241022,
)
import subprocess
import base64
from PIL import Image
import io

class AnthropicComputerUse:
    """
    Triển khai Sử dụng Máy tính Anthropic chính thức.

    Yêu cầu:
    - Container Docker với màn hình ảo
    - VNC để xem hành động của tác nhân
    - Triển khai công cụ phù hợp
    """

    def __init__(self):
        self.client = Anthropic()
        self.model = "claude-sonnet-4-20250514"  # Tốt nhất cho sử dụng máy tính
        self.screen_size = (1280, 800)

    def get_tools(self) -> list:
        """Định nghĩa các công cụ sử dụng máy tính."""
        return [
            BetaToolComputerUse20241022(
                type="computer_20241022",
                name="computer",
                display_width_px=self.screen_size[0],
                display_height_px=self.screen_size[1],
            ),
            BetaToolBash20241022(
                type="bash_20241022",
                name="bash",
            ),
            BetaToolTextEditor20241022(
                type="text_editor_20241022",
                name="str_replace_editor",
            ),
        ]

    def execute_tool(self, name: str, input: dict) -> dict:
        """Thực hiện một công cụ và trả về kết quả."""

        if name == "computer":
            return self._handle_computer_action(input)
        elif name == "bash":
            return self._handle_bash(input)
        elif name == "str_replace_editor":
            return self._handle_editor(input)
        else:
            return {"error": f"Unknown tool: {name}"}

    def _handle_computer_action(self, input: dict) -> dict:
        """Xử lý các hành động điều khiển máy tính."""
        action = input.get("action")

        if action == "screenshot":
            # Chụp qua xdotool/scrot
            subprocess.run(["scrot", "/tmp/screenshot.png"])

            with open("/tmp/screenshot.png", "rb") as f:

```

## ⚠️ Các Cạnh Sắc (Sharp Edges)

| Vấn đề | Mức độ nghiêm trọng | Giải pháp                                                         |
| ------ | ------------------- | ----------------------------------------------------------------- |
| Vấn đề | nghiêm trọng        | ## Phòng thủ chuyên sâu - không có giải pháp đơn lẻ nào hoạt động |
| Vấn đề | trung bình          | ## Thêm phương sai giống con người vào các hành động              |
| Vấn đề | cao                 | ## Sử dụng các lựa chọn thay thế bàn phím khi có thể              |
| Vấn đề | trung bình          | ## Chấp nhận sự đánh đổi                                          |
| Vấn đề | cao                 | ## Triển khai quản lý bối cảnh                                    |
| Vấn đề | cao                 | ## Giám sát và giới hạn chi phí                                   |
| Vấn đề | nghiêm trọng        | ## LUÔN LUÔN sử dụng sandbox                                      |
