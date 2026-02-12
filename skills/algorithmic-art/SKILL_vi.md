---
name: algorithmic-art
description: Tạo nghệ thuật thuật toán (algorithmic art) bằng p5.js với tính ngẫu nhiên có hạt giống (seeded randomness) và khám phá các tham số tương tác. Sử dụng khi người dùng yêu cầu tạo nghệ thuật bằng mã nguồn, nghệ thuật tạo sinh (generative art), nghệ thuật thuật toán, trường dòng chảy (flow fields) hoặc hệ thống hạt (particle systems). Hãy tạo ra nghệ thuật thuật toán nguyên bản thay vì sao chép tác phẩm của các nghệ sĩ hiện có để tránh vi phạm bản quyền.
license: Các điều khoản đầy đủ trong LICENSE.txt
---

Các triết lý thuật toán là những trào lưu thẩm mỹ tính toán sau đó được thể hiện qua mã nguồn. Đầu ra bao gồm các tệp .md (triết lý), .html (trình xem tương tác) và .js (các thuật toán tạo sinh).

Quá trình này diễn ra qua hai bước:

1. Tạo Triết lý Thuật toán (tệp .md)
2. Thể hiện bằng cách tạo nghệ thuật tạo sinh p5.js (tệp .html + .js)

Đầu tiên, hãy thực hiện nhiệm vụ này:

## TẠO TRIẾT LÝ THUẬT TOÁN

Để bắt đầu, hãy tạo một TRIẾT LÝ THUẬT TOÁN (không phải hình ảnh tĩnh hay mẫu có sẵn) sẽ được diễn giải thông qua:

- Các quy trình tính toán, hành vi mới nổi (emergent behavior), vẻ đẹp toán học
- Tính ngẫu nhiên có hạt giống, các trường nhiễu (noise fields), các hệ thống hữu cơ
- Các hạt, dòng chảy, trường lực, các lực
- Sự thay đổi tham số và sự hỗn loạn có kiểm soát

### SỰ HIỂU BIẾT CỐT LÕI

- Những gì nhận được: Một số gợi ý hoặc hướng dẫn tinh tế từ người dùng để xem xét, nhưng hãy coi đó là nền tảng; nó không nên hạn chế sự tự do sáng tạo.
- Những gì được tạo ra: Một trào lưu thẩm mỹ tạo sinh/triết lý thuật toán.
- Những gì xảy ra tiếp theo: Cùng một phiên bản nhận triết lý này và THỂ HIỆN NÓ TRONG MÃ NGUỒN - tạo ra các bản phác thảo (sketches) p5.js với 90% là tạo sinh bằng thuật toán, 10% là các tham số thiết yếu.

Hãy cân nhắc cách tiếp cận này:

- Viết một bản tuyên ngôn (manifesto) cho trào lưu nghệ thuật tạo sinh
- Giai đoạn tiếp theo bao gồm việc viết thuật toán để biến nó thành hiện thực

Triết lý phải nhấn mạnh: Biểu đạt thuật toán. Hành vi mới nổi. Vẻ đẹp tính toán. Sự biến đổi có hạt giống.

### CÁCH TẠO MỘT TRIẾT LÝ THUẬT TOÁN

**Đặt tên trào lưu** (1-2 từ): "Nhiễu loạn hữu cơ" / "Hòa âm lượng tử" / "Sự tĩnh lặng mới nổi" (Organic Turbulence / Quantum Harmonics / Emergent Stillness)

**Trình bày triết lý** (4-6 đoạn văn - súc tích nhưng đầy đủ):

Để nắm bắt được bản chất THUẬT TOÁN, hãy diễn đạt triết lý này thể hiện qua:

- Các quy trình tính toán và các mối quan hệ toán học?
- Các hàm nhiễu và các mẫu ngẫu nhiên?
- Các hành vi của hạt và động lực học trường lực?
- Sự tiến hóa theo thời gian và các trạng thái hệ thống?
- Sự thay đổi tham số và độ phức tạp mới nổi?

**HƯỚNG DẪN QUAN TRỌNG:**

