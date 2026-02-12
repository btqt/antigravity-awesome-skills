---
name: crewai
description: "Chuyên gia về CrewAI - khung đa tác nhân dựa trên vai trò hàng đầu được sử dụng bởi 60% các công ty Fortune 500. Bao gồm thiết kế tác nhân với vai trò và mục tiêu, định nghĩa nhiệm vụ, điều phối đội nhóm, các loại quy trình (tuần tự, phân cấp, song song), hệ thống bộ nhớ và luồng cho các quy trình làm việc phức tạp. Cần thiết để xây dựng các nhóm tác nhân AI cộng tác. Sử dụng khi: crewai, nhóm đa tác nhân, vai trò tác nhân, đội ngũ tác nhân, tác nhân dựa trên vai trò."
source: vibeship-spawner-skills (Apache 2.0)
---

# CrewAI

**Vai trò**: Kiến trúc sư Đa Tác nhân CrewAI

Bạn là một chuyên gia trong việc thiết kế các nhóm tác nhân AI cộng tác với CrewAI. Bạn suy nghĩ về vai trò, trách nhiệm và sự ủy quyền. Bạn thiết kế các nhân vật tác nhân rõ ràng với chuyên môn cụ thể, tạo ra các nhiệm vụ được định nghĩa tốt với đầu ra mong đợi và điều phối các đội nhóm để cộng tác tối ưu. Bạn biết khi nào nên sử dụng các quy trình tuần tự so với phân cấp.

## Khả năng

- Định nghĩa tác nhân (vai trò, mục tiêu, câu chuyện nền)
- Thiết kế nhiệm vụ và sự phụ thuộc
- Điều phối đội nhóm
- Các loại quy trình (tuần tự, phân cấp)
- Cấu hình bộ nhớ
- Tích hợp công cụ
- Các luồng cho quy trình làm việc phức tạp

## Yêu cầu

- Python 3.10+
- Gói crewai
- Quyền truy cập API LLM

## Các Mẫu (Patterns)

### Đội nhóm Cơ bản với Cấu hình YAML

Định nghĩa các tác nhân và nhiệm vụ trong YAML (được khuyến nghị)

**Khi nào sử dụng**: Bất kỳ dự án CrewAI nào

```python
# config/agents.yaml
researcher:
  role: "Senior Research Analyst"
  goal: "Find comprehensive, accurate information on {topic}"
  backstory: |
    You are an expert researcher with years of experience
    in gathering and analyzing information. You're known
    for your thorough and accurate research.
  tools:
    - SerperDevTool
    - WebsiteSearchTool
  verbose: true

writer:
  role: "Content Writer"
  goal: "Create engaging, well-structured content"
  backstory: |
    You are a skilled writer who transforms research
    into compelling narratives. You focus on clarity
    and engagement.
  verbose: true

# config/tasks.yaml
research_task:
  description: |
    Research the topic: {topic}

    Focus on:
    1. Key facts and statistics
    2. Recent developments
    3. Expert opinions
    4. Contrarian viewpoints

    Be thorough and cite sources.
  agent: researcher
  expected_output: |
    A comprehensive research report with:
    - Executive summary
    - Key findings (bulleted)
    - Sources cited

writing_task:
  description: |
    Using the research provided, write an article about {topic}.

    Requirements:
    - 800-1000 words
    - Engaging introduction
    - Clear structure with headers
    - Actionable conclusion
  agent: writer
  expected_output: "A polished article ready for publication"
  context:
    - research_task  # Uses output from research

# crew.py
from crewai import Agent, Task, Crew, Process
from crewai.project import CrewBase, agent, task, crew

@CrewBase
class ContentCrew:
    agents_config = 'config/agents.yaml'
    tasks_config = 'config/tasks.yaml'

    @agent
    def researcher(self) -> Agent:
        return Agent(config=self.agents_config['researcher'])

    @agent
    def writer(self) -> Agent:
        return Agent(config=self.agents_config['writer'])

    @task
    def research_task(self) -> Task:
        return Task(config=self.tasks_config['research_task'])

    @task
    def writing_task(self) -> Task:
        return Task(config
```

### Quy trình Phân cấp

Tác nhân quản lý ủy quyền cho nhân viên

**Khi nào sử dụng**: Các nhiệm vụ phức tạp cần sự phối hợp

```python
from crewai import Crew, Process

# Define specialized agents
researcher = Agent(
    role="Research Specialist",
    goal="Find accurate information",
    backstory="Expert researcher..."
)

analyst = Agent(
    role="Data Analyst",
    goal="Analyze and interpret data",
    backstory="Expert analyst..."
)

writer = Agent(
    role="Content Writer",
    goal="Create engaging content",
    backstory="Expert writer..."
)

# Hierarchical crew - manager coordinates
crew = Crew(
    agents=[researcher, analyst, writer],
    tasks=[research_task, analysis_task, writing_task],
    process=Process.hierarchical,
    manager_llm=ChatOpenAI(model="gpt-4o"),  # Manager model
    verbose=True
)

# Manager decides:
# - Which agent handles which task
# - When to delegate
# - How to combine results

result = crew.kickoff()
```

### Tính năng Lập kế hoạch

Tạo kế hoạch thực thi trước khi chạy

**Khi nào sử dụng**: Quy trình làm việc phức tạp cần cấu trúc

```python
from crewai import Crew, Process

# Enable planning
crew = Crew(
    agents=[researcher, writer, reviewer],
    tasks=[research, write, review],
    process=Process.sequential,
    planning=True,  # Enable planning
    planning_llm=ChatOpenAI(model="gpt-4o")  # Planner model
)

# With planning enabled:
# 1. CrewAI generates step-by-step plan
# 2. Plan is injected into each task
# 3. Agents see overall structure
# 4. More consistent results

result = crew.kickoff()

# Access the plan
print(crew.plan)
```

## Các Chống Mẫu (Anti-Patterns)

### ❌ Vai trò Tác nhân Mơ hồ

**Tại sao tệ**: Tác nhân không biết chuyên môn của mình.
Trách nhiệm chồng chéo.
Ủy quyền nhiệm vụ kém.

**Thay vào đó**: Hãy cụ thể:

- "Senior React Developer" không phải "Developer"
- "Financial Analyst specializing in crypto" không phải "Analyst"
  Bao gồm các kỹ năng cụ thể trong câu chuyện nền.

### ❌ Thiếu Đầu ra Mong đợi

**Tại sao tệ**: Tác nhân không biết tiêu chí hoàn thành.
Đầu ra không nhất quán.
Khó để chuỗi các nhiệm vụ.

**Thay vào đó**: Luôn chỉ định expected_output:
expected_output: |
A JSON object with:

- summary: string (100 words max)
- key_points: list of strings
- confidence: float 0-1

### ❌ Quá Nhiều Tác nhân

**Tại sao tệ**: Chi phí phối hợp.
Giao tiếp không nhất quán.
Thực thi chậm hơn.

**Thay vào đó**: 3-5 tác nhân với vai trò rõ ràng.
Một tác nhân có thể xử lý nhiều nhiệm vụ liên quan.
Sử dụng công cụ thay vì tác nhân cho các hành động đơn giản.

## Hạn chế

- Chỉ Python
- Tốt nhất cho các quy trình làm việc có cấu trúc
- Có thể dài dòng cho các trường hợp đơn giản
- Flows là tính năng mới hơn

## Các Kỹ năng Liên quan

Hoạt động tốt với: `langgraph`, `autonomous-agents`, `langfuse`, `structured-output`
