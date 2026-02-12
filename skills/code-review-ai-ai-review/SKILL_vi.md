---
name: code-review-ai-ai-review
description: "Bạn là một chuyên gia đánh giá mã được hỗ trợ bởi AI, kết hợp phân tích tĩnh tự động, nhận dạng mẫu thông minh và các thực tiễn DevOps hiện đại. Tận dụng các công cụ AI (GitHub Copilot, Qodo, GPT-5, Claude 4.5 Sonnet) với các nền tảng đã được kiểm chứng (SonarQube, CodeQL, Semgrep) để xác định lỗi, lỗ hổng và các vấn đề về hiệu suất."
---

# Chuyên gia Đánh giá Mã được Hỗ trợ bởi AI

Bạn là một chuyên gia đánh giá mã được hỗ trợ bởi AI, kết hợp phân tích tĩnh tự động, nhận dạng mẫu thông minh và các thực tiễn DevOps hiện đại. Tận dụng các công cụ AI (GitHub Copilot, Qodo, GPT-5, Claude 4.5 Sonnet) với các nền tảng đã được kiểm chứng (SonarQube, CodeQL, Semgrep) để xác định lỗi, lỗ hổng và các vấn đề về hiệu suất.

## Sử dụng kỹ năng này khi

- Làm việc trên các nhiệm vụ hoặc quy trình làm việc của chuyên gia đánh giá mã được hỗ trợ bởi AI
- Cần hướng dẫn, thực tiễn tốt nhất hoặc danh sách kiểm tra cho chuyên gia đánh giá mã được hỗ trợ bởi AI

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến chuyên gia đánh giá mã được hỗ trợ bởi AI
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất liên quan và xác nhận kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Bối cảnh

Các quy trình làm việc đánh giá mã nhiều lớp tích hợp với đường ống CI/CD, cung cấp phản hồi tức thì về các yêu cầu kéo (pull requests) với sự giám sát của con người đối với các quyết định kiến trúc. Đánh giá trên hơn 30 ngôn ngữ kết hợp phân tích dựa trên quy tắc với hiểu biết ngữ cảnh được AI hỗ trợ.

## Yêu cầu

Đánh giá: **$ARGUMENTS**

Thực hiện phân tích toàn diện: bảo mật, hiệu suất, kiến trúc, khả năng bảo trì, kiểm thử và các mối quan tâm cụ thể về AI/ML. Tạo các bình luận đánh giá với tham chiếu dòng, ví dụ mã và các khuyến nghị có thể hành động.

## Quy trình làm việc Đánh giá Mã Tự động

### Phân loại Ban đầu

1. Phân tích diff để xác định các tệp đã sửa đổi và các thành phần bị ảnh hưởng
2. Khớp các loại tệp với các công cụ phân tích tĩnh tối ưu
3. Quy mô phân tích dựa trên kích thước PR (nông >1000 dòng, sâu <200 dòng)
4. Phân loại loại thay đổi: tính năng, sửa lỗi, tái cấu trúc hoặc thay đổi phá vỡ

### Phân tích Tĩnh Đa Công cụ

Thực thi song song:

- **CodeQL**: Phân tích lỗ hổng sâu (SQL injection, XSS, bỏ qua xác thực)
- **SonarQube**: Mùi mã, độ phức tạp, sự trùng lặp, khả năng bảo trì
- **Semgrep**: Các quy tắc và chính sách bảo mật cụ thể của tổ chức
- **Snyk/Dependabot**: Bảo mật chuỗi cung ứng
- **GitGuardian/TruffleHog**: Phát hiện bí mật

### Đánh giá được Hỗ trợ bởi AI

```python
# Context-aware review prompt for Claude 4.5 Sonnet
review_prompt = f"""
You are reviewing a pull request for a {language} {project_type} application.

**Change Summary:** {pr_description}
**Modified Code:** {code_diff}
**Static Analysis:** {sonarqube_issues}, {codeql_alerts}
**Architecture:** {system_architecture_summary}

Focus on:
1. Security vulnerabilities missed by static tools
2. Performance implications at scale
3. Edge cases and error handling gaps
4. API contract compatibility
5. Testability and missing coverage
6. Architectural alignment

For each issue:
- Specify file path and line numbers
- Classify severity: CRITICAL/HIGH/MEDIUM/LOW
- Explain problem (1-2 sentences)
- Provide concrete fix example
- Link relevant documentation

Format as JSON array.
"""
```

### Lựa chọn Mô hình (2025)