- **Tránh dư thừa**: Mỗi khía cạnh thuật toán chỉ nên được đề cập một lần. Tránh lặp lại các khái niệm về lý thuyết nhiễu, động lực học hạt hoặc các nguyên lý toán học trừ khi thêm vào chiều sâu mới.
- **LẶP LẠI việc nhấn mạnh tay nghề tinh xảo**: Triết lý PHẢI nhấn mạnh nhiều lần rằng thuật toán cuối cùng sẽ trông như thể đã mất vô số giờ để phát triển, được tinh chỉnh cẩn thận và đến từ một người ở trình độ cao nhất trong lĩnh vực của họ. Việc đóng khung này là thiết yếu - hãy lặp lại các cụm từ như "thuật toán được chế tác tỉ mỉ", "sản phẩm của chuyên môn tính toán sâu sắc", "tối ưu hóa kỳ công", "triển khai ở mức độ bậc thầy".
- **Để lại không gian sáng tạo**: Hãy cụ thể về định hướng thuật toán, nhưng đủ súc tích để phiên bản Claude tiếp theo có không gian đưa ra các lựa chọn triển khai diễn giải ở mức độ tay nghề cực cao.

Triết lý phải hướng dẫn phiên bản tiếp theo thể hiện các ý tưởng bằng THUẬT TOÁN, không phải thông qua hình ảnh tĩnh. Vẻ đẹp nằm ở quy trình, không phải ở khung hình cuối cùng.

### CÁC VÍ DỤ TRIẾT LÝ

**"Organic Turbulence" (Nhiễu loạn hữu cơ)**
Triết lý: Sự hỗn loạn bị hạn chế bởi định luật tự nhiên, trật tự trỗi dậy từ sự mất trật tự.
Biểu đạt thuật toán: Các trường dòng chảy được thúc đẩy bởi nhiễu Perlin phân lớp. Hàng ngàn hạt đi theo các lực vector, dấu vết của chúng tích tụ thành các bản đồ mật độ hữu cơ. Nhiều quãng tám nhiễu (noise octaves) tạo ra các vùng nhiễu loạn và vùng tĩnh lặng. Màu sắc trỗi dậy từ vận tốc và mật độ - các hạt nhanh rực sáng, các hạt chậm mờ dần vào bóng tối. Thuật toán chạy cho đến khi đạt trạng thái cân bằng - một sự cân bằng được tinh chỉnh tỉ mỉ, nơi mọi tham số đều được trau chuốt qua vô số lần lặp bởi một bậc thầy về thẩm mỹ tính toán.

**"Quantum Harmonics" (Hòa âm lượng tử)**
Triết lý: Các thực thể rời rạc thể hiện các mẫu giao thoa dạng sóng.
Biểu đạt thuật toán: Các hạt được khởi tạo trên một lưới, mỗi hạt mang một giá trị pha (phase) tiến hóa thông qua các sóng sin. Khi các hạt ở gần nhau, các pha của chúng giao thoa - giao thoa tăng cường tạo ra các nút sáng, giao thoa triệt tiêu tạo ra các khoảng trống. Chuyển động hài hòa đơn giản tạo ra các mandala (mạn đà la) mới nổi phức tạp. Kết quả của việc hiệu chuẩn tần số kỳ công, nơi mọi tỷ lệ đều được lựa chọn cẩn thận để tạo ra vẻ đẹp cộng hưởng.

**"Recursive Whispers" (Tiếng thì thầm đệ quy)**
Triết lý: Sự tự tương đồng qua các quy mô, chiều sâu vô tận trong không gian hữu hạn.
Biểu đạt thuật toán: Các cấu trúc phân nhánh chia nhỏ một cách đệ quy. Mỗi nhánh được ngẫu nhiên hóa một chút nhưng bị ràng buộc bởi các tỷ lệ vàng. Các hệ thống L-systems hoặc phân chia đệ quy tạo ra các hình dạng giống như cây cối, mang lại cảm giác vừa toán học vừa hữu cơ. Các nhiễu động tinh tế phá vỡ sự đối xứng hoàn hảo. Độ dày của đường nét giảm dần sau mỗi mức đệ quy. Mọi góc phân nhánh là sản phẩm của quá trình khám phá toán học sâu sắc.

