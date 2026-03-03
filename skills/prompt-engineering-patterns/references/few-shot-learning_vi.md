# Few-Shot Learning Guide

## Tổng quan (Overview)

Few-shot learning cho phép các LLM thực hiện function của các task bằng cách cung cấp một số lượng nhỏ các ví dụ (hiển nhiên là từ 1-10) ngay bên trong prompt. Kỹ thuật này đạt hiệu quả cực kỳ cao đối với các task yêu cầu định dạng (format), phong cách (style) cụ thể hoặc các knowledge theo miền domain.

## Các Chiến lược Lựa chọn Ví dụ (Example Selection Strategies)

### 1. Semantic Similarity

Lựa chọn các ví dụ có độ tương đồng lớn nhất với input query đầu vào bằng cách sử dụng retrieval dựa trên embedding.

```python
from sentence_transformers import SentenceTransformer
import numpy as np

class SemanticExampleSelector:
    def __init__(self, examples, model_name='all-MiniLM-L6-v2'):
        self.model = SentenceTransformer(model_name)
        self.examples = examples
        self.example_embeddings = self.model.encode([ex['input'] for ex in examples])

    def select(self, query, k=3):
        query_embedding = self.model.encode([query])
        similarities = np.dot(self.example_embeddings, query_embedding.T).flatten()
        top_indices = np.argsort(similarities)[-k:][::-1]
        return [self.examples[i] for i in top_indices]
```

**Tốt nhất cho (Best For)**: Hỏi đáp (Question answering), text classification, các extraction tasks

### 2. Diversity Sampling

Tối đa hóa độ bao phủ của các pattern khác nhau và các edge cases.

```python
from sklearn.cluster import KMeans

class DiversityExampleSelector:
    def __init__(self, examples, model_name='all-MiniLM-L6-v2'):
        self.model = SentenceTransformer(model_name)
        self.examples = examples
        self.embeddings = self.model.encode([ex['input'] for ex in examples])

    def select(self, k=5):
        # Use k-means to find diverse cluster centers
        kmeans = KMeans(n_clusters=k, random_state=42)
        kmeans.fit(self.embeddings)

        # Select example closest to each cluster center
        diverse_examples = []
        for center in kmeans.cluster_centers_:
            distances = np.linalg.norm(self.embeddings - center, axis=1)
            closest_idx = np.argmin(distances)
            diverse_examples.append(self.examples[closest_idx])

        return diverse_examples
```

**Tốt nhất cho (Best For)**: Việc minh họa tính đa dạng của task (task variability), xử lý các edge case

### 3. Difficulty-Based Selection

Gia tăng dần dần mức độ Complexity của các ví dụ để tạo nền tảng (scaffold) cho quá trình learning.

```python
class ProgressiveExampleSelector:
    def __init__(self, examples):
        # Examples should have 'difficulty' scores (0-1)
        self.examples = sorted(examples, key=lambda x: x['difficulty'])

    def select(self, k=3):
        # Select examples with linearly increasing difficulty
        step = len(self.examples) // k
        return [self.examples[i * step] for i in range(k)]
```

**Tốt nhất cho (Best For)**: Các tác vụ suy luận phức tạp (Complex reasoning tasks), code generation

### 4. Error-Based Selection

Bao gồm các ví dụ giải quyết những failure modes phổ biến thường gặp.

```python
class ErrorGuidedSelector:
    def __init__(self, examples, error_patterns):
        self.examples = examples
        self.error_patterns = error_patterns  # Common mistakes to avoid

    def select(self, query, k=3):
        # Select examples demonstrating correct handling of error patterns
        selected = []
        for pattern in self.error_patterns[:k]:
            matching = [ex for ex in self.examples if pattern in ex['demonstrates']]
            if matching:
                selected.append(matching[0])
        return selected
```

**Tốt nhất cho (Best For)**: Các tác vụ với các pattern kiến nghị lỗi đã biết, safety-critical applications (các ứng dụng cực kỳ quan trọng đòi hỏi độ an toàn cao)

## Các Best Practice Xây dựng Ví dụ (Example Construction Best Practices)

### Format Consistency (Tính Nhất quán Định dạng)