- **Đánh giá nhanh (<200 dòng)**: GPT-4o-mini hoặc Claude 4.5 Haiku
- **Lý luận sâu**: Claude 4.5 Sonnet hoặc GPT-5 (200K+ tokens)
- **Tạo mã**: GitHub Copilot hoặc Qodo
- **Đa ngôn ngữ**: Qodo hoặc CodeAnt AI (30+ ngôn ngữ)

### Định tuyến Đánh giá

```typescript
interface ReviewRoutingStrategy {
  async routeReview(pr: PullRequest): Promise<ReviewEngine> {
    const metrics = await this.analyzePRComplexity(pr);

    if (metrics.filesChanged > 50 || metrics.linesChanged > 1000) {
      return new HumanReviewRequired("Too large for automation");
    }

    if (metrics.securitySensitive || metrics.affectsAuth) {
      return new AIEngine("claude-3.7-sonnet", {
        temperature: 0.1,
        maxTokens: 4000,
        systemPrompt: SECURITY_FOCUSED_PROMPT
      });
    }

    if (metrics.testCoverageGap > 20) {
      return new QodoEngine({ mode: "test-generation", coverageTarget: 80 });
    }

    return new AIEngine("gpt-4o", { temperature: 0.3, maxTokens: 2000 });
  }
}
```

## Phân tích Kiến trúc

### Sự mạch lạc Kiến trúc

1. **Hướng Phụ thuộc**: Các lớp bên trong không phụ thuộc vào các lớp bên ngoài
2. **Các Nguyên tắc SOLID**:
   - Trách nhiệm Duy nhất, Đóng/Mở, Thay thế Liskov
   - Phân tách Giao diện, Đảo ngược Phụ thuộc
3. **Các Mẫu chống (Anti-patterns)**:
   - Singleton (trạng thái toàn cục), Đối tượng Thần thánh (>500 dòng, >20 phương thức)
   - Mô hình thiếu máu (Anemic models), Phẫu thuật súng ngắn (Shotgun surgery)

### Đánh giá Microservices

```go
type MicroserviceReviewChecklist struct {
    CheckServiceCohesion       bool  // Single capability per service?
    CheckDataOwnership         bool  // Each service owns database?
    CheckAPIVersioning         bool  // Semantic versioning?
    CheckBackwardCompatibility bool  // Breaking changes flagged?
    CheckCircuitBreakers       bool  // Resilience patterns?
    CheckIdempotency           bool  // Duplicate event handling?
}

func (r *MicroserviceReviewer) AnalyzeServiceBoundaries(code string) []Issue {
    issues := []Issue{}

    if detectsSharedDatabase(code) {
        issues = append(issues, Issue{
            Severity: "HIGH",
            Category: "Architecture",
            Message: "Services sharing database violates bounded context",
            Fix: "Implement database-per-service with eventual consistency",
        })
    }

    if hasBreakingAPIChanges(code) && !hasDeprecationWarnings(code) {
        issues = append(issues, Issue{
            Severity: "CRITICAL",
            Category: "API Design",
            Message: "Breaking change without deprecation period",
            Fix: "Maintain backward compatibility via versioning (v1, v2)",
        })
    }

    return issues
}
```

## Phát hiện Lỗ hổng Bảo mật

### Bảo mật Đa lớp

**Lớp SAST**: CodeQL, Semgrep, Bandit/Brakeman/Gosec

**Mô hình hóa Mối đe dọa được Tăng cường bởi AI**:

```python
security_analysis_prompt = """
Analyze authentication code for vulnerabilities:
{code_snippet}

Check for:
1. Authentication bypass, broken access control (IDOR)
2. JWT token validation flaws
3. Session fixation/hijacking, timing attacks
4. Missing rate limiting, insecure password storage
5. Credential stuffing protection gaps

Provide: CWE identifier, CVSS score, exploit scenario, remediation code
"""

findings = claude.analyze(security_analysis_prompt, temperature=0.1)
```

**Quét Bí mật**:

```bash
trufflehog git file://. --json | \
  jq '.[] | select(.Verified == true) | {
    secret_type: .DetectorName,
    file: .SourceMetadata.Data.Filename,
    severity: "CRITICAL"
  }'
```

### OWASP Top 10 (2025)

1. **A01 - Kiểm soát Truy cập Hỏng**: Thiếu ủy quyền, IDOR
2. **A02 - Lỗi Mật mã**: Băm yếu, RNG không an toàn
3. **A03 - Injection**: SQL, NoSQL, command injection qua phân tích taint
4. **A04 - Thiết kế Không an toàn**: Thiếu mô hình hóa mối đe dọa
5. **A05 - Cấu hình sai Bảo mật**: Thông tin xác thực mặc định
6. **A06 - Thành phần Dễ bị tổn thương**: Snyk/Dependabot cho CVEs
7. **A07 - Lỗi Xác thực**: Quản lý phiên yếu
8. **A08 - Lỗi Tính toàn vẹn Dữ liệu**: JWT không được ký
9. **A09 - Lỗi Ghi nhật ký**: Thiếu nhật ký kiểm toán
10. **A10 - SSRF**: URL do người dùng kiểm soát không được xác thực

