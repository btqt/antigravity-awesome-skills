# So sánh các Công cụ Chuyển biên (Transcription Tools Comparison)

So sánh toàn diện các công cụ chuyển biên âm thanh được hỗ trợ bởi kỹ năng audio-transcriber.

## Tổng quan

| Công cụ               | Loại           | Tốc độ     | Chất lượng | Chi phí    | Quyền riêng tư | Ngoại tuyến | Ngôn ngữ |
| --------------------- | -------------- | ---------- | ---------- | ---------- | -------------- | ----------- | -------- |
| **Faster-Whisper**    | Mã nguồn mở    | ⚡⚡⚡⚡⚡ | ⭐⭐⭐⭐⭐ | Miễn phí   | 100%           | ✅          | 99       |
| **Whisper**           | Mã nguồn mở    | ⚡⚡⚡     | ⭐⭐⭐⭐⭐ | Miễn phí   | 100%           | ✅          | 99       |
| Google Speech-to-Text | API Thương mại | ⚡⚡⚡⚡   | ⭐⭐⭐⭐⭐ | $0.006/15s | Một phần       | ❌          | 125+     |
| Azure Speech          | API Thương mại | ⚡⚡⚡⚡   | ⭐⭐⭐⭐   | $1/giờ     | Một phần       | ❌          | 100+     |
| AssemblyAI            | API Thương mại | ⚡⚡⚡⚡   | ⭐⭐⭐⭐⭐ | $0.00025/s | Một phần       | ❌          | 99       |

---

## Faster-Whisper (Khuyến nghị)

### Ưu điện

✅ **Nhanh hơn 4-5 lần** so với Whisper gốc  
✅ **Chất lượng tương đương** Whisper gốc  
✅ **Sử dụng ít bộ nhớ hơn** (RAM ít hơn 50-60%)  
✅ **Miễn phí và mã nguồn mở**  
✅ **Ngoại tuyến 100%** (đảm bảo quyền riêng tư)  
✅ **Cài đặt dễ dàng** (`pip install faster-whisper`)  
✅ **Thay thế hoàn hảo** cho Whisper

### Nhược điểm

❌ Yêu cầu Python 3.8+  
❌ Lần đầu tải mô hình hơi lâu (~100MB-1.5GB)  
❌ GPU là tùy chọn nhưng sẽ giúp tăng tốc đáng kể

### Cài đặt

```bash
pip install faster-whisper
```

### Ví dụ Sử dụng

```python
from faster_whisper import WhisperModel

# Tải mô hình (tự động tải trong lần chạy đầu tiên)
model = WhisperModel("base", device="cpu", compute_type="int8")

# Chuyển biên
segments, info = model.transcribe("audio.mp3", language="pt")

# In kết quả
for segment in segments:
    print(f"[{segment.start:.2f}s -> {segment.end:.2f}s] {segment.text}")
```

### Kích thước Mô hình

| Mô hình  | Kích thước | RAM    | Tốc độ (CPU)                    | Chất lượng |
| -------- | ---------- | ------ | ------------------------------- | ---------- |
| `tiny`   | 39 MB      | ~1 GB  | Rất nhanh (~10x thời gian thực) | Cơ bản     |
| `base`   | 74 MB      | ~1 GB  | Nhanh (~7x thời gian thực)      | Tốt        |
| `small`  | 244 MB     | ~2 GB  | Vừa phải (~4x thời gian thực)   | Rất tốt    |
| `medium` | 769 MB     | ~5 GB  | Chậm (~2x thời gian thực)       | Tuyệt vời  |
| `large`  | 1550 MB    | ~10 GB | Rất chậm (~1x thời gian thực)   | Tốt nhất   |

**Khuyến nghị:** Sử dụng `small` hoặc `medium` cho nhu cầu sản xuất.

---

## Whisper (Gốc)

### Ưu điểm

✅ **Mô hình chính thức của OpenAI**  
✅ **Chất lượng tuyệt vời**  
✅ **Miễn phí và mã nguồn mở**  
✅ **Ngoại tuyến 100%**  
✅ **Tài liệu đầy đủ**  
✅ **Cộng đồng lớn**

### Nhược điểm

❌ **Chậm hơn** Faster-Whisper (4-5 lần)  
❌ **Sử dụng nhiều bộ nhớ hơn**  
❌ Yêu cầu PyTorch (thành phần phụ thuộc lớn)  
❌ Khuyến nghị sử dụng GPU cho các mô hình lớn hơn

### Cài đặt

```bash
pip install openai-whisper
```