Tất cả các ví dụ nên tuân theo một định dạng hoàn toàn đồng nhất:

```python
# Good: Consistent format
examples = [
    {
        "input": "What is the capital of France?",
        "output": "Paris"
    },
    {
        "input": "What is the capital of Germany?",
        "output": "Berlin"
    }
]

# Bad: Inconsistent format
examples = [
    "Q: What is the capital of France? A: Paris",
    {"question": "What is the capital of Germany?", "answer": "Berlin"}
]
```

### Input-Output Alignment

Đảm bảo các ví dụ minh họa chính xác task mà bạn muốn model thực hiện:

```python
# Good: Clear input-output relationship
example = {
    "input": "Sentiment: The movie was terrible and boring.",
    "output": "Negative"
}

# Bad: Ambiguous relationship
example = {
    "input": "The movie was terrible and boring.",
    "output": "This review expresses negative sentiment toward the film."
}
```

### Complexity Balance (Cân bằng độ phức tạp)

Bao gồm các ví dụ trải dài trên toàn bộ phạm vi mức độ khó dự kiến:

```python
examples = [
    # Simple case
    {"input": "2 + 2", "output": "4"},

    # Moderate case
    {"input": "15 * 3 + 8", "output": "53"},

    # Complex case
    {"input": "(12 + 8) * 3 - 15 / 5", "output": "57"}
]
```

## Quản lý Context Window (Context Window Management)

### Token Budget Allocation (Phân bổ Ngân sách Token)

Tỷ lệ phân phối điển hình cho một context window giới hạn 4K:

```
System Prompt:        500 tokens  (12%)
Few-Shot Examples:   1500 tokens  (38%)
User Input:           500 tokens  (12%)
Response:            1500 tokens  (38%)
```

### Dynamic Example Truncation

```python
class TokenAwareSelector:
    def __init__(self, examples, tokenizer, max_tokens=1500):
        self.examples = examples
        self.tokenizer = tokenizer
        self.max_tokens = max_tokens

    def select(self, query, k=5):
        selected = []
        total_tokens = 0

        # Start with most relevant examples
        candidates = self.rank_by_relevance(query)

        for example in candidates[:k]:
            example_tokens = len(self.tokenizer.encode(
                f"Input: {example['input']}\nOutput: {example['output']}\n\n"
            ))

            if total_tokens + example_tokens <= self.max_tokens:
                selected.append(example)
                total_tokens += example_tokens
            else:
                break

        return selected
```

## Xử lý các tình huống biên (Edge Case Handling)

### Include Boundary Examples (Bao hàm các Ví dụ Ranh giới tính tới)

```python
edge_case_examples = [
    # Empty input
    {"input": "", "output": "Please provide input text."},

    # Very long input (truncated in example)
    {"input": "..." + "word " * 1000, "output": "Input exceeds maximum length."},

    # Ambiguous input
    {"input": "bank", "output": "Ambiguous: Could refer to financial institution or river bank."},

    # Invalid input
    {"input": "!@#$%", "output": "Invalid input format. Please provide valid text."}
]
```

## Các Few-Shot Prompt Templates

### Classification Template

```python
def build_classification_prompt(examples, query, labels):
    prompt = f"Classify the text into one of these categories: {', '.join(labels)}\n\n"

    for ex in examples:
        prompt += f"Text: {ex['input']}\nCategory: {ex['output']}\n\n"

    prompt += f"Text: {query}\nCategory:"
    return prompt
```

### Extraction Template

```python
def build_extraction_prompt(examples, query):
    prompt = "Extract structured information from the text.\n\n"

    for ex in examples:
        prompt += f"Text: {ex['input']}\nExtracted: {json.dumps(ex['output'])}\n\n"

    prompt += f"Text: {query}\nExtracted:"
    return prompt
```

### Transformation Template

```python
def build_transformation_prompt(examples, query):
    prompt = "Transform the input according to the pattern shown in examples.\n\n"

    for ex in examples:
        prompt += f"Input: {ex['input']}\nOutput: {ex['output']}\n\n"

    prompt += f"Input: {query}\nOutput:"
    return prompt
```

## Đánh giá và Tối ưu hóa (Evaluation and Optimization)

