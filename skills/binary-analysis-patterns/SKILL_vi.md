---
name: binary-analysis-patterns
description: Nắm vững các mẫu phân tích nhị phân bao gồm disassembly, decompilation, phân tích luồng điều khiển, và nhận dạng mẫu mã. Sử dụng khi phân tích các tệp thực thi, hiểu mã đã biên dịch, hoặc thực hiện phân tích tĩnh trên mã nhị phân.
---

# Các Mẫu Phân tích Nhị phân (Binary Analysis Patterns)

Các mẫu và kỹ thuật toàn diện để phân tích mã nhị phân đã biên dịch, hiểu mã assembly, và tái cấu trúc logic chương trình.

## Sử dụng kỹ năng này khi

- Làm việc trên các nhiệm vụ hoặc quy trình phân tích nhị phân
- Cần hướng dẫn, thực hành tốt nhất, hoặc danh sách kiểm tra cho các mẫu phân tích nhị phân

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến các mẫu phân tích nhị phân
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc, và đầu vào cần thiết.
- Áp dụng các thực hành tốt nhất liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần ví dụ chi tiết, mở `resources/implementation-playbook.md`.

## Các Nguyên tắc Cơ bản về Disassembly

### Mẫu Lệnh x86-64

#### Function Prologue/Epilogue (Mở đầu/Kết thúc Hàm)

```asm
; Standard prologue
push rbp           ; Lưu base pointer
mov rbp, rsp       ; Thiết lập stack frame
sub rsp, 0x20      ; Cấp phát biến cục bộ

; Leaf function (không gọi hàm khác)
; Có thể bỏ qua thiết lập frame pointer
sub rsp, 0x18      ; Chỉ cấp phát biến cục bộ

; Standard epilogue
mov rsp, rbp       ; Khôi phục stack pointer
pop rbp            ; Khôi phục base pointer
ret

; Leave instruction (tương đương)
leave              ; mov rsp, rbp; pop rbp
ret
```

#### Quy ước Gọi (Calling Conventions)

**System V AMD64 (Linux, macOS)**

```asm
; Đối số: RDI, RSI, RDX, RCX, R8, R9, sau đó là stack
; Trả về: RAX (và RDX cho 128-bit)
; Caller-saved: RAX, RCX, RDX, RSI, RDI, R8-R11
; Callee-saved: RBX, RBP, R12-R15

; Ví dụ: func(a, b, c, d, e, f, g)
mov rdi, [a]       ; Đối số 1
mov rsi, [b]       ; Đối số 2
mov rdx, [c]       ; Đối số 3
mov rcx, [d]       ; Đối số 4
mov r8, [e]        ; Đối số 5
mov r9, [f]        ; Đối số 6
push [g]           ; Đối số 7 trên stack
call func
```

**Microsoft x64 (Windows)**

```asm
; Đối số: RCX, RDX, R8, R9, sau đó là stack
; Shadow space: 32 bytes dành riêng trên stack
; Trả về: RAX

; Ví dụ: func(a, b, c, d, e)
sub rsp, 0x28      ; Shadow space + alignment
mov rcx, [a]       ; Đối số 1
mov rdx, [b]       ; Đối số 2
mov r8, [c]        ; Đối số 3
mov r9, [d]        ; Đối số 4
mov [rsp+0x20], [e] ; Đối số 5 trên stack
call func
add rsp, 0x28
```

### Các Mẫu Assembly ARM

#### Quy ước Gọi ARM64 (AArch64)

```asm
; Đối số: X0-X7
; Trả về: X0 (và X1 cho 128-bit)
; Frame pointer: X29
; Link register: X30

; Function prologue
stp x29, x30, [sp, #-16]!  ; Lưu FP và LR
mov x29, sp                 ; Thiết lập frame pointer

; Function epilogue
ldp x29, x30, [sp], #16    ; Khôi phục FP và LR
ret
```

#### Quy ước Gọi ARM32

```asm
; Đối số: R0-R3, sau đó là stack
; Trả về: R0 (và R1 cho 64-bit)
; Link register: LR (R14)

; Function prologue
push {fp, lr}
add fp, sp, #4

; Function epilogue
pop {fp, pc}    ; Trả về bằng cách pop PC
```

## Các Mẫu Luồng Điều khiển (Control Flow Patterns)

### Rẽ nhánh Có điều kiện (Conditional Branches)

```asm
; if (a == b)
cmp eax, ebx
jne skip_block
; ... if body ...
skip_block:

; if (a < b) - có dấu (signed)
cmp eax, ebx
jge skip_block    ; Nhảy nếu lớn hơn hoặc bằng
; ... if body ...
skip_block:

; if (a < b) - không dấu (unsigned)
cmp eax, ebx
jae skip_block    ; Nhảy nếu trên hoặc bằng
; ... if body ...
skip_block:
```