### Ví dụ Sử dụng

```python
import whisper

# Tải mô hình
model = whisper.load_model("base")

# Chuyển biên
result = model.transcribe("audio.mp3", language="pt")

# In kết quả
print(result["text"])
```

### Khi nào dùng Whisper so với Faster-Whisper

**Dùng Faster-Whisper nếu:**

- Tốc độ là yếu tố quan trọng
- Bộ nhớ RAM hạn chế
- Cần xử lý nhiều tệp

**Dùng Whisper gốc nếu:**

- Gặp vấn đề khi cài đặt Faster-Whisper
- Cần bản triển khai chính xác của OpenAI
- Đã có Whisper trong các thành phần phụ thuộc của dự án

---

## Google Cloud Speech-to-Text

### Ưu điểm

✅ **Rất chính xác** (hàng đầu trong ngành)  
✅ **Xử lý nhanh** (cơ sở hạ tầng đám mây)  
✅ **Hỗ trợ 125+ ngôn ngữ**  
✅ **Dấu thời gian ở cấp độ từ**  
✅ **Tự động thêm dấu câu và viết hoa**  
✅ **Nhận dạng người nói (diarization)** (phiên bản cao cấp)

### Nhược điểm

❌ **Yêu cầu internet** (chỉ dành cho đám mây)  
❌ **Tốn phí** (sau khi hết hạn mức miễn phí)  
❌ **Lo ngại về quyền riêng tư** (âm thanh được tải lên Google)  
❌ Yêu cầu thiết lập tài khoản GCP  
❌ Xác thực phức tạp

### Giá cả

- **Mức miễn phí:** 60 phút/tháng
- **Gói tiêu chuẩn:** $0.006 cho mỗi 15 giây ($1.44/giờ)
- **Gói cao cấp:** $0.009 cho mỗi 15 giây (kèm nhận dạng người nói)

### Cài đặt

```bash
pip install google-cloud-speech
```

### Thiết lập

1. Tạo dự án GCP
2. Bật Speech-to-Text API
3. Tạo tài khoản dịch vụ (service account) & tải tệp khóa JSON
4. Thiết lập biến môi trường:
   ```bash
   export GOOGLE_APPLICATION_CREDENTIALS="duong/dan/den/key.json"
   ```

### Ví dụ Sử dụng

```python
from google.cloud import speech

client = speech.SpeechClient()

with open("audio.wav", "rb") as audio_file:
    content = audio_file.read()

audio = speech.RecognitionAudio(content=content)
config = speech.RecognitionConfig(
    encoding=speech.RecognitionConfig.AudioEncoding.LINEAR16,
    sample_rate_hertz=16000,
    language_code="pt-BR",
)

response = client.recognize(config=config, audio=audio)

for result in response.results:
    print(result.alternatives[0].transcript)
```

---

## Azure Speech Services

### Ưu điểm

✅ **Độ chính xác cao**  
✅ **Hỗ trợ 100+ ngôn ngữ**  
✅ **Chuyển biên thời gian thực**  
✅ **Mô hình tùy chỉnh** (huấn luyện trên dữ liệu của bạn)  
✅ **Tích hợp tốt với hệ sinh thái Microsoft**

### Nhược điểm

❌ **Yêu cầu internet**  
❌ **Tốn phí** (sau khi hết hạn mức miễn phí)  
❌ **Lo ngại về quyền riêng tư** (xử lý trên đám mây)  
❌ Yêu cầu tài khoản Azure  
❌ Thiết lập phức tạp

### Giá cả

- **Mức miễn phí:** 5 giờ/tháng
- **Gói tiêu chuẩn:** $1.00 cho mỗi giờ âm thanh

### Cài đặt

```bash
pip install azure-cognitiveservices-speech
```

### Thiết lập

1. Tạo tài khoản Azure
2. Tạo tài nguyên Speech
3. Lấy API key và vùng (region)
4. Thiết lập biến môi trường:
   ```bash
   export AZURE_SPEECH_KEY="key-cua-ban"
   export AZURE_SPEECH_REGION="vung-cua-ban"
   ```

### Ví dụ Sử dụng

```python
import azure.cognitiveservices.speech as speechsdk

speech_config = speechsdk.SpeechConfig(
    subscription=os.environ.get('AZURE_SPEECH_KEY'),
    region=os.environ.get('AZURE_SPEECH_REGION')
)

audio_config = speechsdk.audio.AudioConfig(filename="audio.wav")
speech_recognizer = speechsdk.SpeechRecognizer(
    speech_config=speech_config,
    audio_config=audio_config
)

result = speech_recognizer.recognize_once()
print(result.text)
```

