---
name: azure-ai-ml-py
description: |
  Azure Machine Learning SDK v2 cho Python. Sử dụng cho các không gian làm việc (workspaces), công việc (jobs), mô hình, bộ dữ liệu, tính toán (compute) và đường ống (pipelines) ML.
  Kích hoạt: "azure-ai-ml", "MLClient", "không gian làm việc", "đăng ký mô hình", "công việc huấn luyện", "bộ dữ liệu".
package: azure-ai-ml
---

# Azure Machine Learning SDK v2 cho Python

Thư viện Client để quản lý các tài nguyên Azure ML: không gian làm việc, công việc, mô hình, dữ liệu và tính toán.

## Cài đặt

```bash
pip install azure-ai-ml
```

## Biến Môi trường

```bash
AZURE_SUBSCRIPTION_ID=<your-subscription-id>
AZURE_RESOURCE_GROUP=<your-resource-group>
AZURE_ML_WORKSPACE_NAME=<your-workspace-name>
```

## Xác thực

```python
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential

ml_client = MLClient(
    credential=DefaultAzureCredential(),
    subscription_id=os.environ["AZURE_SUBSCRIPTION_ID"],
    resource_group_name=os.environ["AZURE_RESOURCE_GROUP"],
    workspace_name=os.environ["AZURE_ML_WORKSPACE_NAME"]
)
```

### Từ Tệp Cấu hình

```python
from azure.ai.ml import MLClient
from azure.identity import DefaultAzureCredential

# Sử dụng config.json trong thư mục hiện tại hoặc cha
ml_client = MLClient.from_config(
    credential=DefaultAzureCredential()
)
```

## Quản lý Không gian làm việc (Workspace)

### Tạo Không gian làm việc

```python
from azure.ai.ml.entities import Workspace

ws = Workspace(
    name="my-workspace",
    location="eastus",
    display_name="My Workspace",
    description="ML workspace cho các thử nghiệm",
    tags={"purpose": "demo"}
)

ml_client.workspaces.begin_create(ws).result()
```

### Liệt kê Không gian làm việc

```python
for ws in ml_client.workspaces.list():
    print(f"{ws.name}: {ws.location}")
```

## Tài sản Dữ liệu (Data Assets)

### Đăng ký Dữ liệu

```python
from azure.ai.ml.entities import Data
from azure.ai.ml.constants import AssetTypes

# Đăng ký một tệp
my_data = Data(
    name="my-dataset",
    version="1",
    path="azureml://datastores/workspaceblobstore/paths/data/train.csv",
    type=AssetTypes.URI_FILE,
    description="Dữ liệu huấn luyện"
)

ml_client.data.create_or_update(my_data)
```

### Đăng ký Thư mục

```python
my_data = Data(
    name="my-folder-dataset",
    version="1",
    path="azureml://datastores/workspaceblobstore/paths/data/",
    type=AssetTypes.URI_FOLDER
)

ml_client.data.create_or_update(my_data)
```

## Đăng ký Mô hình (Model Registry)

### Đăng ký Mô hình

```python
from azure.ai.ml.entities import Model
from azure.ai.ml.constants import AssetTypes

model = Model(
    name="my-model",
    version="1",
    path="./model/",
    type=AssetTypes.CUSTOM_MODEL,
    description="Mô hình đã huấn luyện của tôi"
)

ml_client.models.create_or_update(model)
```

### Liệt kê Mô hình

```python
for model in ml_client.models.list(name="my-model"):
    print(f"{model.name} v{model.version}")
```

## Tính toán (Compute)

### Tạo Cụm Tính toán

```python
from azure.ai.ml.entities import AmlCompute

cluster = AmlCompute(
    name="cpu-cluster",
    type="amlcompute",
    size="Standard_DS3_v2",
    min_instances=0,
    max_instances=4,
    idle_time_before_scale_down=120
)

ml_client.compute.begin_create_or_update(cluster).result()
```

### Liệt kê Tính toán

```python
for compute in ml_client.compute.list():
    print(f"{compute.name}: {compute.type}")
```

## Công việc (Jobs)

### Công việc Lệnh (Command Job)

```python
from azure.ai.ml import command, Input

job = command(
    code="./src",
    command="python train.py --data ${{inputs.data}} --lr ${{inputs.learning_rate}}",
    inputs={
        "data": Input(type="uri_folder", path="azureml:my-dataset:1"),
        "learning_rate": 0.01
    },
    environment="AzureML-sklearn-1.0-ubuntu20.04-py38-cpu@latest",
    compute="cpu-cluster",
    display_name="training-job"
)

returned_job = ml_client.jobs.create_or_update(job)
print(f"Job URL: {returned_job.studio_url}")
```

### Giám sát Công việc

```python
ml_client.jobs.stream(returned_job.name)
```

## Đường ống (Pipelines)

```python
from azure.ai.ml import dsl, Input, Output
from azure.ai.ml.entities import Pipeline

@dsl.pipeline(
    compute="cpu-cluster",
    description="Đường ống huấn luyện"
)
def training_pipeline(data_input):
    prep_step = prep_component(data=data_input)
    train_step = train_component(
        data=prep_step.outputs.output_data,
        learning_rate=0.01
    )
    return {"model": train_step.outputs.model}

pipeline = training_pipeline(
    data_input=Input(type="uri_folder", path="azureml:my-dataset:1")
)

pipeline_job = ml_client.jobs.create_or_update(pipeline)
```

## Môi trường (Environments)

### Tạo Môi trường Tùy chỉnh

```python
from azure.ai.ml.entities import Environment

env = Environment(
    name="my-env",
    version="1",
    image="mcr.microsoft.com/azureml/openmpi4.1.0-ubuntu20.04",
    conda_file="./environment.yml"
)

ml_client.environments.create_or_update(env)
```

## Kho dữ liệu (Datastores)

### Liệt kê Kho dữ liệu

```python
for ds in ml_client.datastores.list():
    print(f"{ds.name}: {ds.type}")
```

### Lấy Kho dữ liệu Mặc định

```python
default_ds = ml_client.datastores.get_default()
print(f"Mặc định: {default_ds.name}")
```

## Các Hoạt động MLClient

| Thuộc tính     | Các Hoạt động                               |
| -------------- | ------------------------------------------- |
| `workspaces`   | create, get, list, delete                   |
| `jobs`         | create_or_update, get, list, stream, cancel |
| `models`       | create_or_update, get, list, archive        |
| `data`         | create_or_update, get, list                 |
| `compute`      | begin_create_or_update, get, list, delete   |
| `environments` | create_or_update, get, list                 |
| `datastores`   | create_or_update, get, list, get_default    |
| `components`   | create_or_update, get, list                 |

## Thực hành Tốt nhất

1. **Sử dụng phiên bản hóa** cho dữ liệu, mô hình và môi trường
2. **Cấu hình idle scale-down** để giảm chi phí tính toán
3. **Sử dụng môi trường** cho việc huấn luyện có thể tái tạo
4. **Stream log công việc** để giám sát tiến độ
5. **Đăng ký mô hình** sau các công việc huấn luyện thành công
6. **Sử dụng đường ống** cho các quy trình làm việc nhiều bước
7. **Gắn thẻ tài nguyên** để tổ chức và theo dõi chi phí
