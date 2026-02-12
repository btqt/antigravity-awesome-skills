---
name: azure-compute-batch-java
description: |
  Azure Batch SDK cho Java. Chạy các công việc batch song song quy mô lớn và HPC với pools, jobs, tasks, và compute nodes.
  Kích hoạt: "BatchClient java", "azure batch java", "batch pool java", "batch job java", "HPC java", "parallel computing java".
---

# Azure Batch SDK cho Java

Thư viện Client để chạy các công việc batch song song quy mô lớn và tính toán hiệu năng cao (HPC) trong Azure.

## Cài đặt

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-compute-batch</artifactId>
    <version>1.0.0-beta.5</version>
</dependency>
```

## Điều kiện Tiên quyết

- Tài khoản Azure Batch
- Pool được cấu hình với các compute nodes
- Azure subscription

## Biến Môi trường

```bash
AZURE_BATCH_ENDPOINT=https://<account>.<region>.batch.azure.com
AZURE_BATCH_ACCOUNT=<account-name>
AZURE_BATCH_ACCESS_KEY=<account-key>
```

## Tạo Client

### Với Microsoft Entra ID (Khuyên dùng)

```java
import com.azure.compute.batch.BatchClient;
import com.azure.compute.batch.BatchClientBuilder;
import com.azure.identity.DefaultAzureCredentialBuilder;

BatchClient batchClient = new BatchClientBuilder()
    .credential(new DefaultAzureCredentialBuilder().build())
    .endpoint(System.getenv("AZURE_BATCH_ENDPOINT"))
    .buildClient();
```

### Client Bất đồng bộ (Async Client)

```java
import com.azure.compute.batch.BatchAsyncClient;

BatchAsyncClient batchAsyncClient = new BatchClientBuilder()
    .credential(new DefaultAzureCredentialBuilder().build())
    .endpoint(System.getenv("AZURE_BATCH_ENDPOINT"))
    .buildAsyncClient();
```

### Với Shared Key Credentials

```java
import com.azure.core.credential.AzureNamedKeyCredential;

String accountName = System.getenv("AZURE_BATCH_ACCOUNT");
String accountKey = System.getenv("AZURE_BATCH_ACCESS_KEY");
AzureNamedKeyCredential sharedKeyCreds = new AzureNamedKeyCredential(accountName, accountKey);

BatchClient batchClient = new BatchClientBuilder()
    .credential(sharedKeyCreds)
    .endpoint(System.getenv("AZURE_BATCH_ENDPOINT"))
    .buildClient();
```

## Các Khái niệm Chính

| Khái niệm    | Mô tả                                |
| ------------ | ------------------------------------ |
| Pool         | Tập hợp các compute nodes chạy tasks |
| Job          | Nhóm logic của các tasks             |
| Task         | Đơn vị tính toán (lệnh/kịch bản)     |
| Node         | VM thực thi tasks                    |
| Job Schedule | Tạo job định kỳ                      |

## Các Thao tác Pool

### Tạo Pool

```java
import com.azure.compute.batch.models.*;

batchClient.createPool(new BatchPoolCreateParameters("myPoolId", "STANDARD_DC2s_V2")
    .setVirtualMachineConfiguration(
        new VirtualMachineConfiguration(
            new BatchVmImageReference()
                .setPublisher("Canonical")
                .setOffer("UbuntuServer")
                .setSku("22_04-lts")
                .setVersion("latest"),
            "batch.node.ubuntu 22.04"))
    .setTargetDedicatedNodes(2)
    .setTargetLowPriorityNodes(0), null);
```

### Lấy Pool

```java
BatchPool pool = batchClient.getPool("myPoolId");
System.out.println("Trạng thái Pool: " + pool.getState());
System.out.println("Dedicated nodes hiện tại: " + pool.getCurrentDedicatedNodes());
```

### Liệt kê Pools

```java
import com.azure.core.http.rest.PagedIterable;

PagedIterable<BatchPool> pools = batchClient.listPools();
for (BatchPool pool : pools) {
    System.out.println("Pool: " + pool.getId() + ", Trạng thái: " + pool.getState());
}
```

### Thay đổi Kích thước Pool (Resize)

```java
import com.azure.core.util.polling.SyncPoller;