---

## AssemblyAI

### Ưu điểm

✅ **API hiện đại, thân thiện với nhà phát triển**  
✅ **Độ chính xác tuyệt vời**  
✅ **Các tính năng nâng cao** (phân tích cảm xúc, phát hiện chủ đề, xóa bỏ PII)  
✅ **Nhận dạng người nói (diarization)** (được bao gồm)  
✅ **Xử lý nhanh**  
✅ **Tài liệu hướng dẫn tốt**

### Nhược điểm

❌ **Yêu cầu internet**  
❌ **Tốn phí** (không có mức miễn phí, chỉ có credit dùng thử)  
❌ **Lo ngại về quyền riêng tư** (xử lý trên đám mây)  
❌ Yêu cầu API key

### Giá cả

- **Dùng thử miễn phí:** Credit trị giá $50
- **Gói tiêu chuẩn:** $0.00025 cho mỗi giây (~$0.90/giờ)

### Cài đặt

```bash
pip install assemblyai
```

### Thiết lập

1. Đăng ký tại assemblyai.com
2. Lấy API key
3. Thiết lập biến môi trường:
   ```bash
   export ASSEMBLYAI_API_KEY="key-cua-ban"
   ```

### Ví dụ Sử dụng

```python
import assemblyai as aai

aai.settings.api_key = os.environ["ASSEMBLYAI_API_KEY"]

transcriber = aai.Transcriber()
transcript = transcriber.transcribe("audio.mp3")

print(transcript.text)

# Nhận dạng người nói (Speaker diarization)
for utterance in transcript.utterances:
    print(f"Người nói {utterance.speaker}: {utterance.text}")
```

---

## Ma trận Khuyến nghị

### Sử dụng Faster-Whisper nếu:

- ✅ Quyền riêng tư là tối quan trọng (xử lý cục bộ)
- ✅ Muốn chi phí bằng không (miễn phí mãi mãi)
- ✅ Cần khả năng hoạt động ngoại tuyến
- ✅ Xử lý nhiều tệp (tốc độ là quan trọng)
- ✅ Ngân sách hạn chế

### Sử dụng Google Speech-to-Text nếu:

- ✅ Cần độ chính xác tuyệt đối tốt nhất
- ✅ Có ngân sách cho các dịch vụ đám mây
- ✅ Muốn các tính năng nâng cao (dấu câu, diarization)
- ✅ Đang sử dụng hệ sinh thái GCP

### Sử dụng Azure Speech nếu:

- ✅ Đang trong hệ sinh thái Microsoft
- ✅ Cần huấn luyện mô hình tùy chỉnh
- ✅ Muốn chuyển biên thời gian thực
- ✅ Có sẵn credit Azure

### Sử dụng AssemblyAI nếu:

- ✅ Cần các tính năng nâng cao (cảm xúc, chủ đề)
- ✅ Muốn trải nghiệm API dễ dàng nhất
- ✅ Cần tự động xóa bỏ PII (thông tin định danh cá nhân)
- ✅ Coi trọng trải nghiệm của nhà phát triển

---

## Điểm chuẩn Hiệu suất (Performance Benchmarks)

**Thử nghiệm:** Podcast dài 1 giờ (MP3, 44.1kHz, stereo)

| Công cụ                | Thời gian xử lý | Độ chính xác | Chi phí |
| ---------------------- | --------------- | ------------ | ------- |
| Faster-Whisper (small) | 8 phút          | 94%          | $0      |
| Whisper (small)        | 32 phút         | 94%          | $0      |
| Google Speech          | 2 phút          | 96%          | $1.44   |
| Azure Speech           | 3 phút          | 95%          | $1.00   |
| AssemblyAI             | 4 phút          | 96%          | $0.90   |

_Điểm chuẩn được chạy trên MacBook Pro M1, 16GB RAM_

---

## Kết luận

**Đối với kỹ năng audio-transcriber:**

1. **Chính:** Faster-Whisper (sự cân bằng tốt nhất giữa tốc độ, chất lượng, quyền riêng tư và chi phí)
2. **Dự phòng:** Whisper (nếu Faster-Whisper không khả dụng)
3. **Tùy chọn:** Các API đám mây (người dùng lựa chọn cho các tính năng cao cấp)

Điều này đảm bảo kỹ năng hoạt động ngay lập tức cho hầu hết người dùng trong khi cho phép những người dùng nâng cao tích hợp các dịch vụ thương mại nếu cần.
