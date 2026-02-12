---
name: audio-transcriber
description: "Chuyển đổi các bản ghi âm thành tài liệu Markdown chuyên nghiệp với các tóm tắt thông minh bằng cách sử dụng tích hợp LLM"
version: 1.2.0
author: Eric Andrade
created: 2025-02-01
updated: 2026-02-04
platforms: [github-copilot-cli, claude-code, codex]
category: content
tags: [audio, transcription, whisper, meeting-minutes, speech-to-text]
risk: safe
---

## Mục đích

Kỹ năng này tự động hóa việc chuyển đổi âm thanh thành văn bản với đầu ra Markdown chuyên nghiệp, trích xuất siêu dữ liệu kỹ thuật phong phú (người nói, dấu thời gian, ngôn ngữ, kích thước tệp, thời lượng) và tạo biên bản cuộc họp cũng như tóm tắt điều hành có cấu trúc. Nó sử dụng Faster-Whisper hoặc Whisper với cấu hình bằng không (zero configuration), hoạt động toàn diện trên các dự án mà không cần các đường dẫn được mã hóa cứng hoặc mã khóa API.

Lấy cảm hứng từ các công cụ như Plaud, kỹ năng này chuyển đổi các bản ghi âm thô thành tài liệu có thể hành động, làm cho nó trở nên lý tưởng cho các cuộc họp, phỏng vấn, bài giảng và phân tích nội dung.

## Khi nào sử dụng

Sử dụng kỹ năng này khi:

- Người dùng cần chuyển biên các tệp âm thanh/video thành văn bản
- Người dùng muốn biên bản cuộc họp được tạo tự động từ các bản ghi âm
- Người dùng yêu cầu nhận dạng người nói (diarization) trong các cuộc hội thoại
- Người dùng cần phụ đề (định dạng SRT, VTT)
- Người dùng muốn tóm tắt điều hành cho nội dung âm thanh dài
- Người dùng hỏi các biến thể của "transcribe this audio" (chuyển biên âm thanh này), "convert audio to text" (chuyển âm thanh thành văn bản), "generate meeting notes from recording" (tạo ghi chú họp từ bản ghi âm)
- Người dùng có các tệp âm thanh ở các định dạng phổ biến (MP3, WAV, M4A, OGG, FLAC, WEBM)

## Quy trình công việc

### Bước 0: Khám phá (Tự động phát hiện các công cụ chuyển biên)

**Mục tiêu:** Xác định các công cụ chuyển biên có sẵn mà không cần người dùng cấu hình.

**Hành động:**

Chạy các lệnh phát hiện để tìm các công cụ đã cài đặt:

```bash
# Kiểm tra Faster-Whisper (được ưu tiên - nhanh hơn 4-5 lần)
if python3 -c "import faster_whisper" 2>/dev/null; then
    TRANSCRIBER="faster-whisper"
    echo "✅ Đã phát hiện Faster-Whisper (tối ưu hóa)"
# Dự phòng bằng Whisper gốc
elif python3 -c "import whisper" 2>/dev/null; then
    TRANSCRIBER="whisper"
    echo "✅ Đã phát hiện OpenAI Whisper"
else
    TRANSCRIBER="none"
    echo "⚠️ Không tìm thấy công cụ chuyển biên nào"
fi

# Kiểm tra ffmpeg (chuyển đổi định dạng âm thanh)
if command -v ffmpeg &>/dev/null; then
    echo "✅ ffmpeg khả dụng (đã bật chuyển đổi định dạng)"
else
    echo "ℹ️ Không tìm thấy ffmpeg (hỗ trợ định dạng hạn chế)"
fi
```

**Nếu không tìm thấy công cụ chuyển biên:**

Đề xuất cài đặt tự động bằng tập lệnh được cung cấp:

