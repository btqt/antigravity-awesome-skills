# Sổ tay triển khai các mẫu Apache Airflow DAG

Tệp này chứa các mẫu chi tiết, danh sách kiểm tra và các mẫu mã được tham chiếu trong kỹ năng này.

## Các khái niệm cốt lõi

### 1. Nguyên tắc thiết kế DAG

| Nguyên tắc                         | Mô tả                                                             |
| ---------------------------------- | ----------------------------------------------------------------- |
| **Tính lặp lại (Idempotent)**      | Chạy hai lần đều cho ra cùng một kết quả                          |
| **Tính nguyên tử (Atomic)**        | Các tác vụ thành công hoặc thất bại hoàn toàn                     |
| **Tính lũy tiến (Incremental)**    | Chỉ xử lý dữ liệu mới hoặc đã thay đổi                            |
| **Khả năng quan sát (Observable)** | Có nhật ký (logs), chỉ số (metrics), cảnh báo (alerts) ở mọi bước |

### 2. Sự phụ thuộc của tác vụ (Task Dependencies)

```python
# Tuyến tính (Linear)
task1 >> task2 >> task3

# Tỏa ra (Fan-out)
task1 >> [task2, task3, task4]

# Gom lại (Fan-in)
[task1, task2, task3] >> task4

# Phức tạp
task1 >> task2 >> task4
task1 >> task3 >> task4
```

## Bắt đầu nhanh

```python
# dags/example_dag.py
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.empty import EmptyOperator

default_args = {
    'owner': 'data-team',
    'depends_on_past': False,
    'email_on_failure': True,
    'email_on_retry': False,
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
    'retry_exponential_backoff': True,
    'max_retry_delay': timedelta(hours=1),
}

with DAG(
    dag_id='example_etl',
    default_args=default_args,
    description='Ví dụ về đường dẫn ETL',
    schedule='0 6 * * *',  # Hàng ngày lúc 6 giờ sáng
    start_date=datetime(2024, 1, 1),
    catchup=False,
    tags=['etl', 'example'],
    max_active_runs=1,
) as dag:

    start = EmptyOperator(task_id='start')

    def extract_data(**context):
        execution_date = context['ds']
        # Logic trích xuất dữ liệu tại đây
        return {'records': 1000}

    extract = PythonOperator(
        task_id='extract',
        python_callable=extract_data,
    )

    end = EmptyOperator(task_id='end')

    start >> extract >> end
```

## Các mẫu (Patterns)

### Mẫu 1: TaskFlow API (Airflow 2.0+)

```python
# dags/taskflow_example.py
from datetime import datetime
from airflow.decorators import dag, task
from airflow.models import Variable

@dag(
    dag_id='taskflow_etl',
    schedule='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
    tags=['etl', 'taskflow'],
)
def taskflow_etl():
    """Đường dẫn ETL sử dụng TaskFlow API"""

    @task()
    def extract(source: str) -> dict:
        """Trích xuất dữ liệu từ nguồn"""
        import pandas as pd

        df = pd.read_csv(f's3://bucket/{source}/{{ ds }}.csv')
        return {'data': df.to_dict(), 'rows': len(df)}

    @task()
    def transform(extracted: dict) -> dict:
        """Biến đổi dữ liệu đã trích xuất"""
        import pandas as pd

        df = pd.DataFrame(extracted['data'])
        df['processed_at'] = datetime.now()
        df = df.dropna()
        return {'data': df.to_dict(), 'rows': len(df)}

    @task()
    def load(transformed: dict, target: str):
        """Tải dữ liệu vào đích"""
        import pandas as pd

        df = pd.DataFrame(transformed['data'])
        df.to_parquet(f's3://bucket/{target}/{{ ds }}.parquet')
        return transformed['rows']

    @task()
    def notify(rows_loaded: int):
        """Gửi thông báo"""
        print(f'Đã tải {rows_loaded} dòng')

    # Định nghĩa sự phụ thuộc với việc truyền XCom
    extracted = extract(source='raw_data')
    transformed = transform(extracted)
    loaded = load(transformed, target='processed_data')
    notify(loaded)

# Khởi tạo DAG
taskflow_etl()
```