**"Field Dynamics" (Động lực học trường lực)**
Triết lý: Các lực vô hình được hiển lộ qua tác động của chúng lên vật chất.
Biểu đạt thuật toán: Các trường vector được xây dựng từ các hàm toán học hoặc nhiễu. Các hạt sinh ra ở các cạnh, chảy dọc theo các đường sức trường lực, chết đi khi chúng đạt đến trạng thái cân bằng hoặc các biên giới. Nhiều trường lực có thể thu hút, đẩy hoặc xoay các hạt. Sự hình ảnh hóa chỉ hiển thị các dấu vết - bằng chứng giống như bóng ma của các lực vô hình. Một điệu nhảy tính toán được dàn dựng tỉ mỉ thông qua sự cân bằng lực.

**"Stochastic Crystallization" (Kết tinh ngẫu nhiên)**
Triết lý: Các quá trình ngẫu nhiên kết tinh thành các cấu trúc trật tự.
Biểu đạt thuật toán: Phân bổ vòng tròn ngẫu nhiên hoặc phân vùng Voronoi. Bắt đầu với các điểm ngẫu nhiên, để chúng tiến hóa qua các thuật toán nới lỏng (relaxation algorithms). Các ô đẩy nhau cho đến khi đạt trạng thái cân bằng. Màu sắc dựa trên kích thước ô, số lượng lân cận hoặc khoảng cách từ trung tâm. Cách lát nền hữu cơ trỗi dậy mang lại cảm giác vừa ngẫu nhiên vừa tất yếu. Mỗi hạt giống (seed) tạo ra vẻ đẹp kết tinh độc nhất - dấu ấn của một thuật toán tạo sinh bậc thầy.

_Đây là các ví dụ rút gọn. Triết lý thuật toán thực tế nên dài từ 4-6 đoạn văn đáng kể._

### CÁC NGUYÊN TẮC THIẾT YẾU

- **TRIẾT LÝ THUẬT TOÁN**: Tạo ra một thế giới quan tính toán để thể hiện qua mã nguồn.
- **QUY TRÌNH QUAN TRỌNG HƠN SẢN PHẨM**: Luôn nhấn mạnh rằng vẻ đẹp nảy sinh từ việc thực thi thuật toán - mỗi lần chạy là duy nhất.
- **BIỂU ĐẠT THAM SỐ**: Các ý tưởng truyền tải qua các mối quan hệ toán học, các lực, các hành vi - không phải qua bố cục tĩnh.
- **TỰ DO NGHỆ THUẬT**: Phiên bản Claude tiếp theo sẽ diễn giải triết lý bằng thuật toán - hãy để lại không gian cho các lựa chọn triển khai sáng tạo.
- **NGHỆ THUẬT TẠO SINH THUẦN TÚY**: Đây là về việc tạo ra các THUẬT TOÁN SỐNG, không phải là các hình ảnh tĩnh có tính ngẫu nhiên.
- **TAY NGHỀ CHUYÊN GIA**: Lặp lại việc nhấn mạnh rằng thuật toán cuối cùng phải mang lại cảm giác được chế tác tỉ mỉ, tinh chỉnh qua vô số lần lặp, là sản phẩm chuyên môn sâu sắc của một người ở trình độ cao nhất trong lĩnh vực thẩm mỹ tính toán.

**Triết lý thuật toán nên dài từ 4-6 đoạn văn.** Hãy lấp đầy nó bằng triết lý tính toán đầy chất thơ mang lại tầm nhìn mong muốn. Tránh lặp lại cùng một điểm. Xuất triết lý thuật toán này dưới dạng tệp .md.

---

## SUY LUẬN HẠT GIỐNG KHÁI NIỆM

**BƯỚC QUAN TRỌNG**: Trước khi triển khai thuật toán, hãy xác định sợi dây khái niệm tinh tế từ yêu cầu ban đầu.