```bash
echo "⚠️ Không tìm thấy công cụ chuyển biên nào"
echo ""
echo "🔧 Tự động cài đặt các thành phần phụ thuộc? (Khuyến nghị)"
read -p "Chạy tập lệnh cài đặt? [Y/n]: " AUTO_INSTALL

if [[ ! "$AUTO_INSTALL" =~ ^[Nn] ]]; then
    # Lấy thư mục kỹ năng (hoạt động cho cả cài đặt repo và symlinked)
    SKILL_DIR="$(cd "$(dirname "${BASH_SOURCE[0]}")" && pwd)"

    # Chạy tập lệnh cài đặt
    if [[ -f "$SKILL_DIR/scripts/install-requirements.sh" ]]; then
        bash "$SKILL_DIR/scripts/install-requirements.sh"
    else
        echo "❌ Không tìm thấy tập lệnh cài đặt"
        echo ""
        echo "📦 Cài đặt thủ công:"
        echo "  pip install faster-whisper  # Khuyến nghị"
        echo "  pip install openai-whisper  # Thay thế"
        echo "  brew install ffmpeg         # Tùy chọn (macOS)"
        exit 1
    fi

    # Xác minh cài đặt thành công
    if python3 -c "import faster_whisper" 2>/dev/null || python3 -c "import whisper" 2>/dev/null; then
        echo "✅ Cài đặt thành công! Đang tiến hành chuyển biên..."
    else
        echo "❌ Cài đặt thất bại. Vui lòng cài đặt thủ công."
        exit 1
    fi
else
    echo ""
    echo "📦 Yêu cầu cài đặt thủ công:"
    echo ""
    echo "Khuyến nghị (nhanh nhất):"
    echo "  pip install faster-whisper"
    echo ""
    echo "Thay thế (gốc):"
    echo "  pip install openai-whisper"
    echo ""
    echo "Tùy chọn (chuyển đổi định dạng):"
    echo "  brew install ffmpeg  # macOS"
    echo "  apt install ffmpeg   # Linux"
    echo ""
    exit 1
fi
```

Điều này đảm bảo người dùng có thể cài đặt các thành phần phụ thuộc với một lần xác nhận, hoặc chọn cài đặt thủ công nếu muốn.

**Nếu tìm thấy công cụ chuyển biên:**

Tiến hành Bước 0b (Phát hiện CLI).

### Bước 1: Xác thực Tệp Âm thanh

**Mục tiêu:** Xác minh tệp tồn tại, kiểm tra định dạng và trích xuất siêu dữ liệu.

**Hành động:**

1. **Chấp nhận đường dẫn tệp hoặc URL** từ người dùng:
   - Tệp cục bộ: `meeting.mp3`
   - URL: `https://example.com/audio.mp3` (tải xuống thư mục tạm thời)

2. **Xác minh tệp tồn tại:**

```bash
if [[ ! -f "$AUDIO_FILE" ]]; then
    echo "❌ Không tìm thấy tệp: $AUDIO_FILE"
    exit 1
fi
```

3. **Trích xuất siêu dữ liệu** sử dụng ffprobe hoặc các tiện ích tệp:

```bash
# Lấy kích thước tệp
FILE_SIZE=$(du -h "$AUDIO_FILE" | cut -f1)

# Lấy thời lượng và định dạng sử dụng ffprobe
DURATION=$(ffprobe -v error -show_entries format=duration \
    -of default=noprint_wrappers=1:nokey=1 "$AUDIO_FILE" 2>/dev/null)
FORMAT=$(ffprobe -v error -select_streams a:0 -show_entries \
    stream=codec_name -of default=noprint_wrappers=1:nokey=1 "$AUDIO_FILE" 2>/dev/null)

# Chuyển đổi thời lượng sang HH:MM:SS
DURATION_HMS=$(date -u -r "$DURATION" +%H:%M:%S 2>/dev/null || echo "Unknown")
```

4. **Kiểm tra kích thước tệp** (cảnh báo nếu lớn đối với các API đám mây):

```bash
SIZE_MB=$(du -m "$AUDIO_FILE" | cut -f1)
if [[ $SIZE_MB -gt 25 ]]; then
    echo "⚠️ Tệp lớn ($FILE_SIZE) - quá trình xử lý có thể mất vài phút"
fi
```

5. **Xác thực định dạng** (hỗ trợ: MP3, WAV, M4A, OGG, FLAC, WEBM):

```bash
EXTENSION="${AUDIO_FILE##*.}"
SUPPORTED_FORMATS=("mp3" "wav" "m4a" "ogg" "flac" "webm" "mp4")

if [[ ! " ${SUPPORTED_FORMATS[@]} " =~ " ${EXTENSION,,} " ]]; then
    echo "⚠️ Định dạng không được hỗ trợ: $EXTENSION"
    if command -v ffmpeg &>/dev/null; then
        echo "🔄 Đang chuyển đổi sang WAV..."
        ffmpeg -i "$AUDIO_FILE" -ar 16000 "${AUDIO_FILE%.*}.wav" -y
        AUDIO_FILE="${AUDIO_FILE%.*}.wav"
    else
        echo "❌ Cài đặt ffmpeg để chuyển đổi định dạng: brew install ffmpeg"
        exit 1
    fi
fi
```