### Mẫu 2: Tạo DAG động (Dynamic DAG Generation)

```python
# dags/dynamic_dag_factory.py
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.models import Variable
import json

# Cấu hình cho nhiều đường dẫn tương tự nhau
PIPELINE_CONFIGS = [
    {'name': 'customers', 'schedule': '@daily', 'source': 's3://raw/customers'},
    {'name': 'orders', 'schedule': '@hourly', 'source': 's3://raw/orders'},
    {'name': 'products', 'schedule': '@weekly', 'source': 's3://raw/products'},
]

def create_dag(config: dict) -> DAG:
    """Hàm factory để tạo các DAG từ cấu hình"""

    dag_id = f"etl_{config['name']}"

    default_args = {
        'owner': 'data-team',
        'retries': 3,
        'retry_delay': timedelta(minutes=5),
    }

    dag = DAG(
        dag_id=dag_id,
        default_args=default_args,
        schedule=config['schedule'],
        start_date=datetime(2024, 1, 1),
        catchup=False,
        tags=['etl', 'dynamic', config['name']],
    )

    with dag:
        def extract_fn(source, **context):
            print(f"Đang trích xuất từ {source} cho ngày {context['ds']}")

        def transform_fn(**context):
            print(f"Đang biến đổi dữ liệu cho ngày {context['ds']}")

        def load_fn(table_name, **context):
            print(f"Đang tải vào {table_name} cho ngày {context['ds']}")

        extract = PythonOperator(
            task_id='extract',
            python_callable=extract_fn,
            op_kwargs={'source': config['source']},
        )

        transform = PythonOperator(
            task_id='transform',
            python_callable=transform_fn,
        )

        load = PythonOperator(
            task_id='load',
            python_callable=load_fn,
            op_kwargs={'table_name': config['name']},
        )

        extract >> transform >> load

    return dag

# Tạo các DAG
for config in PIPELINE_CONFIGS:
    globals()[f"dag_{config['name']}"] = create_dag(config)
```

### Mẫu 3: Phân nhánh và Logic điều kiện

```python
# dags/branching_example.py
from airflow.decorators import dag, task
from airflow.operators.python import BranchPythonOperator
from airflow.operators.empty import EmptyOperator
from airflow.utils.trigger_rule import TriggerRule

@dag(
    dag_id='branching_pipeline',
    schedule='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
)
def branching_pipeline():

    @task()
    def check_data_quality() -> dict:
        """Kiểm tra chất lượng dữ liệu và trả về các chỉ số"""
        quality_score = 0.95  # Giả lập
        return {'score': quality_score, 'rows': 10000}

    def choose_branch(**context) -> str:
        """Xác định nhánh nào sẽ được thực thi"""
        ti = context['ti']
        metrics = ti.xcom_pull(task_ids='check_data_quality')

        if metrics['score'] >= 0.9:
            return 'high_quality_path'
        elif metrics['score'] >= 0.7:
            return 'medium_quality_path'
        else:
            return 'low_quality_path'

    quality_check = check_data_quality()

    branch = BranchPythonOperator(
        task_id='branch',
        python_callable=choose_branch,
    )

    high_quality = EmptyOperator(task_id='high_quality_path')
    medium_quality = EmptyOperator(task_id='medium_quality_path')
    low_quality = EmptyOperator(task_id='low_quality_path')

    # Điểm hội tụ - chạy sau khi bất kỳ nhánh nào hoàn thành
    join = EmptyOperator(
        task_id='join',
        trigger_rule=TriggerRule.NONE_FAILED_MIN_ONE_SUCCESS,
    )

    quality_check >> branch >> [high_quality, medium_quality, low_quality] >> join

branching_pipeline()
```

### Mẫu 4: Sensors và các Phụ thuộc bên ngoài

