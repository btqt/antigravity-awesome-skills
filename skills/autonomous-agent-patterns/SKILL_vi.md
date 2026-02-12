---
name: autonomous-agent-patterns
description: "Các mẫu thiết kế để xây dựng các agent lập trình tự hành. Bao gồm tích hợp công cụ, hệ thống quyền hạn, tự động hóa trình duyệt và các quy trình công việc có yếu tố con người (human-in-the-loop). Sử dụng khi xây dựng các AI agent, thiết kế các API cho công cụ, triển khai hệ thống quyền hạn, hoặc tạo ra các trợ lý lập trình tự hành."
---

# 🕹️ Các mẫu Agent Tự hành (Autonomous Agent Patterns)

> Các mẫu thiết kế để xây dựng các agent lập trình tự hành, được truyền cảm hứng bởi [Cline](https://github.com/cline/cline) và [OpenAI Codex](https://github.com/openai/codex).

## Khi nào sử dụng Kỹ năng này

Sử dụng kỹ năng này khi:

- Xây dựng các AI agent tự hành
- Thiết kế các API gọi công cụ/hàm (tool/function calling)
- Triển khai hệ thống quyền hạn và phê duyệt
- Tạo hệ thống tự động hóa trình duyệt cho các agent
- Thiết kế các quy trình công việc có yếu tố con người (human-in-the-loop)

---

## 1. Kiến trúc Agent Cốt lõi (Core Agent Architecture)

### 1.1 Vòng lặp Agent (Agent Loop)

```
┌─────────────────────────────────────────────────────────────┐
│                     VÒNG LẶP AGENT                           │
│                                                              │
│  ┌──────────┐    ┌──────────┐    ┌──────────┐              │
│  │   Suy    │───▶│  Quyết   │───▶│   Hành   │              │
│  │  nghĩ    │    │   định   │    │   động   │              │
│  └──────────┘    └──────────┘    └──────────┘              │
│       ▲                               │                     │
│       │         ┌──────────┐          │                     │
│       └─────────│ Quan sát │◀─────────┘                     │
│                 │ (Kết quả)│                                │
│                 └──────────┘                                │
└─────────────────────────────────────────────────────────────┘
```

```python
class AgentLoop:
    def __init__(self, llm, tools, max_iterations=50):
        self.llm = llm
        self.tools = {t.name: t for t in tools}
        self.max_iterations = max_iterations
        self.history = []

    def run(self, task: str) -> str:
        self.history.append({"role": "user", "content": task})

        for i in range(self.max_iterations):
            # Suy nghĩ: Lấy phản hồi từ LLM với các tùy chọn công cụ
            response = self.llm.chat(
                messages=self.history,
                tools=self._format_tools(),
                tool_choice="auto"
            )

            # Quyết định: Kiểm tra xem agent có muốn sử dụng công cụ nào không
            if response.tool_calls:
                for tool_call in response.tool_calls:
                    # Hành động: Thực thi công cụ
                    result = self._execute_tool(tool_call)

                    # Quan sát: Thêm kết quả vào lịch sử
                    self.history.append({
                        "role": "tool",
                        "tool_call_id": tool_call.id,
                        "content": str(result)
                    })
            else:
                # Không còn lệnh gọi công cụ nữa = nhiệm vụ hoàn thành
                return response.content

        return "Đã đạt đến số lần lặp tối đa"

    def _execute_tool(self, tool_call) -> Any:
        tool = self.tools[tool_call.name]
        args = json.loads(tool_call.arguments)
        return tool.execute(**args)
```

### 1.2 Kiến trúc Đa mô hình (Multi-Model Architecture)

```python
class MultiModelAgent:
    """
    Sử dụng các mô hình khác nhau cho các mục đích khác nhau:
    - Mô hình nhanh để lập kế hoạch
    - Mô hình mạnh cho suy luận phức tạp
    - Mô hình chuyên biệt cho tạo mã nguồn
    """

    def __init__(self):
        self.models = {
            "fast": "gpt-3.5-turbo",      # Quyết định nhanh
            "smart": "gpt-4-turbo",        # Suy luận phức tạp
            "code": "claude-3-sonnet",     # Tạo mã nguồn
        }

    def select_model(self, task_type: str) -> str:
        if task_type == "planning":
            return self.models["fast"]
        elif task_type == "analysis":
            return self.models["smart"]
        elif task_type == "code":
            return self.models["code"]
        return self.models["smart"]
```

---

## 2. Các mẫu Thiết kế Công cụ (Tool Design Patterns)

### 2.1 Sơ đồ Công cụ (Tool Schema)

```python
class Tool:
    """Lớp cơ sở cho các công cụ của agent"""

    @property
    def schema(self) -> dict:
        """JSON Schema cho công cụ"""
        return {
            "name": self.name,
            "description": self.description,
            "parameters": {
                "type": "object",
                "properties": self._get_parameters(),
                "required": self._get_required()
            }
        }

    def execute(self, **kwargs) -> ToolResult:
        """Thực thi công cụ và trả về kết quả"""
        raise NotImplementedError

class ReadFileTool(Tool):
    name = "read_file"
    description = "Đọc nội dung của một tệp từ hệ thống tệp"

    def _get_parameters(self):
        return {
            "path": {
                "type": "string",
                "description": "Đường dẫn tuyệt đối đến tệp"
            },
            "start_line": {
                "type": "integer",
                "description": "Dòng bắt đầu đọc (bắt đầu từ 1)"
            },
            "end_line": {
                "type": "integer",
                "description": "Dòng kết thúc đọc (bao gồm dòng này)"
            }
        }

    def _get_required(self):
        return ["path"]

    def execute(self, path: str, start_line: int = None, end_line: int = None) -> ToolResult:
        try:
            with open(path, 'r') as f:
                lines = f.readlines()

            if start_line and end_line:
                lines = lines[start_line-1:end_line]

            return ToolResult(
                success=True,
                output="".join(lines)
            )
        except FileNotFoundError:
            return ToolResult(
                success=False,
                error=f"Không tìm thấy tệp: {path}"
            )
```

### 2.2 Các Công cụ Agent Cơ bản

```python
CODING_AGENT_TOOLS = {
    # Hoạt động với tệp
    "read_file": "Đọc nội dung tệp",
    "write_file": "Tạo mới hoặc ghi đè một tệp",
    "edit_file": "Thực hiện các chỉnh sửa mục tiêu cho một tệp",
    "list_directory": "Liệt kê các tệp và thư mục",
    "search_files": "Tìm kiếm các tệp theo mẫu",

    # Hiểu mã nguồn
    "search_code": "Tìm kiếm các mẫu mã nguồn (grep)",
    "get_definition": "Tìm định nghĩa hàm/lớp",
    "get_references": "Tìm tất cả các tham chiếu đến một ký hiệu",

    # Terminal
    "run_command": "Thực thi một lệnh shell",
    "read_output": "Đọc đầu ra của lệnh",
    "send_input": "Gửi đầu vào cho lệnh đang chạy",

    # Trình duyệt (tùy chọn)
    "open_browser": "Mở URL trong trình duyệt",
    "click_element": "Nhấp vào phần tử trang",
    "type_text": "Nhập văn bản vào ô nhập liệu",
    "screenshot": "Chụp ảnh màn hình",

    # Ngữ cảnh
    "ask_user": "Đặt câu hỏi cho người dùng",
    "search_web": "Tìm kiếm thông tin trên web"
}
```

### 2.3 Thiết kế Công cụ Chỉnh sửa (Edit Tool Design)

```python
class EditFileTool(Tool):
    """
    Chỉnh sửa tệp chính xác với tính năng phát hiện xung đột.
    Sử dụng mẫu tìm kiếm/thay thế để chỉnh sửa đáng tin cậy.
    """

    name = "edit_file"
    description = "Chỉnh sửa tệp bằng cách thay thế nội dung cụ thể"

    def execute(
        self,
        path: str,
        search: str,
        replace: str,
        expected_occurrences: int = 1
    ) -> ToolResult:
        """
        Đối số:
            path: Tệp cần chỉnh sửa
            search: Văn bản chính xác cần tìm (phải khớp hoàn toàn, bao gồm cả khoảng trắng)
            replace: Văn bản để thay thế vào
            expected_occurrences: Số lần văn bản tìm kiếm xuất hiện (để xác thực)
        """
        with open(path, 'r') as f:
            content = f.read()

        # Xác thực
        actual_occurrences = content.count(search)
        if actual_occurrences != expected_occurrences:
            return ToolResult(
                success=False,
                error=f"Dự kiến {expected_occurrences} lần xuất hiện, nhưng tìm thấy {actual_occurrences}"
            )

        if actual_occurrences == 0:
            return ToolResult(
                success=False,
                error="Không tìm thấy văn bản cần tìm trong tệp"
            )

        # Áp dụng chỉnh sửa
        new_content = content.replace(search, replace)

        with open(path, 'w') as f:
            f.write(new_content)

        return ToolResult(
            success=True,
            output=f"Đã thay thế {actual_occurrences} lần xuất hiện"
        )
```

---

## 3. Các mẫu về Quyền hạn & An toàn (Permission & Safety Patterns)

### 3.1 Các Cấp độ Quyền hạn

```python
class PermissionLevel(Enum):
    # Hoàn toàn tự động - không cần người dùng phê duyệt
    AUTO = "auto"

    # Hỏi một lần mỗi phiên làm việc
    ASK_ONCE = "ask_once"

    # Hỏi mỗi lần thực hiện
    ASK_EACH = "ask_each"

    # Không bao giờ cho phép
    NEVER = "never"

PERMISSION_CONFIG = {
    # Rủi ro thấp - có thể tự động phê duyệt
    "read_file": PermissionLevel.AUTO,
    "list_directory": PermissionLevel.AUTO,
    "search_code": PermissionLevel.AUTO,

    # Rủi ro trung bình - hỏi một lần
    "write_file": PermissionLevel.ASK_ONCE,
    "edit_file": PermissionLevel.ASK_ONCE,

    # Rủi ro cao - hỏi mỗi lần
    "run_command": PermissionLevel.ASK_EACH,
    "delete_file": PermissionLevel.ASK_EACH,

    # Nguy hiểm - không bao giờ tự động phê duyệt
    "sudo_command": PermissionLevel.NEVER,
    "format_disk": PermissionLevel.NEVER
}
```

### 3.2 Mẫu giao diện phê duyệt (Approval UI Pattern)

```python
class ApprovalManager:
    def __init__(self, ui, config):
        self.ui = ui
        self.config = config
        self.session_approvals = {}

    def request_approval(self, tool_name: str, args: dict) -> bool:
        level = self.config.get(tool_name, PermissionLevel.ASK_EACH)

        if level == PermissionLevel.AUTO:
            return True

        if level == PermissionLevel.NEVER:
            self.ui.show_error(f"Công cụ '{tool_name}' không được cho phép")
            return False

        if level == PermissionLevel.ASK_ONCE:
            if tool_name in self.session_approvals:
                return self.session_approvals[tool_name]

        # Hiển thị hộp thoại phê duyệt
        approved = self.ui.show_approval_dialog(
            tool=tool_name,
            args=args,
            risk_level=self._assess_risk(tool_name, args)
        )

        if level == PermissionLevel.ASK_ONCE:
            self.session_approvals[tool_name] = approved

        return approved

    def _assess_risk(self, tool_name: str, args: dict) -> str:
        """Phân tích lệnh gọi cụ thể để xác định mức độ rủi ro"""
        if tool_name == "run_command":
            cmd = args.get("command", "")
            if any(danger in cmd for danger in ["rm -rf", "sudo", "chmod"]):
                return "HIGH"
        return "MEDIUM"
```

### 3.3 Môi trường cô lập (Sandboxing)

```python
class SandboxedExecution:
    """
    Thực thi mã/lệnh trong môi trường cô lập
    """

    def __init__(self, workspace_dir: str):
        self.workspace = workspace_dir
        self.allowed_commands = ["npm", "python", "node", "git", "ls", "cat"]
        self.blocked_paths = ["/etc", "/usr", "/bin", os.path.expanduser("~")]

    def validate_path(self, path: str) -> bool:
        """Đảm bảo đường dẫn nằm trong không gian làm việc"""
        real_path = os.path.realpath(path)
        workspace_real = os.path.realpath(self.workspace)
        return real_path.startswith(workspace_real)

    def validate_command(self, command: str) -> bool:
        """Kiểm tra xem lệnh có được cho phép không"""
        cmd_parts = shlex.split(command)
        if not cmd_parts:
            return False

        base_cmd = cmd_parts[0]
        return base_cmd in self.allowed_commands

    def execute_sandboxed(self, command: str) -> ToolResult:
        if not self.validate_command(command):
            return ToolResult(
                success=False,
                error=f"Lệnh không được cho phép: {command}"
            )

        # Thực thi trong môi trường cô lập
        result = subprocess.run(
            command,
            shell=True,
            cwd=self.workspace,
            capture_output=True,
            timeout=30,
            env={
                **os.environ,
                "HOME": self.workspace,  # Cô lập thư mục home
            }
        )

        return ToolResult(
            success=result.returncode == 0,
            output=result.stdout.decode(),
            error=result.stderr.decode() if result.returncode != 0 else None
        )
```

---

## 4. Tự động hóa Trình duyệt (Browser Automation)

### 4.1 Mẫu Công cụ Trình duyệt (Browser Tool Pattern)

```python
class BrowserTool:
    """
    Tự động hóa trình duyệt cho các agent sử dụng Playwright/Puppeteer.
    Cho phép gỡ lỗi trực quan và kiểm thử web.
    """

    def __init__(self, headless: bool = True):
        self.browser = None
        self.page = None
        self.headless = headless

    async def open_url(self, url: str) -> ToolResult:
        """Điều hướng đến URL và trả về thông tin trang"""
        if not self.browser:
            self.browser = await playwright.chromium.launch(headless=self.headless)
            self.page = await self.browser.new_page()

        await self.page.goto(url)

        # Chụp trạng thái
        screenshot = await self.page.screenshot(type='png')
        title = await self.page.title()

        return ToolResult(
            success=True,
            output=f"Đã tải: {title}",
            metadata={
                "screenshot": base64.b64encode(screenshot).decode(),
                "url": self.page.url
            }
        )

    async def click(self, selector: str) -> ToolResult:
        """Nhấp vào một phần tử"""
        try:
            await self.page.click(selector, timeout=5000)
            await self.page.wait_for_load_state("networkidle")

            screenshot = await self.page.screenshot()
            return ToolResult(
                success=True,
                output=f"Đã nhấp: {selector}",
                metadata={"screenshot": base64.b64encode(screenshot).decode()}
            )
        except TimeoutError:
            return ToolResult(
                success=False,
                error=f"Không tìm thấy phần tử: {selector}"
            )

    async def type_text(self, selector: str, text: str) -> ToolResult:
        """Nhập văn bản vào một ô nhập liệu"""
        await self.page.fill(selector, text)
        return ToolResult(success=True, output=f"Đã nhập vào {selector}")

    async def get_page_content(self) -> ToolResult:
        """Lấy nội dung văn bản có thể truy cập của trang"""
        content = await self.page.evaluate("""
            () => {
                // Lấy văn bản hiển thị
                const walker = document.createTreeWalker(
                    document.body,
                    NodeFilter.SHOW_TEXT,
                    null,
                    false
                );

                let text = '';
                while (walker.nextNode()) {
                    const node = walker.currentNode;
                    if (node.textContent.trim()) {
                        text += node.textContent.trim() + '\\n';
                    }
                }
                return text;
            }
        """)
        return ToolResult(success=True, output=content)
```

### 4.2 Mẫu Agent Trực quan (Visual Agent Pattern)

```python
class VisualAgent:
    """
    Agent sử dụng ảnh màn hình để hiểu các trang web.
    Có thể xác định các phần tử trực quan mà không cần selector.
    """

    def __init__(self, llm, browser):
        self.llm = llm
        self.browser = browser

    async def describe_page(self) -> str:
        """Sử dụng mô hình vision để mô tả trang hiện tại"""
        screenshot = await self.browser.screenshot()

        response = self.llm.chat([
            {
                "role": "user",
                "content": [
                    {"type": "text", "text": "Mô tả trang web này. Liệt kê tất cả các phần tử tương tác mà bạn thấy."},
                    {"type": "image", "data": screenshot}
                ]
            }
        ])

        return response.content

    async def find_and_click(self, description: str) -> ToolResult:
        """Tìm phần tử theo mô tả trực quan và nhấp vào nó"""
        screenshot = await self.browser.screenshot()

        # Yêu cầu mô hình vision tìm phần tử
        response = self.llm.chat([
            {
                "role": "user",
                "content": [
                    {
                        "type": "text",
                        "text": f"""
                        Tìm phần tử khớp với: "{description}"
                        Trả về tọa độ xấp xỉ dưới dạng JSON: {{"x": number, "y": number}}
                        """
                    },
                    {"type": "image", "data": screenshot}
                ]
            }
        ])

        coords = json.loads(response.content)
        await self.browser.page.mouse.click(coords["x"], coords["y"])

        return ToolResult(success=True, output=f"Đã nhấp tại ({coords['x']}, {coords['y']})")
```

---

## 5. Quản lý Ngữ cảnh (Context Management)

### 5.1 Các mẫu Chèn Ngữ cảnh (Context Injection Patterns)

````python
class ContextManager:
    """
    Quản lý ngữ cảnh được cung cấp cho agent.
    Lấy cảm hứng từ các mẫu @-mention của Cline.
    """

    def __init__(self, workspace: str):
        self.workspace = workspace
        self.context = []

    def add_file(self, path: str) -> None:
        """@file - Thêm nội dung tệp vào ngữ cảnh"""
        with open(path, 'r') as f:
            content = f.read()

        self.context.append({
            "type": "file",
            "path": path,
            "content": content
        })

    def add_folder(self, path: str, max_files: int = 20) -> None:
        """@folder - Thêm tất cả các tệp trong thư mục"""
        for root, dirs, files in os.walk(path):
            for file in files[:max_files]:
                file_path = os.path.join(root, file)
                self.add_file(file_path)

    def add_url(self, url: str) -> None:
        """@url - Truy cập và thêm nội dung URL"""
        response = requests.get(url)
        content = html_to_markdown(response.text)

        self.context.append({
            "type": "url",
            "url": url,
            "content": content
        })

    def add_problems(self, diagnostics: list) -> None:
        """@problems - Thêm các chẩn đoán (diagnostics) từ IDE"""
        self.context.append({
            "type": "diagnostics",
            "problems": diagnostics
        })

    def format_for_prompt(self) -> str:
        """Định dạng tất cả ngữ cảnh cho prompt của LLM"""
        parts = []
        for item in self.context:
            if item["type"] == "file":
                parts.append(f"## Tệp: {item['path']}\n```\n{item['content']}\n```")
            elif item["type"] == "url":
                parts.append(f"## URL: {item['url']}\n{item['content']}")
            elif item["type"] == "diagnostics":
                parts.append(f"## Các vấn đề:\n{json.dumps(item['problems'], indent=2)}")

        return "\n\n".join(parts)
````

### 5.2 Điểm kiểm tra/Tiếp tục (Checkpoint/Resume)

```python
class CheckpointManager:
    """
    Lưu và khôi phục trạng thái agent cho các tác vụ chạy lâu.
    """

    def __init__(self, storage_dir: str):
        self.storage_dir = storage_dir
        os.makedirs(storage_dir, exist_ok=True)

    def save_checkpoint(self, session_id: str, state: dict) -> str:
        """Lưu trạng thái agent hiện tại"""
        checkpoint = {
            "timestamp": datetime.now().isoformat(),
            "session_id": session_id,
            "history": state["history"],
            "context": state["context"],
            "workspace_state": self._capture_workspace(state["workspace"]),
            "metadata": state.get("metadata", {})
        }

        path = os.path.join(self.storage_dir, f"{session_id}.json")
        with open(path, 'w') as f:
            json.dump(checkpoint, f, indent=2)

        return path

    def restore_checkpoint(self, checkpoint_path: str) -> dict:
        """Khôi phục trạng thái agent từ điểm kiểm tra"""
        with open(checkpoint_path, 'r') as f:
            checkpoint = json.load(f)

        return {
            "history": checkpoint["history"],
            "context": checkpoint["context"],
            "workspace": self._restore_workspace(checkpoint["workspace_state"]),
            "metadata": checkpoint["metadata"]
        }

    def _capture_workspace(self, workspace: str) -> dict:
        """Chụp lại trạng thái không gian làm việc có liên quan"""
        # Trạng thái Git, hash tệp, v.v.
        return {
            "git_ref": subprocess.getoutput(f"cd {workspace} && git rev-parse HEAD"),
            "git_dirty": subprocess.getoutput(f"cd {workspace} && git status --porcelain")
        }
```

---

## 6. Tích hợp MCP (Model Context Protocol)

### 6.1 Mẫu Máy chủ MCP (MCP Server Pattern)

```python
from mcp import Server, Tool

class MCPAgent:
    """
    Agent có thể khám phá động và sử dụng các công cụ MCP.
    Mẫu 'Thêm một công cụ để...' từ Cline.
    """

    def __init__(self, llm):
        self.llm = llm
        self.mcp_servers = {}
        self.available_tools = {}

    def connect_server(self, name: str, config: dict) -> None:
        """Kết nối đến một máy chủ MCP"""
        server = Server(config)
        self.mcp_servers[name] = server

        # Khám phá các công cụ
        tools = server.list_tools()
        for tool in tools:
            self.available_tools[tool.name] = {
                "server": name,
                "schema": tool.schema
            }

    async def create_tool(self, description: str) -> str:
        """
        Tạo một máy chủ MCP mới dựa trên mô tả của người dùng.
        Ví dụ: 'Thêm một công cụ lấy các ticket từ Jira'
        """
        # Tạo mã nguồn máy chủ MCP
        code = self.llm.generate(f"""
        Tạo một máy chủ MCP Python với một công cụ thực hiện:
        {description}

        Sử dụng framework FastMCP. Bao gồm xử lý lỗi đúng cách.
        Chỉ trả về mã nguồn Python.
        """)

        # Lưu và cài đặt
        server_name = self._extract_name(description)
        path = f"./mcp_servers/{server_name}/server.py"

        with open(path, 'w') as f:
            f.write(code)

        # Tải lại nóng (Hot-reload)
        self.connect_server(server_name, {"path": path})

        return f"Đã tạo công cụ: {server_name}"
```

---

## Danh sách kiểm tra Thực hành Tốt nhất

### Thiết kế Agent

- [ ] Phân rã nhiệm vụ rõ ràng
- [ ] Độ chi tiết của công cụ phù hợp
- [ ] Xử lý lỗi tại mỗi bước
- [ ] Người dùng có thể thấy được tiến trình

### An toàn

- [ ] Hệ thống quyền hạn đã được triển khai
- [ ] Các hoạt động nguy hiểm đã bị chặn
- [ ] Môi trường cô lập (Sandbox) cho mã nguồn chưa được tin tưởng
- [ ] Ghi nhật ký kiểm tra (Audit logging) đã bật

### Trải nghiệm người dùng (UX)

- [ ] Giao diện phê duyệt rõ ràng
- [ ] Có cung cấp cập nhật tiến trình
- [ ] Có sẵn tính năng Hoàn tác/Khôi phục (Undo/rollback)
- [ ] Có giải thích về các hành động

---

## Tài nguyên

- [Cline](https://github.com/cline/cline)
- [OpenAI Codex](https://github.com/openai/codex)
- [Model Context Protocol](https://modelcontextprotocol.io/)
- [Anthropic Tool Use](https://docs.anthropic.com/claude/docs/tool-use)