### Bước 3: Tạo Đầu ra Markdown

**Mục tiêu:** Tạo Markdown có cấu trúc với siêu dữ liệu, bản chuyển biên, biên bản cuộc họp và tóm tắt.

**Mẫu đầu ra:**

```markdown
# Báo cáo Chuyển biên Âm thanh

## 📊 Siêu dữ liệu (Metadata)

| Trường                       | Giá trị                      |
| ---------------------------- | ---------------------------- |
| **Tên tệp**                  | {filename}                   |
| **Kích thước tệp**           | {file_size}                  |
| **Thời lượng**               | {duration_hms}               |
| **Ngôn ngữ**                 | {language} ({language_code}) |
| **Ngày xử lý**               | {process_date}               |
| **Người nói được nhận dạng** | {num_speakers}               |
| **Công cụ chuyển biên**      | {engine} (mô hình: {model})  |

## 📋 Biên bản cuộc họp

### Người tham gia

- {speaker_1}
- {speaker_2}
- ...

### Các chủ đề đã thảo luận

1. **{topic_1}** ({timestamp})
   - {key_point_1}
   - {key_point_2}

2. **{topic_2}** ({timestamp})
   - {key_point_1}

### Các quyết định đã đưa ra

- ✅ {decision_1}
- ✅ {decision_2}

### Các mục hành động (Action Items)

- [ ] **{action_1}** - Được giao cho: {speaker} - Hạn: {date_if_mentioned}
- [ ] **{action_2}** - Được giao cho: {speaker}

_Được tạo bởi kỹ năng audio-transcriber v1.0.0_  
_Công cụ chuyển biên: {engine} | Thời gian xử lý: {elapsed_time}s_
```

**Triển khai:**

Sử dụng Python hoặc bash với mô hình AI (Claude/GPT) để tóm tắt thông minh:

```python
def generate_meeting_minutes(segments):
    """Trích xuất các chủ đề, quyết định, mục hành động từ bản chuyển biên."""

    # Nhóm các phân đoạn theo chủ đề (gom cụm đơn giản theo dấu thời gian)
    topics = cluster_by_topic(segments)

    # Xác định các mục hành động (từ khóa: "nên", "sẽ", "cần phải", "hành động")
    action_items = extract_action_items(segments)

    # Xác định các quyết định (từ khóa: "đã quyết định", "đã đồng ý", "đã phê duyệt")
    decisions = extract_decisions(segments)

    return {
        "topics": topics,
        "decisions": decisions,
        "action_items": action_items
    }

def generate_summary(segments, max_paragraphs=5):
    """Tạo tóm tắt điều hành bằng AI (Claude/GPT qua API hoặc mô hình cục bộ)."""

    full_text = " ".join([s["text"] for s in segments])

    # Sử dụng phương pháp Chain of Density (từ các khung công tác kỹ thuật prompt)
    summary_prompt = f"""
    Tóm tắt bản chuyển biên sau đây trong {max_paragraphs} đoạn văn ngắn gọn.
    Tập trung vào các chủ đề chính, quyết định và mục hành động.

    Bản chuyển biên:
    {full_text}
    """

    # Gọi mô hình AI (chỗ trống - người dùng có thể tích hợp API Claude hoặc sử dụng mô hình cục bộ)
    summary = call_ai_model(summary_prompt)

    return summary
```

**Đặt tên tệp đầu ra:**

```bash
# v1.1.0: Sử dụng dấu thời gian để tránh ghi đè
TIMESTAMP=$(date +%Y%m%d-%H%M%S)
TRANSCRIPT_FILE="transcript-${TIMESTAMP}.md"
ATA_FILE="ata-${TIMESTAMP}.md"

echo "$TRANSCRIPT_CONTENT" > "$TRANSCRIPT_FILE"
echo "✅ Bản chuyển biên đã được lưu: $TRANSCRIPT_FILE"

if [[ -n "$ATA_CONTENT" ]]; then
    echo "$ATA_CONTENT" > "$ATA_FILE"
    echo "✅ Biên bản đã được lưu: $ATA_FILE"
fi
```