```python
# dags/sensor_patterns.py
from datetime import datetime, timedelta
from airflow import DAG
from airflow.sensors.filesystem import FileSensor
from airflow.providers.amazon.aws.sensors.s3 import S3KeySensor
from airflow.sensors.external_task import ExternalTaskSensor
from airflow.operators.python import PythonOperator

with DAG(
    dag_id='sensor_example',
    schedule='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
) as dag:

    # Đợi tệp trên S3
    wait_for_file = S3KeySensor(
        task_id='wait_for_s3_file',
        bucket_name='data-lake',
        bucket_key='raw/{{ ds }}/data.parquet',
        aws_conn_id='aws_default',
        timeout=60 * 60 * 2,  # 2 giờ
        poke_interval=60 * 5,  # Kiểm tra mỗi 5 phút
        mode='reschedule',  # Giải phóng slot worker trong khi đợi
    )

    # Đợi một DAG khác hoàn thành
    wait_for_upstream = ExternalTaskSensor(
        task_id='wait_for_upstream_dag',
        external_dag_id='upstream_etl',
        external_task_id='final_task',
        execution_date_fn=lambda dt: dt,  # Cùng ngày thực thi
        timeout=60 * 60 * 3,
        mode='reschedule',
    )

    # Sensor tùy chỉnh sử dụng decorator @task.sensor
    @task.sensor(poke_interval=60, timeout=3600, mode='reschedule')
    def wait_for_api() -> PokeReturnValue:
        """Sensor tùy chỉnh cho tính khả dụng của API"""
        import requests

        response = requests.get('https://api.example.com/health')
        is_done = response.status_code == 200

        return PokeReturnValue(is_done=is_done, xcom_value=response.json())

    api_ready = wait_for_api()

    def process_data(**context):
        api_result = context['ti'].xcom_pull(task_ids='wait_for_api')
        print(f"API đã trả về: {api_result}")

    process = PythonOperator(
        task_id='process',
        python_callable=process_data,
    )

    [wait_for_file, wait_for_upstream, api_ready] >> process
```

### Mẫu 5: Xử lý lỗi và Cảnh báo

```python
# dags/error_handling.py
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.utils.trigger_rule import TriggerRule
from airflow.models import Variable

def task_failure_callback(context):
    """Callback khi tác vụ thất bại"""
    task_instance = context['task_instance']
    exception = context.get('exception')

    # Gửi đến Slack/PagerDuty/v.v.
    message = f"""
    Tác vụ thất bại!
    DAG: {task_instance.dag_id}
    Tác vụ: {task_instance.task_id}
    Ngày thực thi: {context['ds']}
    Lỗi: {exception}
    URL nhật ký: {task_instance.log_url}
    """
    # send_slack_alert(message)
    print(message)

def dag_failure_callback(context):
    """Callback khi DAG thất bại"""
    # Tổng hợp các lỗi, gửi tóm tắt
    pass

with DAG(
    dag_id='error_handling_example',
    schedule='@daily',
    start_date=datetime(2024, 1, 1),
    catchup=False,
    on_failure_callback=dag_failure_callback,
    default_args={
        'on_failure_callback': task_failure_callback,
        'retries': 3,
        'retry_delay': timedelta(minutes=5),
    },
) as dag:

    def might_fail(**context):
        import random
        if random.random() < 0.3:
            raise ValueError("Lỗi ngẫu nhiên!")
        return "Thành công"

    risky_task = PythonOperator(
        task_id='risky_task',
        python_callable=might_fail,
    )

    def cleanup(**context):
        """Dọn dẹp chạy bất kể các tác vụ phía trước có thất bại hay không"""
        print("Đang dọn dẹp...")

    cleanup_task = PythonOperator(
        task_id='cleanup',
        python_callable=cleanup,
        trigger_rule=TriggerRule.ALL_DONE,  # Chạy ngay cả khi phía trước thất bại
    )

    def notify_success(**context):
        """Chỉ chạy nếu tất cả các tác vụ phía trước thành công"""
        print("Tất cả tác vụ đã thành công!")

    success_notification = PythonOperator(
        task_id='notify_success',
        python_callable=notify_success,
        trigger_rule=TriggerRule.ALL_SUCCESS,
    )

    risky_task >> [cleanup_task, success_notification]
```