### Các Mẫu Vòng lặp (Loop Patterns)

```asm
; for (int i = 0; i < n; i++)
xor ecx, ecx           ; i = 0
loop_start:
cmp ecx, [n]           ; i < n
jge loop_end
; ... loop body ...
inc ecx                ; i++
jmp loop_start
loop_end:

; while (condition)
jmp loop_check
loop_body:
; ... body ...
loop_check:
cmp eax, ebx
jl loop_body

; do-while
loop_body:
; ... body ...
cmp eax, ebx
jl loop_body
```

### Các Mẫu Lệnh Switch

```asm
; Jump table pattern
mov eax, [switch_var]
cmp eax, max_case
ja default_case
jmp [jump_table + eax*8]

; So sánh tuần tự (switch nhỏ)
cmp eax, 1
je case_1
cmp eax, 2
je case_2
cmp eax, 3
je case_3
jmp default_case
```

## Các Mẫu Cấu trúc Dữ liệu

### Truy cập Mảng

```asm
; array[i] - phần tử 4-byte
mov eax, [rbx + rcx*4]        ; rbx=base, rcx=index

; array[i] - phần tử 8-byte
mov rax, [rbx + rcx*8]

; Mảng đa chiều[i][j]
; arr[i][j] = base + (i * cols + j) * element_size
imul eax, [cols]
add eax, [j]
mov edx, [rbx + rax*4]
```

### Truy cập Cấu trúc (Structure Access)

```c
struct Example {
    int a;      // offset 0
    char b;     // offset 4
    // padding  // offset 5-7
    long c;     // offset 8
    short d;    // offset 16
};
```

```asm
; Truy cập các trường struct
mov rdi, [struct_ptr]
mov eax, [rdi]         ; s->a (offset 0)
movzx eax, byte [rdi+4] ; s->b (offset 4)
mov rax, [rdi+8]       ; s->c (offset 8)
movzx eax, word [rdi+16] ; s->d (offset 16)
```

### Duyệt Danh sách Liên kết

```asm
; while (node != NULL)
list_loop:
test rdi, rdi          ; node == NULL?
jz list_done
; ... xử lý node ...
mov rdi, [rdi+8]       ; node = node->next (giả sử next ở offset 8)
jmp list_loop
list_done:
```

## Các Mẫu Mã Phổ biến

### Các Hoạt động Chuỗi

```asm
; strlen pattern
xor ecx, ecx
strlen_loop:
cmp byte [rdi + rcx], 0
je strlen_done
inc ecx
jmp strlen_loop
strlen_done:
; ecx chứa độ dài

; strcpy pattern
strcpy_loop:
mov al, [rsi]
mov [rdi], al
test al, al
jz strcpy_done
inc rsi
inc rdi
jmp strcpy_loop
strcpy_done:

; memcpy sử dụng rep movsb
mov rdi, dest
mov rsi, src
mov rcx, count
rep movsb
```

### Các Mẫu Số học

```asm
; Nhân với hằng số
; x * 3
lea eax, [rax + rax*2]

; x * 5
lea eax, [rax + rax*4]

; x * 10
lea eax, [rax + rax*4]  ; x * 5
add eax, eax            ; * 2

; Chia cho lũy thừa của 2 (có dấu)
mov eax, [x]
cdq                     ; Mở rộng dấu sang EDX:EAX
and edx, 7              ; Để chia cho 8
add eax, edx            ; Điều chỉnh cho số âm
sar eax, 3              ; Dịch phải số học

; Modulo lũy thừa của 2
and eax, 7              ; x % 8
```

### Thao tác Bit

```asm
; Kiểm tra bit cụ thể
test eax, 0x80          ; Kiểm tra bit 7
jnz bit_set

; Set bit
or eax, 0x10            ; Set bit 4

; Clear bit
and eax, ~0x10          ; Clear bit 4

; Toggle bit
xor eax, 0x10           ; Toggle bit 4

; Đếm số 0 dẫn đầu
bsr eax, ecx            ; Bit scan reverse
xor eax, 31             ; Chuyển đổi sang số 0 dẫn đầu

; Population count (popcnt)
popcnt eax, ecx         ; Đếm số bit set
```

## Các Mẫu Decompilation

### Khôi phục Biến (Variable Recovery)