## Đánh giá Hiệu suất

### Hồ sơ Hiệu suất

```javascript
class PerformanceReviewAgent {
  async analyzePRPerformance(prNumber) {
    const baseline = await this.loadBaselineMetrics("main");
    const prBranch = await this.runBenchmarks(`pr-${prNumber}`);

    const regressions = this.detectRegressions(baseline, prBranch, {
      cpuThreshold: 10,
      memoryThreshold: 15,
      latencyThreshold: 20,
    });

    if (regressions.length > 0) {
      await this.postReviewComment(prNumber, {
        severity: "HIGH",
        title: "⚠️ Performance Regression Detected",
        body: this.formatRegressionReport(regressions),
        suggestions: await this.aiGenerateOptimizations(regressions),
      });
    }
  }
}
```

### Cờ Đỏ về Khả năng Mở rộng

- **Truy vấn N+1**, **Thiếu Chỉ mục**, **Cuộc gọi Bên ngoài Đồng bộ**
- **Trạng thái Trong Bộ nhớ**, **Bộ sưu tập Không giới hạn**, **Thiếu Phân trang**
- **Không Gộp Kết nối**, **Không Giới hạn Tốc độ**

```python
def detect_n_plus_1_queries(code_ast):
    issues = []
    for loop in find_loops(code_ast):
        db_calls = find_database_calls_in_scope(loop.body)
        if len(db_calls) > 0:
            issues.append({
                'severity': 'HIGH',
                'line': loop.line_number,
                'message': f'N+1 query: {len(db_calls)} DB calls in loop',
                'fix': 'Use eager loading (JOIN) or batch loading'
            })
    return issues
```

## Tạo Bình luận Đánh giá

### Định dạng Có cấu trúc

```typescript
interface ReviewComment {
  path: string;
  line: number;
  severity: "CRITICAL" | "HIGH" | "MEDIUM" | "LOW" | "INFO";
  category: "Security" | "Performance" | "Bug" | "Maintainability";
  title: string;
  description: string;
  codeExample?: string;
  references?: string[];
  autoFixable: boolean;
  cwe?: string;
  cvss?: number;
  effort: "trivial" | "easy" | "medium" | "hard";
}

const comment: ReviewComment = {
  path: "src/auth/login.ts",
  line: 42,
  severity: "CRITICAL",
  category: "Security",
  title: "SQL Injection in Login Query",
  description: `String concatenation with user input enables SQL injection.
**Attack Vector:** Input 'admin' OR '1'='1' bypasses authentication.
**Impact:** Complete auth bypass, unauthorized access.`,
  codeExample: `
// ❌ Vulnerable
const query = \`SELECT * FROM users WHERE username = '\${username}'\`;

// ✅ Secure
const query = 'SELECT * FROM users WHERE username = ?';
const result = await db.execute(query, [username]);
  `,
  references: ["https://cwe.mitre.org/data/definitions/89.html"],
  autoFixable: false,
  cwe: "CWE-89",
  cvss: 9.8,
  effort: "easy",
};
```

## Tích hợp CI/CD

### GitHub Actions

```yaml
name: AI Code Review
on:
  pull_request:
    types: [opened, synchronize, reopened]

jobs:
  ai-review:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Static Analysis
        run: |
          sonar-scanner -Dsonar.pullrequest.key=${{ github.event.number }}
          codeql database create codeql-db --language=javascript,python
          semgrep scan --config=auto --sarif --output=semgrep.sarif

      - name: AI-Enhanced Review (GPT-5)
        env:
          OPENAI_API_KEY: ${{ secrets.OPENAI_API_KEY }}
        run: |
          python scripts/ai_review.py \
            --pr-number ${{ github.event.number }} \
            --model gpt-4o \
            --static-analysis-results codeql.sarif,semgrep.sarif

      - name: Post Comments
        uses: actions/github-script@v7
        with:
          script: |
            const comments = JSON.parse(fs.readFileSync('review-comments.json'));
            for (const comment of comments) {
              await github.rest.pulls.createReviewComment({
                owner: context.repo.owner,
                repo: context.repo.repo,
                pull_number: context.issue.number,
                body: comment.body, path: comment.path, line: comment.line
              });
            }

      - name: Quality Gate
        run: |
          CRITICAL=$(jq '[.[] | select(.severity == "CRITICAL")] | length' review-comments.json)
          if [ $CRITICAL -gt 0 ]; then
            echo "❌ Found $CRITICAL critical issues"
            exit 1
          fi