### Example Quality Metrics

```python
def evaluate_example_quality(example, validation_set):
    metrics = {
        'clarity': rate_clarity(example),  # 0-1 score
        'representativeness': calculate_similarity_to_validation(example, validation_set),
        'difficulty': estimate_difficulty(example),
        'uniqueness': calculate_uniqueness(example, other_examples)
    }
    return metrics
```

### A/B Testing Example Sets

```python
class ExampleSetTester:
    def __init__(self, llm_client):
        self.client = llm_client

    def compare_example_sets(self, set_a, set_b, test_queries):
        results_a = self.evaluate_set(set_a, test_queries)
        results_b = self.evaluate_set(set_b, test_queries)

        return {
            'set_a_accuracy': results_a['accuracy'],
            'set_b_accuracy': results_b['accuracy'],
            'winner': 'A' if results_a['accuracy'] > results_b['accuracy'] else 'B',
            'improvement': abs(results_a['accuracy'] - results_b['accuracy'])
        }

    def evaluate_set(self, examples, test_queries):
        correct = 0
        for query in test_queries:
            prompt = build_prompt(examples, query['input'])
            response = self.client.complete(prompt)
            if response == query['expected_output']:
                correct += 1
        return {'accuracy': correct / len(test_queries)}
```

## Các kỹ thuật Meta-Learning Nâng cao (Advanced Techniques)

### Meta-Learning (Learning to Select)

Teach một small model để dự đoán những ví dụ nào sẽ mang lại hiệu quả cao nhất:

```python
from sklearn.ensemble import RandomForestClassifier

class LearnedExampleSelector:
    def __init__(self):
        self.selector_model = RandomForestClassifier()

    def train(self, training_data):
        # training_data: list of (query, example, success) tuples
        features = []
        labels = []

        for query, example, success in training_data:
            features.append(self.extract_features(query, example))
            labels.append(1 if success else 0)

        self.selector_model.fit(features, labels)

    def extract_features(self, query, example):
        return [
            semantic_similarity(query, example['input']),
            len(example['input']),
            len(example['output']),
            keyword_overlap(query, example['input'])
        ]

    def select(self, query, candidates, k=3):
        scores = []
        for example in candidates:
            features = self.extract_features(query, example)
            score = self.selector_model.predict_proba([features])[0][1]
            scores.append((score, example))

        return [ex for _, ex in sorted(scores, reverse=True)[:k]]
```

### Adaptive Example Count

Tinh chỉnh linh hoạt cấu hình số lượng các ví dụ tuỳ chỉnh (number of examples) dưa trên độ khó của bản thân task:

```python
class AdaptiveExampleSelector:
    def __init__(self, examples):
        self.examples = examples

    def select(self, query, max_examples=5):
        # Start with 1 example
        for k in range(1, max_examples + 1):
            selected = self.get_top_k(query, k)

            # Quick confidence check (could use a lightweight model)
            if self.estimated_confidence(query, selected) > 0.9:
                return selected

        return selected  # Return max_examples if never confident enough
```

## Những sai lầm thường gặp (Common Mistakes)

1. **Quá nhiều ví dụ (Too Many Examples)**: Nhiều hơn không phải lúc nào cũng tốt hơn; có thể làm loãng sự tập trung
2. **Ví dụ không liên quan (Irrelevant Examples)**: Các ví dụ nên khớp với target task một cách sát sao nhất
3. **Định dạng không đồng nhất (Inconsistent Formatting)**: Làm model bối rối về việc nên chọn định dạng output format nào
4. **Overfitting với các ví dụ**: Model sao chép các mô hình của ví dụ một cách quá máy móc theo nghĩa đen (too literally)
5. **Phớt lờ các Token Limits (Ignoring Token Limits)**: Cạn kiệt sức chứa không gian dành cho actual input/output đích thực

## Tài nguyên (Resources)

- Các kho Example dataset repositories
- Pre-built example selectors cho những tác vụ nhiệm vụ thường nhật (common tasks)
- Các mảng Evaluation frameworks cho phương thức kiểm soát độ few-shot performance
- Token counting utilities cho các luồng different models khác nhau