### Mẫu 6: Kiểm thử DAG

```python
# tests/test_dags.py
import pytest
from datetime import datetime
from airflow.models import DagBag

@pytest.fixture
def dagbag():
    return DagBag(dag_folder='dags/', include_examples=False)

def test_dag_loaded(dagbag):
    """Kiểm tra tất cả các DAG được nạp mà không có lỗi"""
    assert len(dagbag.import_errors) == 0, f"Lỗi nạp DAG: {dagbag.import_errors}"

def test_dag_structure(dagbag):
    """Kiểm tra cấu trúc DAG cụ thể"""
    dag = dagbag.get_dag('example_etl')

    assert dag is not None
    assert len(dag.tasks) == 3
    assert dag.schedule_interval == '0 6 * * *'

def test_task_dependencies(dagbag):
    """Kiểm tra sự phụ thuộc của tác vụ là chính xác"""
    dag = dagbag.get_dag('example_etl')

    extract_task = dag.get_task('extract')
    assert 'start' in [t.task_id for t in extract_task.upstream_list]
    assert 'end' in [t.task_id for t in extract_task.downstream_list]

def test_dag_integrity(dagbag):
    """Kiểm tra DAG không bị lặp vòng và hợp lệ"""
    for dag_id, dag in dagbag.dags.items():
        assert dag.test_cycle() is None, f"Phát hiện vòng lặp trong {dag_id}"

# Kiểm tra logic của từng tác vụ riêng lẻ
def test_extract_function():
    """Unit test cho hàm extract"""
    from dags.example_dag import extract_data

    result = extract_data(ds='2024-01-01')
    assert 'records' in result
    assert isinstance(result['records'], int)
```

## Cấu trúc dự án

```
airflow/
├── dags/
│   ├── __init__.py
│   ├── common/
│   │   ├── __init__.py
│   │   ├── operators.py    # Các operator tùy chỉnh
│   │   ├── sensors.py      # Các sensor tùy chỉnh
│   │   └── callbacks.py    # Các callback cảnh báo
│   ├── etl/
│   │   ├── customers.py
│   │   └── orders.py
│   └── ml/
│       └── training.py
├── plugins/
│   └── custom_plugin.py
├── tests/
│   ├── __init__.py
│   ├── test_dags.py
│   └── test_operators.py
├── docker-compose.yml
└── requirements.txt
```

## Các thực hành tốt nhất

### Nên làm

- **Sử dụng TaskFlow API** - Mã sạch hơn, tự động xử lý XCom.
- **Thiết lập thời gian chờ (timeouts)** - Ngăn chặn các tác vụ "thây ma" (zombie tasks).
- **Sử dụng `mode='reschedule'`** - Đối với các sensor, để giải phóng worker.
- **Kiểm thử các DAG** - Sử dụng cả unit tests và integration tests.
- **Các tác vụ có tính lặp lại (Idempotent)** - Đảm bảo an toàn khi thử lại.

### Không nên làm

- **Không sử dụng `depends_on_past=True`** - Dễ gây ra hiện tượng thắt nút cổ chai.
- **Không ghi cứng ngày tháng** - Sử dụng các macro như `{{ ds }}`.
- **Không sử dụng trạng thái toàn cục** - Các tác vụ nên là không trạng thái (stateless).
- **Không bỏ qua việc chạy bù (catchup) một cách mù quáng** - Hãy hiểu rõ các tác động của nó.
- **Không để các logic nặng nề trong tệp DAG** - Hãy import từ các module bên ngoài.

## Tài liệu tham khảo

- [Tài liệu Airflow](https://airflow.apache.org/docs/)
- [Hướng dẫn từ Astronomer](https://docs.astronomer.io/learn)
- [TaskFlow API](https://airflow.apache.org/docs/apache-airflow/stable/tutorial/taskflow.html)