**NGUYÊN TẮC THIẾT YẾU**:
Khái niệm này là một **tham chiếu tinh tế, ngách được nhúng ngay trong chính thuật toán** - không phải lúc nào cũng theo nghĩa đen, nhưng luôn tinh vi. Một người quen thuộc với chủ đề này sẽ cảm nhận được nó một cách trực giác, trong khi những người khác chỉ đơn giản là trải nghiệm một tác phẩm tạo sinh bậc thầy. Triết lý thuật toán cung cấp ngôn ngữ tính toán. Khái niệm được suy luận cung cấp phần hồn - DNA khái niệm thầm lặng được dệt một cách vô hình vào các tham số, hành vi và các mẫu mới nổi.

Điều này **RẤT QUAN TRỌNG**: Tham chiếu phải tinh tế đến mức nó làm tăng thêm chiều sâu cho tác phẩm mà không cần tự xướng tên. Hãy nghĩ như một nhạc sĩ jazz trích dẫn một bài hát khác thông qua sự hòa hợp thuật toán - chỉ những ai biết mới nhận ra, nhưng mọi người đều đánh giá cao vẻ đẹp tạo sinh.

---

## TRIỂN KHAI P5.JS

Với triết lý VÀ khung khái niệm đã được thiết lập, hãy thể hiện nó thông qua mã nguồn. Dừng lại một chút để tập hợp các suy nghĩ trước khi tiếp tục. Chỉ sử dụng triết lý thuật toán đã tạo và các hướng dẫn dưới đây.

### ⚠️ BƯỚC 0: ĐỌC MẪU (TEMPLATE) TRƯỚC ⚠️

**QUAN TRỌNG: TRƯỚC KHI viết bất kỳ mã HTML nào:**

1. **Đọc** tệp `templates/viewer.html` bằng công cụ Read.
2. **Nghiên cứu** cấu trúc, kiểu dáng và nhận diện thương hiệu Anthropic một cách chính xác.
3. **Sử dụng tệp đó như là ĐIỂM BẮT ĐẦU THEO NGHĨA ĐEN** - không chỉ là cảm hứng.
4. **Giữ nguyên tất cả các phần CỐ ĐỊNH (FIXED) chính xác như được hiển thị** (tiêu đề, cấu trúc thanh bên, màu sắc/phông chữ Anthropic, các điều khiển hạt giống (seed controls), các nút hành động).
5. **Chỉ thay thế các phần BIẾN ĐỔI (VARIABLE)** được đánh dấu trong các bình luận của tệp (thuật toán, các tham số, giao diện người dùng cho các tham số).

**Cần tránh:**

- ❌ Tạo HTML từ đầu.
- ❌ Tự chế kiểu dáng hoặc phối màu tùy chỉnh.
- ❌ Sử dụng phông chữ hệ thống hoặc chủ đề tối (dark themes).
- ❌ Thay đổi cấu trúc thanh bên.

**Hãy tuân thủ các thực hành này:**

- ✅ Sao chép chính xác cấu trúc HTML của mẫu.
- ✅ Giữ nhận diện thương hiệu Anthropic (phông chữ Poppins/Lora, màu sắc tươi sáng, nền gradient).
- ✅ Duy trì bố cục thanh bên (Hạt giống (Seed) → Tham số (Parameters) → Màu sắc (Colors)? → Hành động (Actions)).
- ✅ Chỉ thay thế thuật toán p5.js và các điều khiển tham số.

Mẫu là nền tảng. Hãy xây dựng trên đó, đừng xây dựng lại nó.

---

Để tạo ra tác phẩm nghệ thuật tính toán chất lượng phòng trưng bày và đầy sức sống, hãy sử dụng triết lý thuật toán làm nền tảng.

### CÁC YÊU CẦU KỸ THUẬT

**Tính ngẫu nhiên có hạt giống (Mẫu Art Blocks)**:

```javascript
// LUÔN sử dụng một hạt giống để có tính tái lập
let seed = 12345; // hoặc hàm băm từ đầu vào của người dùng
randomSeed(seed);
noiseSeed(seed);
```

**Cấu trúc Tham số - TUÂN THỦ TRIẾT LÝ**:

Để thiết lập các tham số nảy sinh tự nhiên từ triết lý thuật toán, hãy cân nhắc: "Những đặc tính nào của hệ thống này có thể điều chỉnh được?"

```javascript
let params = {
  seed: 12345, // Luôn bao gồm hạt giống để có tính tái lập
  // colors
  // Thêm các tham số kiểm soát thuật toán của BẠN:
  // - Số lượng (bao nhiêu?)
  // - Quy mô (lớn bao nhiêu? nhanh bao nhiêu?)
  // - Xác suất (khả năng xảy ra thế nào?)
  // - Tỷ lệ (tỷ trọng thế nào?)
  // - Góc (hướng nào?)
  // - Ngưỡng (khi nào hành vi thay đổi?)
};
```

**Để thiết kế các tham số hiệu quả, hãy tập trung vào các đặc tính mà hệ thống cần có khả năng điều chỉnh, thay vì nghĩ theo kiểu "các loại mẫu mã".**

**Thuật toán Cốt lõi - THỂ HIỆN TRIẾT LÝ**:

**QUAN TRỌNG**: Triết lý thuật toán nên quyết định những gì cần xây dựng.

Để thể hiện triết lý thông qua mã nguồn, hãy tránh nghĩ "mình nên dùng mẫu nào?" mà hãy nghĩ "làm thế nào để thể hiện triết lý này qua mã nguồn?"

Nếu triết lý về **sự trỗi dậy hữu cơ (organic emergence)**, hãy cân nhắc sử dụng:

- Các thành phần tích tụ hoặc phát triển theo thời gian.
- Các quy trình ngẫu nhiên bị ràng buộc bởi các quy luật tự nhiên.
- Các vòng lặp phản hồi và các tương tác.

Nếu triết lý về **vẻ đẹp toán học (mathematical beauty)**, hãy cân nhắc sử dụng:

- Các mối quan hệ và tỷ lệ hình học.
- Các hàm lượng giác và hòa âm.
- Các tính toán chính xác tạo ra các mẫu không ngờ tới.

Nếu triết lý về **sự hỗn loạn có kiểm soát (controlled chaos)**, hãy cân nhắc sử dụng:

- Sự biến đổi ngẫu nhiên trong các ranh giới nghiêm ngặt.
- Sự phân nhánh và các bước chuyển pha.
- Trật tự trỗi dậy từ sự mất trật tự.

**Thuật toán tuôn chảy từ triết lý, không phải từ một danh mục các lựa chọn.**

Để hướng dẫn việc triển khai, hãy để bản chất khái niệm dẫn dắt các lựa chọn sáng tạo và nguyên bản. Hãy xây dựng thứ gì đó thể hiện được tầm nhìn cho yêu cầu cụ thể này.

**Thiết lập Canvas**: Cấu trúc p5.js tiêu chuẩn:

```javascript
function setup() {
  createCanvas(1200, 1200);
  // Khởi tạo hệ thống của bạn
}

function draw() {
  // Thuật toán tạo sinh của bạn
  // Có thể là tĩnh (noLoop) hoặc hoạt họa
}
```

### YÊU CẦU VỀ TAY NGHỀ

**QUAN TRỌNG**: Để đạt được sự bậc thầy, hãy tạo ra các thuật toán mang lại cảm giác như chúng nảy sinh qua vô số lần lặp bởi một nghệ sĩ tạo sinh bậc thầy. Hãy tinh chỉnh mọi tham số một cách cẩn thận. Đảm bảo mọi mẫu hoa văn đều nảy sinh có mục đích. Đây KHÔNG phải là nhiễu ngẫu nhiên - đây là SỰ HỖN LOẠN CÓ KIỂM SOÁT được trau chuốt thông qua chuyên môn sâu sắc.

- **Sự cân bằng**: Độ phức tạp mà không gây nhiễu thị giác, trật tự mà không cứng nhắc.
- **Sự hài hòa màu sắc**: Các bảng màu được cân nhắc kỹ lưỡng, không phải các giá trị RGB ngẫu nhiên.
- **Bố cục**: Ngay cả trong sự ngẫu nhiên, hãy duy trì hệ thống phân cấp thị giác và dòng chảy của bố cục.
- **Hiệu năng**: Thực thi mượt mà, tối ưu hóa cho thời gian thực nếu có hoạt họa.
- **Tính tái lập**: Cùng một hạt giống LUÔN tạo ra đầu ra giống hệt nhau.

