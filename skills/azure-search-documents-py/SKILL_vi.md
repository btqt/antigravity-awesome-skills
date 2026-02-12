---
name: azure-search-documents-py
description: |
  Azure AI Search SDK cho Python. Sử dụng cho vector search, hybrid search, semantic ranking, indexing, và skillsets.
  Triggers: "azure-search-documents", "SearchClient", "SearchIndexClient", "vector search", "hybrid search", "semantic search".
package: azure-search-documents
---

# Azure AI Search SDK cho Python

Dịch vụ tìm kiếm vector, full-text, và hybrid search với khả năng làm giàu dữ liệu bằng AI.

## Cài đặt

```bash
pip install azure-search-documents
```

## Biến Môi trường

```bash
AZURE_SEARCH_ENDPOINT=https://<service-name>.search.windows.net
AZURE_SEARCH_API_KEY=<your-api-key>
AZURE_SEARCH_INDEX_NAME=<your-index-name>
```

## Xác thực

### API Key

```python
from azure.search.documents import SearchClient
from azure.core.credentials import AzureKeyCredential
import os

client = SearchClient(
    endpoint=os.environ["AZURE_SEARCH_ENDPOINT"],
    index_name=os.environ["AZURE_SEARCH_INDEX_NAME"],
    credential=AzureKeyCredential(os.environ["AZURE_SEARCH_API_KEY"])
)
```

### Entra ID (Khuyên dùng)

```python
from azure.search.documents import SearchClient
from azure.identity import DefaultAzureCredential
import os

client = SearchClient(
    endpoint=os.environ["AZURE_SEARCH_ENDPOINT"],
    index_name=os.environ["AZURE_SEARCH_INDEX_NAME"],
    credential=DefaultAzureCredential()
)
```

## Các Loại Client

| Client                | Mục đích                           |
| --------------------- | ---------------------------------- |
| `SearchClient`        | Các hoạt động tìm kiếm và tài liệu |
| `SearchIndexClient`   | Quản lý index, synonym maps        |
| `SearchIndexerClient` | Indexers, nguồn dữ liệu, skillsets |

## Tạo Index với Trường Vector

```python
from azure.search.documents.indexes import SearchIndexClient
from azure.search.documents.indexes.models import (
    SearchIndex,
    SearchField,
    SearchFieldDataType,
    VectorSearch,
    HnswAlgorithmConfiguration,
    VectorSearchProfile,
    SearchableField,
    SimpleField
)

index_client = SearchIndexClient(endpoint, AzureKeyCredential(key))

fields = [
    SimpleField(name="id", type=SearchFieldDataType.String, key=True),
    SearchableField(name="title", type=SearchFieldDataType.String),
    SearchableField(name="content", type=SearchFieldDataType.String),
    SearchField(
        name="content_vector",
        type=SearchFieldDataType.Collection(SearchFieldDataType.Single),
        searchable=True,
        vector_search_dimensions=1536,
        vector_search_profile_name="my-vector-profile"
    )
]

vector_search = VectorSearch(
    algorithms=[
        HnswAlgorithmConfiguration(name="my-hnsw")
    ],
    profiles=[
        VectorSearchProfile(
            name="my-vector-profile",
            algorithm_configuration_name="my-hnsw"
        )
    ]
)

index = SearchIndex(
    name="my-index",
    fields=fields,
    vector_search=vector_search
)

index_client.create_or_update_index(index)
```

## Tải lên Tài liệu

```python
from azure.search.documents import SearchClient

client = SearchClient(endpoint, "my-index", AzureKeyCredential(key))

documents = [
    {
        "id": "1",
        "title": "Azure AI Search",
        "content": "Full-text and vector search service",
        "content_vector": [0.1, 0.2]  # Giả sử 1536 chiều
    }
]

result = client.upload_documents(documents)
print(f"Uploaded {len(result)} documents")
```

## Tìm kiếm Từ khóa (Keyword Search)

```python
results = client.search(
    search_text="azure search",
    select=["id", "title", "content"],
    top=10
)

for result in results:
    print(f"{result['title']}: {result['@search.score']}")
```

## Vector Search

```python
from azure.search.documents.models import VectorizedQuery

# Embedding truy vấn của bạn (ví dụ: 1536 chiều)
query_vector = get_embedding("semantic search capabilities")

vector_query = VectorizedQuery(
    vector=query_vector,
    k_nearest_neighbors=10,
    fields="content_vector"
)

results = client.search(
    vector_queries=[vector_query],
    select=["id", "title", "content"]
)

for result in results:
    print(f"{result['title']}: {result['@search.score']}")
```

## Hybrid Search (Vector + Keyword)