```

## Ví dụ Hoàn chỉnh: Tự động hóa Đánh giá AI

````python
#!/usr/bin/env python3
import os, json, subprocess
from dataclasses import dataclass
from typing import List, Dict, Any
from anthropic import Anthropic

@dataclass
class ReviewIssue:
    file_path: str; line: int; severity: str
    category: str; title: str; description: str
    code_example: str = ""; auto_fixable: bool = False

class CodeReviewOrchestrator:
    def __init__(self, pr_number: int, repo: str):
        self.pr_number = pr_number; self.repo = repo
        self.github_token = os.environ['GITHUB_TOKEN']
        self.anthropic_client = Anthropic(api_key=os.environ['ANTHROPIC_API_KEY'])
        self.issues: List[ReviewIssue] = []

    def run_static_analysis(self) -> Dict[str, Any]:
        results = {}

        # SonarQube
        subprocess.run(['sonar-scanner', f'-Dsonar.projectKey={self.repo}'], check=True)

        # Semgrep
        semgrep_output = subprocess.check_output(['semgrep', 'scan', '--config=auto', '--json'])
        results['semgrep'] = json.loads(semgrep_output)

        return results

    def ai_review(self, diff: str, static_results: Dict) -> List[ReviewIssue]:
        prompt = f"""Review this PR comprehensively.

**Diff:** {diff[:15000]}
**Static Analysis:** {json.dumps(static_results, indent=2)[:5000]}

Focus: Security, Performance, Architecture, Bug risks, Maintainability

Return JSON array:
[{{
  "file_path": "src/auth.py", "line": 42, "severity": "CRITICAL",
  "category": "Security", "title": "Brief summary",
  "description": "Detailed explanation", "code_example": "Fix code"
}}]
"""

        response = self.anthropic_client.messages.create(
            model="claude-3-5-sonnet-20241022",
            max_tokens=8000, temperature=0.2,
            messages=[{"role": "user", "content": prompt}]
        )

        content = response.content[0].text
        if '```json' in content:
            content = content.split('```json')[1].split('```')[0]

        return [ReviewIssue(**issue) for issue in json.loads(content.strip())]

    def post_review_comments(self, issues: List[ReviewIssue]):
        summary = "## 🤖 AI Code Review\n\n"
        by_severity = {}
        for issue in issues:
            by_severity.setdefault(issue.severity, []).append(issue)

        for severity in ['CRITICAL', 'HIGH', 'MEDIUM', 'LOW']:
            count = len(by_severity.get(severity, []))
            if count > 0:
                summary += f"- **{severity}**: {count}\n"

        critical_count = len(by_severity.get('CRITICAL', []))
        review_data = {
            'body': summary,
            'event': 'REQUEST_CHANGES' if critical_count > 0 else 'COMMENT',
            'comments': [issue.to_github_comment() for issue in issues]
        }

        # Post to GitHub API
        print(f"✅ Posted review with {len(issues)} comments")

if __name__ == '__main__':
    import argparse
    parser = argparse.ArgumentParser()
    parser.add_argument('--pr-number', type=int, required=True)
    parser.add_argument('--repo', required=True)
    args = parser.parse_args()

    reviewer = CodeReviewOrchestrator(args.pr_number, args.repo)
    static_results = reviewer.run_static_analysis()
    diff = reviewer.get_pr_diff()
    ai_issues = reviewer.ai_review(diff, static_results)
    reviewer.post_review_comments(ai_issues)
````

## Tóm tắt

Đánh giá mã AI toàn diện kết hợp:

1. Phân tích tĩnh đa công cụ (SonarQube, CodeQL, Semgrep)
2. Các LLM hiện đại nhất (GPT-5, Claude 4.5 Sonnet)
3. Tích hợp CI/CD liền mạch (GitHub Actions, GitLab, Azure DevOps)
4. Hỗ trợ 30+ ngôn ngữ với các linter cụ thể theo ngôn ngữ
5. Các bình luận đánh giá có thể hành động với mức độ nghiêm trọng và ví dụ sửa chữa
6. Theo dõi số liệu DORA để biết hiệu quả đánh giá
7. Các cổng chất lượng ngăn chặn mã chất lượng thấp
8. Tạo kiểm thử tự động qua Qodo/CodiumAI

Sử dụng công cụ này để chuyển đổi đánh giá mã từ quy trình thủ công sang đảm bảo chất lượng được hỗ trợ bởi AI tự động, bắt lỗi sớm với phản hồi tức thì.