BatchPoolResizeParameters resizeParams = new BatchPoolResizeParameters()
    .setTargetDedicatedNodes(4)
    .setTargetLowPriorityNodes(2);

SyncPoller<BatchPool, BatchPool> poller = batchClient.beginResizePool("myPoolId", resizeParams);
poller.waitForCompletion();
BatchPool resizedPool = poller.getFinalResult();
```

### Bật Tự động Thay đổi Kích thước (AutoScale)

```java
BatchPoolEnableAutoScaleParameters autoScaleParams = new BatchPoolEnableAutoScaleParameters()
    .setAutoScaleEvaluationInterval(Duration.ofMinutes(5))
    .setAutoScaleFormula("$TargetDedicatedNodes = min(10, $PendingTasks.GetSample(TimeInterval_Minute * 5));");

batchClient.enablePoolAutoScale("myPoolId", autoScaleParams);
```

### Xóa Pool

```java
SyncPoller<BatchPool, Void> deletePoller = batchClient.beginDeletePool("myPoolId");
deletePoller.waitForCompletion();
```

## Các Thao tác Job

### Tạo Job

```java
batchClient.createJob(
    new BatchJobCreateParameters("myJobId", new BatchPoolInfo().setPoolId("myPoolId"))
        .setPriority(100)
        .setConstraints(new BatchJobConstraints()
            .setMaxWallClockTime(Duration.ofHours(24))
            .setMaxTaskRetryCount(3)),
    null);
```

### Lấy Job

```java
BatchJob job = batchClient.getJob("myJobId", null, null);
System.out.println("Trạng thái Job: " + job.getState());
```

### Liệt kê Jobs

```java
PagedIterable<BatchJob> jobs = batchClient.listJobs(new BatchJobsListOptions());
for (BatchJob job : jobs) {
    System.out.println("Job: " + job.getId() + ", Trạng thái: " + job.getState());
}
```

### Lấy Số lượng Task

```java
BatchTaskCountsResult counts = batchClient.getJobTaskCounts("myJobId");
System.out.println("Hoạt động: " + counts.getTaskCounts().getActive());
System.out.println("Đang chạy: " + counts.getTaskCounts().getRunning());
System.out.println("Hoàn thành: " + counts.getTaskCounts().getCompleted());
```

### Chấm dứt Job

```java
BatchJobTerminateParameters terminateParams = new BatchJobTerminateParameters()
    .setTerminationReason("Chấm dứt thủ công");
BatchJobTerminateOptions options = new BatchJobTerminateOptions().setParameters(terminateParams);

SyncPoller<BatchJob, BatchJob> poller = batchClient.beginTerminateJob("myJobId", options, null);
poller.waitForCompletion();
```

### Xóa Job

```java
SyncPoller<BatchJob, Void> deletePoller = batchClient.beginDeleteJob("myJobId");
deletePoller.waitForCompletion();
```

## Các Thao tác Task

### Tạo Một Task

```java
BatchTaskCreateParameters task = new BatchTaskCreateParameters("task1", "echo 'Hello World'");
batchClient.createTask("myJobId", task);
```

### Tạo Task với Điều kiện Thoát

```java
batchClient.createTask("myJobId", new BatchTaskCreateParameters("task2", "cmd /c exit 3")
    .setExitConditions(new ExitConditions()
        .setExitCodeRanges(Arrays.asList(
            new ExitCodeRangeMapping(2, 4,
                new ExitOptions().setJobAction(BatchJobActionKind.TERMINATE)))))
    .setUserIdentity(new UserIdentity()
        .setAutoUser(new AutoUserSpecification()
            .setScope(AutoUserScope.TASK)
            .setElevationLevel(ElevationLevel.NON_ADMIN))),
    null);