#### **KỊCH BẢN A: Người dùng Cung cấp Prompt Tùy chỉnh**

**Quy trình:**

1. **Hiển thị prompt của người dùng:**

   ```
   📝 Prompt được cung cấp bởi người dùng:
   ┌──────────────────────────────────┐
   │ [Bản xem trước prompt của người dùng] │
   └──────────────────────────────────┘
   ```

2. **Tự động cải thiện bằng kỹ sư prompt (nếu có):**

   ```bash
   🔧 Đang cải thiện prompt với prompt-engineer...
   [Thực hiện: gh copilot -p "cải thiện prompt này: {user_prompt}"]
   ```

3. **Hiển thị cả hai phiên bản:**

   ```
   ✨ Phiên bản đã được cải thiện:
   ┌──────────────────────────────────┐
   │ Vai trò: Bạn là một chuyên gia lập tài liệu... │
   │ Hướng dẫn: Chuyển đổi...         │
   │ Các bước: 1) ... 2) ...          │
   │ Mục tiêu cuối cùng: ...           │
   └──────────────────────────────────┘

   📝 Phiên bản gốc:
   ┌──────────────────────────────────┐
   │ [Prompt gốc của người dùng]        │
   └──────────────────────────────────┘
   ```

4. **Hỏi dùng phiên bản nào:**

   ```bash
   💡 Sử dụng phiên bản đã cải thiện? [y/n] (mặc định: y):
   ```

5. **Xử lý với prompt đã chọn:**
   - Nếu "y": sử dụng bản đã cải thiện
   - Nếu "n": sử dụng bản gốc

#### **Xử lý LLM (Cả hai kịch bản)**

Khi prompt đã được chốt:

```python
from rich.progress import Progress, SpinnerColumn, TextColumn

def process_with_llm(transcript, prompt, cli_tool='claude'):
    full_prompt = f"{prompt}\n\n---\n\nBản chuyển biên:\n\n{transcript}"

    with Progress(
        SpinnerColumn(),
        TextColumn("[progress.description]{task.description}"),
        transient=True
    ) as progress:
        progress.add_task(
            description=f"🤖 Đang xử lý với {cli_tool}...",
            total=None
        )

        if cli_tool == 'claude':
            result = subprocess.run(
                ['claude', '-'],
                input=full_prompt,
                capture_output=True,
                text=True,
                timeout=300  # 5 phút
            )
        elif cli_tool == 'gh-copilot':
            result = subprocess.run(
                ['gh', 'copilot', 'suggest', '-t', 'shell', full_prompt],
                capture_output=True,
                text=True,
                timeout=300
            )

    if result.returncode == 0:
        return result.stdout.strip()
    else:
        return None
```

**Đầu ra tiến trình:**

```
🤖 Đang xử lý với claude... ⠋
[Sau khi hoàn thành:]
✅ Biên bản đã được tạo thành công!
```

#### **Đầu ra Cuối cùng**

**Thành công (cả hai tệp):**

```bash
💾 Đang lưu các tệp...

✅ Các tệp đã được tạo:
  - transcript-20260203-023045.md  (bản chuyển biên thuần)
  - ata-20260203-023045.md         (đã được xử lý bởi LLM)

🧹 Đã xóa các tệp tạm thời: metadata.json, transcription.json

✅ Đã hoàn thành! Tổng thời gian: 3m 45s
```

**Chỉ bản chuyển biên (người dùng từ chối LLM):**

```bash
💾 Đang lưu các tệp...

✅ Tệp đã được tạo:
  - transcript-20260203-023045.md

ℹ️ Biên bản chưa được tạo (người dùng từ chối xử lý LLM)

🧹 Đã xóa các tệp tạm thời: metadata.json, transcription.json

✅ Đã hoàn thành!
```

### Bước 5: Hiển thị Tóm tắt Kết quả

**Mục tiêu:** Hiển thị trạng thái hoàn thành và các bước tiếp theo.

**Đầu ra:**