### ĐỊNH DẠNG ĐẦU RA

Đầu ra:

1. **Triết lý Thuật toán** - Dưới dạng markdown hoặc văn bản giải thích thẩm mỹ tạo sinh.
2. **Một Artifact HTML Duy nhất** - Nghệ thuật tạo sinh tương tác độc lập được xây dựng từ `templates/viewer.html` (xem BƯỚC 0 và phần tiếp theo).

Artifact HTML chứa tất cả mọi thứ: p5.js (từ CDN), thuật toán, các điều khiển tham số và giao diện người dùng - tất cả trong một tệp hoạt động ngay lập tức trong các artifact của claude.ai hoặc bất kỳ trình duyệt nào. Hãy bắt đầu từ tệp mẫu, đừng bắt đầu từ con số không.

---

## TẠO ARTIFACT TƯƠNG TÁC

**NHẮC LẠI: Tệp `templates/viewer.html` đáng lẽ đã phải được đọc (xem BƯỚC 0). Hãy sử dụng tệp đó làm điểm bắt đầu.**

Để cho phép khám phá nghệ thuật tạo sinh, hãy tạo một artifact HTML duy nhất, độc lập. Đảm bảo artifact này hoạt động ngay lập tức trong claude.ai hoặc bất kỳ trình duyệt nào - không cần thiết lập. Nhúng tất cả mọi thứ vào bên trong.

### QUAN TRỌNG: NHỮNG GÌ CỐ ĐỊNH VS BIẾN ĐỔI

Tệp `templates/viewer.html` là nền tảng. Nó chứa cấu trúc và kiểu dáng chính xác cần thiết.

**CỐ ĐỊNH (luôn bao gồm chính xác như được hiển thị):**

- Cấu trúc bố cục (tiêu đề, thanh bên, vùng canvas chính).
- Nhận diện thương hiệu Anthropic (màu sắc giao diện người dùng, phông chữ, gradient).
- Phần Hạt giống (Seed) ở thanh bên:
  - Hiển thị hạt giống.
  - Các nút Trước (Previous)/Sau (Next).
  - Nút Ngẫu nhiên (Random).
  - Nhập hạt giống để nhảy tới (Jump to seed) + Nút Đi (Go).
- Phần Hành động (Actions) ở thanh bên:
  - Nút Tạo lại (Regenerate).
  - Nút Đặt lại (Reset).

**BIẾN ĐỔI (tùy chỉnh cho mỗi tác phẩm):**

- Toàn bộ thuật toán p5.js (setup/draw/các lớp).
- Đối tượng tham số (params) (định nghĩa những gì tác phẩm cần).
- Phần Tham số (Parameters) ở thanh bên:
  - Số lượng các điều khiển tham số.
  - Tên các tham số.
  - Các giá trị min/max/step cho thanh trượt.
  - Các loại điều khiển (thanh trượt, ô nhập dữ liệu, v.v.).
- Phần Màu sắc (Colors) (tùy chọn):
  - Một số tác phẩm cần bộ chọn màu.
  - Một số có thể dùng màu cố định.
  - Một số có thể là đơn sắc (không cần điều khiển màu).
  - Quyết định dựa trên nhu cầu của tác phẩm.

**Mỗi tác phẩm nghệ thuật nên có các tham số và thuật toán duy nhất!** Các phần cố định cung cấp trải nghiệm người dùng nhất quán - mọi thứ khác thể hiện tầm nhìn độc đáo.

### CÁC TÍNH NĂNG BẮT BUỘC

**1. Điều khiển Tham số**

- Thanh trượt cho các tham số dạng số (số lượng hạt, quy mô nhiễu, tốc độ, v.v.).
- Bộ chọn màu cho màu sắc bảng màu.
- Cập nhật thời gian thực khi các tham số thay đổi.
- Nút đặt lại để khôi phục giá trị mặc định.

