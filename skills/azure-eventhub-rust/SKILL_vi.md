---
name: azure-eventhub-rust
description: |
  Azure Event Hubs SDK cho Rust. Sử dụng để gửi và nhận sự kiện, streaming dữ liệu.
  Triggers: "event hubs rust", "ProducerClient rust", "ConsumerClient rust", "send event rust", "streaming rust".
package: azure_messaging_eventhubs
---

# Azure Event Hubs SDK cho Rust

Thư viện Client cho Azure Event Hubs — nền tảng streaming dữ liệu lớn và dịch vụ nhập sự kiện.

## Cài đặt

```sh
cargo add azure_messaging_eventhubs azure_identity
```

## Biến Môi trường

```bash
EVENTHUBS_HOST=<namespace>.servicebus.windows.net
EVENTHUB_NAME=<eventhub-name>
```

## Các Khái niệm Chính

- **Namespace** — container cho Event Hubs
- **Event Hub** — luồng sự kiện được phân vùng để xử lý song song
- **Partition** — chuỗi sự kiện có thứ tự
- **Producer** — gửi sự kiện đến Event Hub
- **Consumer** — nhận sự kiện từ các partitions

## Producer Client

### Tạo Producer

```rust
use azure_identity::DeveloperToolsCredential;
use azure_messaging_eventhubs::ProducerClient;

let credential = DeveloperToolsCredential::new(None)?;
let producer = ProducerClient::builder()
    .open("<namespace>.servicebus.windows.net", "eventhub-name", credential.clone())
    .await?;
```

### Gửi Sự kiện Đơn

```rust
producer.send_event(vec![1, 2, 3, 4], None).await?;
```

### Gửi Lô (Batch)

```rust
let batch = producer.create_batch(None).await?;
batch.try_add_event_data(b"event 1".to_vec(), None)?;
batch.try_add_event_data(b"event 2".to_vec(), None)?;

producer.send_batch(batch, None).await?;
```

## Consumer Client

### Tạo Consumer

```rust
use azure_messaging_eventhubs::ConsumerClient;

let credential = DeveloperToolsCredential::new(None)?;
let consumer = ConsumerClient::builder()
    .open("<namespace>.servicebus.windows.net", "eventhub-name", credential.clone())
    .await?;
```

### Nhận Sự kiện

```rust
// Mở receiver cho partition cụ thể
let receiver = consumer.open_partition_receiver("0", None).await?;

// Nhận sự kiện
let events = receiver.receive_events(100, None).await?;
for event in events {
    println!("Event data: {:?}", event.body());
}
```

### Lấy Thuộc tính Event Hub

```rust
let properties = consumer.get_eventhub_properties(None).await?;
println!("Partitions: {:?}", properties.partition_ids);
```

### Lấy Thuộc tính Partition

```rust
let partition_props = consumer.get_partition_properties("0", None).await?;
println!("Last sequence number: {}", partition_props.last_enqueued_sequence_number);
```

## Thực hành Tốt nhất

1. **Tái sử dụng clients** — tạo một lần, gửi nhiều sự kiện
2. **Sử dụng lô** — hiệu quả hơn gửi riêng lẻ
3. **Kiểm tra dung lượng lô** — `try_add_event_data` trả về false khi đầy
4. **Xử lý partitions song song** — mỗi partition có thể được tiêu thụ độc lập
5. **Sử dụng consumer groups** — cô lập các ứng dụng tiêu thụ khác nhau
6. **Xử lý checkpointing** — sử dụng `azure_messaging_eventhubs_checkpointstore_blob` cho các consumers phân tán

## Checkpoint Store (Tùy chọn)

Cho các consumers phân tán với checkpointing:

```sh
cargo add azure_messaging_eventhubs_checkpointstore_blob
```

## Liên kết Tham khảo

| Tài nguyên    | Liên kết                                                                                      |
| ------------- | --------------------------------------------------------------------------------------------- |
| Tham khảo API | https://docs.rs/azure_messaging_eventhubs                                                     |
| Mã nguồn      | https://github.com/Azure/azure-sdk-for-rust/tree/main/sdk/eventhubs/azure_messaging_eventhubs |
| crates.io     | https://crates.io/crates/azure_messaging_eventhubs                                            |