```python
from azure.search.documents.models import VectorizedQuery

vector_query = VectorizedQuery(
    vector=query_vector,
    k_nearest_neighbors=10,
    fields="content_vector"
)

results = client.search(
    search_text="azure search",
    vector_queries=[vector_query],
    select=["id", "title", "content"],
    top=10
)
```

## Xếp hạng Ngữ nghĩa (Semantic Ranking)

```python
from azure.search.documents.models import QueryType

results = client.search(
    search_text="what is azure search",
    query_type=QueryType.SEMANTIC,
    semantic_configuration_name="my-semantic-config",
    select=["id", "title", "content"],
    top=10
)

for result in results:
    print(f"{result['title']}")
    if result.get("@search.captions"):
        print(f"  Caption: {result['@search.captions'][0].text}")
```

## Bộ lọc (Filters)

```python
results = client.search(
    search_text="*",
    filter="category eq 'Technology' and rating gt 4",
    order_by=["rating desc"],
    select=["id", "title", "category", "rating"]
)
```

## Phân loại (Facets)

```python
results = client.search(
    search_text="*",
    facets=["category,count:10", "rating"],
    top=0  # Chỉ lấy facets, không lấy tài liệu
)

for facet_name, facet_values in results.get_facets().items():
    print(f"{facet_name}:")
    for facet in facet_values:
        print(f"  {facet['value']}: {facet['count']}")
```

## Autocomplete & Suggest

```python
# Autocomplete
results = client.autocomplete(
    search_text="sea",
    suggester_name="my-suggester",
    mode="twoTerms"
)

# Suggest
results = client.suggest(
    search_text="sea",
    suggester_name="my-suggester",
    select=["title"]
)
```

## Indexer với Skillset

```python
from azure.search.documents.indexes import SearchIndexerClient
from azure.search.documents.indexes.models import (
    SearchIndexer,
    SearchIndexerDataSourceConnection,
    SearchIndexerSkillset,
    EntityRecognitionSkill,
    InputFieldMappingEntry,
    OutputFieldMappingEntry
)

indexer_client = SearchIndexerClient(endpoint, AzureKeyCredential(key))

# Tạo nguồn dữ liệu
data_source = SearchIndexerDataSourceConnection(
    name="my-datasource",
    type="azureblob",
    connection_string=connection_string,
    container={"name": "documents"}
)
indexer_client.create_or_update_data_source_connection(data_source)

# Tạo skillset
skillset = SearchIndexerSkillset(
    name="my-skillset",
    skills=[
        EntityRecognitionSkill(
            inputs=[InputFieldMappingEntry(name="text", source="/document/content")],
            outputs=[OutputFieldMappingEntry(name="organizations", target_name="organizations")]
        )
    ]
)
indexer_client.create_or_update_skillset(skillset)

# Tạo indexer
indexer = SearchIndexer(
    name="my-indexer",
    data_source_name="my-datasource",
    target_index_name="my-index",
    skillset_name="my-skillset"
)
indexer_client.create_or_update_indexer(indexer)
```

## Thực hành Tốt nhất

1. **Sử dụng hybrid search** để có độ phù hợp tốt nhất, kết hợp vector và từ khóa
2. **Bật semantic ranking** cho các truy vấn ngôn ngữ tự nhiên
3. **Index theo lô (batch)** từ 100-1000 tài liệu để tăng hiệu quả
4. **Sử dụng bộ lọc** để thu hẹp kết quả trước khi xếp hạng
5. **Cấu hình kích thước vector** phù hợp với mô hình embedding của bạn
6. **Sử dụng thuật toán HNSW** cho tìm kiếm vector quy mô lớn
7. **Tạo suggesters** tại thời điểm tạo index (không thể thêm sau này)

## Các Mô hình Tìm kiếm Azure AI Bổ sung

### Các Mẫu Tạo Index

```python
from azure.search.documents.indexes import SearchIndexClient
from azure.search.documents.indexes.models import (
    SearchIndex, SearchField, VectorSearch, VectorSearchProfile,
    HnswAlgorithmConfiguration, AzureOpenAIVectorizer,
    AzureOpenAIVectorizerParameters, SemanticSearch,
    SemanticConfiguration, SemanticPrioritizedFields, SemanticField
)

index = SearchIndex(
    name=index_name,
    fields=[
        SearchField(name="id", type="Edm.String", key=True),
        SearchField(name="content", type="Edm.String", searchable=True),
        SearchField(name="embedding", type="Collection(Edm.Single)",
                   vector_search_dimensions=3072,
                   vector_search_profile_name="vector-profile"),
    ],
    vector_search=VectorSearch(
        profiles=[VectorSearchProfile(
            name="vector-profile",
            algorithm_configuration_name="hnsw-algo",
            vectorizer_name="openai-vectorizer"
        )],
        algorithms=[HnswAlgorithmConfiguration(name="hnsw-algo")],
        vectorizers=[AzureOpenAIVectorizer(
            vectorizer_name="openai-vectorizer",
            parameters=AzureOpenAIVectorizerParameters(
                resource_url=aoai_endpoint,
                deployment_name=embedding_deployment,
                model_name=embedding_model
            )
        )]
    ),
    semantic_search=SemanticSearch(
        default_configuration_name="semantic-config",
        configurations=[SemanticConfiguration(
            name="semantic-config",
            prioritized_fields=SemanticPrioritizedFields(
                content_fields=[SemanticField(field_name="content")]
            )
        )]
    )
)

index_client = SearchIndexClient(endpoint, credential)
index_client.create_or_update_index(index)
```