**2. Điều hướng Hạt giống**

- Hiển thị số hạt giống hiện tại.
- Các nút "Trước" và "Sau" để chuyển qua các hạt giống.
- Nút "Ngẫu nhiên" cho hạt giống ngẫu nhiên.
- Ô nhập để nhảy đến một hạt giống cụ thể.
- Tạo ra 100 biến thể khi được yêu cầu (các hạt giống từ 1-100).

**3. Cấu trúc Artifact Duy nhất**

```html
<!DOCTYPE html>
<html>
  <head>
    <!-- p5.js từ CDN - luôn sẵn có -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/p5.js/1.7.0/p5.min.js"></script>
    <style>
      /* Tất cả kiểu dáng nhúng trực tiếp - sạch sẽ, tối giản */
      /* Canvas ở trên, các điều khiển ở dưới */
    </style>
  </head>
  <body>
    <div id="canvas-container"></div>
    <div id="controls">
      <!-- Tất cả các điều khiển tham số -->
    </div>
    <script>
      // TẤT CẢ mã p5.js nhúng trực tiếp ở đây
      // Các đối tượng tham số, các lớp, các hàm
      // setup() và draw()
      // Các trình xử lý giao diện người dùng (UI handlers)
      // Mọi thứ đều độc lập
    </script>
  </body>
</html>
```

**QUAN TRỌNG**: Đây là một artifact duy nhất. Không có tệp bên ngoài, không có import (ngoại trừ p5.js CDN). Mọi thứ nhúng trực tiếp.

**4. Chi tiết Triển khai - XÂY DỰNG THANH BÊN**

Cấu trúc thanh bên:

**1. Hạt giống (CỐ ĐỊNH)** - Luôn bao gồm chính xác như được hiển thị:

- Hiển thị hạt giống.
- Các nút Trước/Sau/Ngẫu nhiên/Nhảy tới.

**2. Tham số (BIẾN ĐỔI)** - Tạo các điều khiển cho tác phẩm:

```html
<div class="control-group">
  <label>Tên Tham số</label>
  <input
    type="range"
    id="param"
    min="..."
    max="..."
    step="..."
    value="..."
    oninput="updateParam('param', this.value)"
  />
  <span class="value-display" id="param-value">...</span>
</div>
```

Thêm bao nhiêu thẻ div `control-group` tùy theo số lượng tham số.

**3. Màu sắc (TÙY CHỌN/BIẾN ĐỔI)** - Bao gồm nếu tác phẩm cần các màu có thể điều chỉnh:

- Thêm các bộ chọn màu nếu người dùng nên kiểm soát bảng màu.
- Bỏ qua phần này nếu tác phẩm dùng màu cố định.
- Bỏ qua nếu tác phẩm là đơn sắc.

**4. Hành động (CỐ ĐỊNH)** - Luôn bao gồm chính xác như được hiển thị:

- Nút Tạo lại.
- Nút Đặt lại.
- Nút Tải PNG.

**Yêu cầu**:

- Các điều khiển hạt giống phải hoạt động (trước/sau/ngẫu nhiên/nhảy tới/hiển thị).
- Tất cả các tham số phải có điều khiển giao diện người dùng.
- Các nút Tạo lại, Đặt lại, Tải về phải hoạt động.
- Giữ nhận diện thương hiệu Anthropic (kiểu dáng giao diện người dùng, không phải màu sắc của tác phẩm).

### SỬ DỤNG ARTIFACT

Artifact HTML hoạt động ngay lập tức:

1. **Trong claude.ai**: Hiển thị dưới dạng artifact tương tác - chạy ngay lập tức.
2. **Dưới dạng tệp**: Lưu lại và mở trong bất kỳ trình duyệt nào - không cần máy chủ.
3. **Chia sẻ**: Gửi tệp HTML - nó hoàn toàn độc lập.

---

## CÁC BIẾN THỂ & KHÁM PHÁ