```

### Tạo Bộ sưu tập Task (tối đa 100)

```java
List<BatchTaskCreateParameters> taskList = Arrays.asList(
    new BatchTaskCreateParameters("task1", "echo Task 1"),
    new BatchTaskCreateParameters("task2", "echo Task 2"),
    new BatchTaskCreateParameters("task3", "echo Task 3")
);
BatchTaskGroup taskGroup = new BatchTaskGroup(taskList);
BatchCreateTaskCollectionResult result = batchClient.createTaskCollection("myJobId", taskGroup);
```

### Tạo Nhiều Tasks (không giới hạn)

```java
List<BatchTaskCreateParameters> tasks = new ArrayList<>();
for (int i = 0; i < 1000; i++) {
    tasks.add(new BatchTaskCreateParameters("task" + i, "echo Task " + i));
}
batchClient.createTasks("myJobId", tasks);
```

### Lấy Task

```java
BatchTask task = batchClient.getTask("myJobId", "task1");
System.out.println("Trạng thái Task: " + task.getState());
System.out.println("Mã thoát: " + task.getExecutionInfo().getExitCode());
```

### Liệt kê Tasks

```java
PagedIterable<BatchTask> tasks = batchClient.listTasks("myJobId");
for (BatchTask task : tasks) {
    System.out.println("Task: " + task.getId() + ", Trạng thái: " + task.getState());
}
```

### Lấy Output của Task

```java
import com.azure.core.util.BinaryData;
import java.nio.charset.StandardCharsets;

BinaryData stdout = batchClient.getTaskFile("myJobId", "task1", "stdout.txt");
System.out.println(new String(stdout.toBytes(), StandardCharsets.UTF_8));
```

### Chấm dứt Task

```java
batchClient.terminateTask("myJobId", "task1", null, null);
```

## Các Thao tác Node

### Liệt kê Nodes

```java
PagedIterable<BatchNode> nodes = batchClient.listNodes("myPoolId", new BatchNodesListOptions());
for (BatchNode node : nodes) {
    System.out.println("Node: " + node.getId() + ", Trạng thái: " + node.getState());
}
```

### Khởi động lại Node

```java
SyncPoller<BatchNode, BatchNode> rebootPoller = batchClient.beginRebootNode("myPoolId", "nodeId");
rebootPoller.waitForCompletion();
```

### Lấy Cài đặt Đăng nhập Từ xa

```java
BatchNodeRemoteLoginSettings settings = batchClient.getNodeRemoteLoginSettings("myPoolId", "nodeId");
System.out.println("IP: " + settings.getRemoteLoginIpAddress());
System.out.println("Port: " + settings.getRemoteLoginPort());
```

## Các Thao tác Job Schedule

### Tạo Job Schedule

```java
batchClient.createJobSchedule(new BatchJobScheduleCreateParameters("myScheduleId",
    new BatchJobScheduleConfiguration()
        .setRecurrenceInterval(Duration.ofHours(6))
        .setDoNotRunUntil(OffsetDateTime.now().plusDays(1)),
    new BatchJobSpecification(new BatchPoolInfo().setPoolId("myPoolId"))
        .setPriority(50)),
    null);
```

### Lấy Job Schedule

```java
BatchJobSchedule schedule = batchClient.getJobSchedule("myScheduleId");
System.out.println("Trạng thái Schedule: " + schedule.getState());
```

## Xử lý Lỗi

```java
import com.azure.compute.batch.models.BatchErrorException;
import com.azure.compute.batch.models.BatchError;

try {
    batchClient.getPool("nonexistent-pool");
} catch (BatchErrorException e) {
    BatchError error = e.getValue();
    System.err.println("Mã lỗi: " + error.getCode());
    System.err.println("Thông điệp: " + error.getMessage().getValue());

    if ("PoolNotFound".equals(error.getCode())) {
        System.err.println("Pool được chỉ định không tồn tại.");
    }
}
```

## Thực hành Tốt nhất

1. **Sử dụng Entra ID** — Ưu tiên hơn shared key để xác thực
2. **Sử dụng management SDK cho pools** — `azure-resourcemanager-batch` hỗ trợ managed identities
3. **Tạo batch task** — Sử dụng `createTaskCollection` hoặc `createTasks` cho nhiều tasks
4. **Xử lý LRO đúng cách** — Các thao tác thay đổi kích thước pool, xóa là long-running
5. **Theo dõi số lượng task** — Sử dụng `getJobTaskCounts` để theo dõi tiến độ
6. **Đặt ràng buộc** — Cấu hình `maxWallClockTime` và `maxTaskRetryCount`
7. **Sử dụng low-priority nodes** — Tiết kiệm chi phí cho các khối lượng công việc chịu lỗi
8. **Bật tự động thay đổi kích thước (autoscale)** — Điều chỉnh kích thước pool động dựa trên khối lượng công việc