```bash
echo ""
echo "✅ Chuyển biên Đã hoàn thành!"
echo ""
echo "📊 Kết quả:"
echo "  Tệp: $OUTPUT_FILE"
echo "  Ngôn ngữ: $LANGUAGE"
echo "  Thời lượng: $DURATION_HMS"
echo "  Người nói: $NUM_SPEAKERS"
echo "  Số từ: $WORD_COUNT"
echo "  Thời gian xử lý: ${ELAPSED_TIME}s"
echo ""
echo "📝 Đã tạo:"
echo "  - $OUTPUT_FILE (Báo cáo Markdown)"
[nếu có các định dạng thay thế:]
echo "  - ${OUTPUT_FILE%.*}.srt (Phụ đề)"
echo "  - ${OUTPUT_FILE%.*}.json (Dữ liệu có cấu trúc)"
echo ""
echo "🎯 Các bước tiếp theo:"
echo "  1. Xem lại biên bản cuộc họp và các mục hành động"
echo "  2. Chia sẻ báo cáo với những người tham gia"
echo "  3. Theo dõi các mục hành động cho đến khi hoàn thành"
```

## Ví dụ Sử dụng

### **Ví dụ 1: Chuyển biên Cơ bản**

**Đầu vào của Người dùng:**

```bash
copilot> chuyển biên âm thanh sang markdown: meeting-2026-02-02.mp3
```

**Đầu ra của Kỹ năng:**

```bash
✅ Đã phát hiện Faster-Whisper (tối ưu hóa)
✅ ffmpeg khả dụng (đã bật chuyển đổi định dạng)

📂 Tệp: meeting-2026-02-02.mp3
📊 Kích thước: 12.3 MB
⏱️ Thời lượng: 00:45:32

🎙️ Đang xử lý...
[████████████████████] 100%

✅ Đã phát hiện ngôn ngữ: Tiếng Bồ Đào Nha (pt-BR)
👥 Người nói được nhận dạng: 4
📝 Đang tạo đầu ra Markdown...

✅ Chuyển biên Đã hoàn thành!

📊 Kết quả:
  Tệp: meeting-2026-02-02.md
  Ngôn ngữ: pt-BR
  Thời lượng: 00:45:32
  Người nói: 4
  Số từ: 6,842
  Thời gian xử lý: 127s

📝 Đã tạo:
  - meeting-2026-02-02.md (Báo cáo Markdown)

🎯 Các bước tiếp theo:
  1. Xem lại biên bản cuộc họp và các mục hành động
  2. Chia sẻ báo cáo với những người tham gia
  3. Theo dõi các mục hành động cho đến khi hoàn thành
```

### **Ví dụ 3: Xử lý theo Lô (Batch Processing)**

**Đầu vào của Người dùng:**

```bash
copilot> transcreva estes áudios: recordings/*.mp3
```

**Đầu ra của Kỹ năng:**

```bash
📦 Chế độ lô: đã tìm thấy 5 tệp
  1. team-standup.mp3
  2. client-call.mp3
  3. brainstorm-session.mp3
  4. product-demo.mp3
  5. retrospective.mp3

🎙️ Đang xử lý theo lô...

[1/5] team-standup.mp3 ✅ (2m 34s)
[2/5] client-call.mp3 ✅ (15m 12s)
[3/5] brainstorm-session.mp3 ✅ (8m 47s)
[4/5] product-demo.mp3 ✅ (22m 03s)
[5/5] retrospective.mp3 ✅ (11m 28s)

✅ Lô đã hoàn thành!
📝 Đã tạo 5 báo cáo Markdown
⏱️ Tổng thời gian xử lý: 6m 15s
```

### **Ví dụ 5: Cảnh báo Tệp Lớn**

**Đầu vào của Người dùng:**

```bash
copilot> transcribe audio to markdown: conference-keynote.mp3
```

**Đầu ra của Kỹ năng:**

```bash
✅ Đã phát hiện Faster-Whisper (tối ưu hóa)

📂 Tệp: conference-keynote.mp3
📊 Kích thước: 87.2 MB
⏱️ Thời lượng: 02:15:47
⚠️ Tệp lớn (87.2 MB) - quá trình xử lý có thể mất vài phút

Tiếp tục? [Y/n]:
```

**Người dùng:** `Y`

```bash
🎙️ Đang xử lý... (quá trình này có thể mất 10-15 phút)
[████░░░░░░░░░░░░░░░░] 20% - Thời gian còn lại ước tính: 12m
```

Kỹ năng này là **độc lập với nền tảng** và hoạt động trong bất kỳ ngữ cảnh terminal nào mà GitHub Copilot CLI khả dụng. Nó không phụ thuộc vào các cấu hình dự án cụ thể hoặc các API bên ngoài, tuân theo triết lý cấu hình bằng không.