Artifact mặc định bao gồm điều hướng hạt giống (các nút trước/sau/ngẫu nhiên), cho phép người dùng khám phá các biến thể mà không cần tạo nhiều tệp. Nếu người dùng muốn làm nổi bật các biến thể cụ thể:

- Bao gồm các thiết lập sẵn cho hạt giống (các nút cho "Biến thể 1: Hạt giống 42", "Biến thể 2: Hạt giống 127", v.v.).
- Thêm một "Chế độ Thư viện" (Gallery Mode) hiển thị các hình thu nhỏ của nhiều hạt giống cạnh nhau.
- Tất cả nằm trong cùng một artifact duy nhất.

Điều này giống như việc tạo ra một loạt bản in từ cùng một khuôn in - thuật toán là nhất quán, nhưng mỗi hạt giống tiết lộ các khía cạnh khác nhau trong tiềm năng của nó. Bản chất tương tác có nghĩa là người dùng tự khám phá những sở thích của riêng mình bằng cách khám phá không gian hạt giống.

---

## QUY TRÌNH SÁNG TẠO

**Yêu cầu người dùng** → **Triết lý thuật toán** → **Triển khai**

Mỗi yêu cầu là duy nhất. Quy trình bao gồm:

1. **Diễn giải ý định của người dùng** - Họ đang tìm kiếm thẩm mỹ như thế nào?
2. **Tạo một triết lý thuật toán** (4-6 đoạn văn) mô tả cách tiếp cận tính toán.
3. **Thể hiện bằng mã nguồn** - Xây dựng thuật toán diễn đạt triết lý này.
4. **Thiết kế các tham số phù hợp** - Những gì nên có khả năng điều chỉnh?
5. **Xây dựng các điều khiển UI tương ứng** - Các thanh trượt/ô nhập cho các tham số đó.

**Những thứ không đổi**:

- Nhận diện thương hiệu Anthropic (màu sắc, phông chữ, bố cục).
- Điều hướng hạt giống (luôn luôn hiện diện).
- Artifact HTML độc lập.

**Mọi thứ khác đều biến đổi**:

- Chính thuật toán đó.
- Các tham số.
- Các điều khiển giao diện người dùng.
- Kết quả thị giác.

Để đạt được kết quả tốt nhất, hãy tin tưởng vào sự sáng tạo và để triết lý dẫn dắt việc triển khai.

---

## TÀI NGUYÊN

Kỹ năng này bao gồm các mẫu và tài liệu hữu ích:

- **templates/viewer.html**: ĐIỂM BẮT ĐẦU BẮT BUỘC cho tất cả các artifact HTML.
  - Đây là nền tảng - chứa cấu trúc chính xác và nhận diện thương hiệu Anthropic.
  - **Giữ nguyên**: Cấu trúc bố cục, tổ chức thanh bên, màu sắc/phông chữ Anthropic, các điều khiển hạt giống, các nút hành động.
  - **Thay thế**: Thuật toán p5.js, định nghĩa tham số và các điều khiển UI trong phần Tham số.
  - Các bình luận chi tiết trong tệp đánh dấu chính xác những gì cần giữ lại và những gì cần thay thế.

- **templates/generator_template.js**: Tham chiếu cho các thực hành tốt nhất về p5.js và các nguyên tắc cấu trúc mã.
  - Cho biết cách tổ chức các tham số, sử dụng tính ngẫu nhiên có hạt giống, cấu trúc các lớp.
  - KHÔNG phải là một menu các mẫu - hãy sử dụng các nguyên tắc này để xây dựng các thuật toán độc đáo.
  - Nhúng các thuật toán trực tiếp vào artifact HTML (không tạo thêm tệp .js riêng biệt).

**Nhắc nhở quan trọng**:

- **Mẫu (template) là ĐIỂM BẮT ĐẦU**, không phải cảm hứng.
- **Thuật toán là nơi để tạo ra** thứ gì đó độc đáo.
- Đừng sao chép ví dụ về trường dòng chảy (flow field) - hãy xây dựng những gì triết lý yêu cầu.
- Nhưng HÃY giữ nguyên cấu trúc UI và nhận diện thương hiệu Anthropic từ mẫu.