```asm
; Biến cục bộ tại rbp-8
mov qword [rbp-8], rax  ; Lưu vào cục bộ
mov rax, [rbp-8]        ; Tải từ cục bộ

; Mảng được cấp phát trên stack
lea rax, [rbp-0x40]     ; Mảng bắt đầu tại rbp-0x40
mov [rax], edx          ; array[0] = value
mov [rax+4], ecx        ; array[1] = value
```

### Khôi phục Chữ ký Hàm

```asm
; Xác định tham số bằng cách sử dụng thanh ghi
func:
    ; rdi được sử dụng làm tham số đầu tiên (System V)
    mov [rbp-8], rdi    ; Lưu tham số vào cục bộ
    ; rsi được sử dụng làm tham số thứ hai
    mov [rbp-16], rsi
    ; Xác định trả về bởi RAX ở cuối
    mov rax, [result]
    ret
```

### Khôi phục Kiểu (Type Recovery)

```asm
; Các hoạt động 1-byte gợi ý char/bool
movzx eax, byte [rdi]   ; Zero-extend byte
movsx eax, byte [rdi]   ; Sign-extend byte

; Các hoạt động 2-byte gợi ý short
movzx eax, word [rdi]
movsx eax, word [rdi]

; Các hoạt động 4-byte gợi ý int/float
mov eax, [rdi]
movss xmm0, [rdi]       ; Float

; Các hoạt động 8-byte gợi ý long/double/pointer
mov rax, [rdi]
movsd xmm0, [rdi]       ; Double
```

## Mẹo Phân tích Ghidra

### Cải thiện Decompilation

```java
// Trong kịch bản Ghidra
// Sửa chữ ký hàm
Function func = getFunctionAt(toAddr(0x401000));
func.setReturnType(IntegerDataType.dataType, SourceType.USER_DEFINED);

// Tạo kiểu cấu trúc
StructureDataType struct = new StructureDataType("MyStruct", 0);
struct.add(IntegerDataType.dataType, "field_a", null);
struct.add(PointerDataType.dataType, "next", null);

// Áp dụng cho bộ nhớ
createData(toAddr(0x601000), struct);
```

### Kịch bản Khớp Mẫu

```python
# Tìm tất cả các cuộc gọi đến các hàm nguy hiểm
for func in currentProgram.getFunctionManager().getFunctions(True):
    for ref in getReferencesTo(func.getEntryPoint()):
        if func.getName() in ["strcpy", "sprintf", "gets"]:
            print(f"Dangerous call at {ref.getFromAddress()}")
```

## Các Mẫu IDA Pro

### Phân tích IDAPython

```python
import idaapi
import idautils
import idc

# Tìm tất cả các cuộc gọi hàm
def find_calls(func_name):
    for func_ea in idautils.Functions():
        for head in idautils.Heads(func_ea, idc.find_func_end(func_ea)):
            if idc.print_insn_mnem(head) == "call":
                target = idc.get_operand_value(head, 0)
                if idc.get_func_name(target) == func_name:
                    print(f"Call to {func_name} at {hex(head)}")

# Đổi tên hàm dựa trên chuỗi
def auto_rename():
    for s in idautils.Strings():
        for xref in idautils.XrefsTo(s.ea):
            func = idaapi.get_func(xref.frm)
            if func and "sub_" in idc.get_func_name(func.start_ea):
                # Sử dụng chuỗi làm gợi ý để đặt tên
                pass
```

## Thực hành Tốt nhất

### Quy trình Phân tích

1. **Phân loại ban đầu**: Loại tệp, kiến trúc, imports/exports
2. **Phân tích chuỗi**: Xác định các chuỗi thú vị, thông báo lỗi
3. **Nhận dạng hàm**: Điểm nhập, exports, tham chiếu chéo
4. **Ánh xạ luồng điều khiển**: Hiểu cấu trúc chương trình
5. **Khôi phục cấu trúc dữ liệu**: Xác định structs, arrays, globals
6. **Nhận dạng thuật toán**: Crypto, hashing, nén
7. **Tài liệu hóa**: Chú thích, đổi tên ký hiệu, định nghĩa kiểu

### Các Cạm bẫy Phổ biến

- **Di vật của bộ tối ưu hóa (Optimizer artifacts)**: Mã có thể không khớp với cấu trúc mã nguồn
- **Hàm nội tuyến (Inline functions)**: Các hàm có thể được mở rộng nội tuyến
- **Tối ưu hóa gọi đuôi (Tail call optimization)**: `jmp` thay vì `call` + `ret`
- **Mã chết (Dead code)**: Mã không thể truy cập từ tối ưu hóa
- **Mã độc lập vị trí (Position-independent code)**: Địa chỉ tương đối RIP