### Các Hoạt động Tài liệu

```python
from azure.search.documents import SearchIndexingBufferedSender

# Upload theo lô với tự động batching
with SearchIndexingBufferedSender(endpoint, index_name, credential) as sender:
    sender.upload_documents(documents)

# Các hoạt động trực tiếp qua SearchClient
search_client = SearchClient(endpoint, index_name, credential)
search_client.upload_documents(documents)      # Thêm mới
search_client.merge_documents(documents)       # Cập nhật hiện có
search_client.merge_or_upload_documents(documents)  # Upsert
search_client.delete_documents(documents)      # Xóa
```

### Các Mẫu Tìm kiếm Nâng cao

```python
# Tìm kiếm cơ bản
results = search_client.search(search_text="query")

# Vector search
from azure.search.documents.models import VectorizedQuery

results = search_client.search(
    search_text=None,
    vector_queries=[VectorizedQuery(
        vector=embedding,
        k_nearest_neighbors=5,
        fields="embedding"
    )]
)

# Hybrid search (vector + keyword)
results = search_client.search(
    search_text="query",
    vector_queries=[VectorizedQuery(vector=embedding, k_nearest_neighbors=5, fields="embedding")],
    query_type="semantic",
    semantic_configuration_name="semantic-config"
)

# Với bộ lọc
results = search_client.search(
    search_text="query",
    filter="category eq 'technology'",
    select=["id", "title", "content"],
    top=10
)
```

## Agentic Retrieval (Cơ sở Tri thức)

Cho Q&A được hỗ trợ bởi LLM với tổng hợp câu trả lời, xem [references/agentic-retrieval.md](references/agentic-retrieval.md).

Các khái niệm chính:

- **Nguồn Tri thức (Knowledge Source)**: Trỏ đến một chỉ mục tìm kiếm
- **Cơ sở Tri thức (Knowledge Base)**: Bao bọc các nguồn tri thức + LLM để lập kế hoạch truy vấn và tổng hợp
- **Chế độ đầu ra**: `EXTRACTIVE_DATA` (raw chunks) hoặc `ANSWER_SYNTHESIS` (LLM-generated answers)

## Mẫu Bất đồng bộ (Async Pattern)

```python
from azure.search.documents.aio import SearchClient

async with SearchClient(endpoint, index_name, credential) as client:
    results = await client.search(search_text="query")
    async for result in results:
        print(result["title"])
```

## Tham chiếu Các loại Trường

| EDM Type                 | Python      | Ghi chú                 |
| ------------------------ | ----------- | ----------------------- |
| `Edm.String`             | str         | Văn bản có thể tìm kiếm |
| `Edm.Int32`              | int         | Số nguyên               |
| `Edm.Int64`              | int         | Số nguyên dài           |
| `Edm.Double`             | float       | Số thực dấu phẩy động   |
| `Edm.Boolean`            | bool        | True/False              |
| `Edm.DateTimeOffset`     | datetime    | ISO 8601                |
| `Collection(Edm.Single)` | List[float] | Vector embeddings       |
| `Collection(Edm.String)` | List[str]   | Mảng chuỗi              |

## Xử lý Lỗi

```python
from azure.core.exceptions import (
    HttpResponseError,
    ResourceNotFoundError,
    ResourceExistsError
)

try:
    result = search_client.get_document(key="123")
except ResourceNotFoundError:
    print("Document not found")
except HttpResponseError as e:
    print(f"Search error: {e.message}")
```

## Liên kết Tham khảo

| Tài nguyên                                                       | URL                                                       |
| ---------------------------------------------------------------- | --------------------------------------------------------- |
| [references/vector-search.md](references/vector-search.md)       | Cấu hình HNSW, vectorization tích hợp, truy vấn đa vector |
| [references/semantic-ranking.md](references/semantic-ranking.md) | Cấu hình ngữ nghĩa, chú thích, câu trả lời, mẫu lai       |
| [scripts/setup_vector_index.py](scripts/setup_vector_index.py)   | CLI script để tạo vector-enabled search index             |
