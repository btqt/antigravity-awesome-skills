# Sổ tay Triển khai Các Kỹ thuật Chống Dịch ngược

Tệp này chứa các mẫu chi tiết, danh sách kiểm tra và các mẫu mã được tham chiếu bởi kỹ năng.

# Các Kỹ thuật Chống Dịch ngược

Hiểu các cơ chế bảo vệ gặp phải trong quá trình phân tích phần mềm được ủy quyền, nghiên cứu bảo mật và phân tích phần mềm độc hại. Kiến thức này giúp các nhà phân tích vượt qua các biện pháp bảo vệ để hoàn thành các nhiệm vụ phân tích hợp pháp.

## Các Kỹ thuật Chống Gỡ lỗi (Anti-Debugging Techniques)

### Chống Gỡ lỗi trên Windows

#### Phát hiện dựa trên API

```c
// IsDebuggerPresent
if (IsDebuggerPresent()) {
    exit(1);
}

// CheckRemoteDebuggerPresent
BOOL debugged = FALSE;
CheckRemoteDebuggerPresent(GetCurrentProcess(), &debugged);
if (debugged) exit(1);

// NtQueryInformationProcess
typedef NTSTATUS (NTAPI *pNtQueryInformationProcess)(
    HANDLE, PROCESSINFOCLASS, PVOID, ULONG, PULONG);

DWORD debugPort = 0;
NtQueryInformationProcess(
    GetCurrentProcess(),
    ProcessDebugPort,        // 7
    &debugPort,
    sizeof(debugPort),
    NULL
);
if (debugPort != 0) exit(1);

// Debug flags
DWORD debugFlags = 0;
NtQueryInformationProcess(
    GetCurrentProcess(),
    ProcessDebugFlags,       // 0x1F
    &debugFlags,
    sizeof(debugFlags),
    NULL
);
if (debugFlags == 0) exit(1);  // 0 means being debugged
```

**Các Cách tiếp cận Vượt qua:**

```python
# x64dbg: ScyllaHide plugin
# Vá các kiểm tra chống gỡ lỗi phổ biến

# Vá thủ công trong trình gỡ lỗi:
# - Đặt giá trị trả về của IsDebuggerPresent thành 0
# - Vá PEB.BeingDebugged thành 0
# - Hook NtQueryInformationProcess

# IDAPython: Vá các kiểm tra
ida_bytes.patch_byte(check_addr, 0x90)  # NOP
```

#### Phát hiện dựa trên PEB

```c
// Truy cập PEB trực tiếp
#ifdef _WIN64
    PPEB peb = (PPEB)__readgsqword(0x60);
#else
    PPEB peb = (PPEB)__readfsdword(0x30);
#endif

// Cờ BeingDebugged
if (peb->BeingDebugged) exit(1);

// NtGlobalFlag
// Debugged: 0x70 (FLG_HEAP_ENABLE_TAIL_CHECK |
//                 FLG_HEAP_ENABLE_FREE_CHECK |
//                 FLG_HEAP_VALIDATE_PARAMETERS)
if (peb->NtGlobalFlag & 0x70) exit(1);

// Các cờ Heap
PDWORD heapFlags = (PDWORD)((PBYTE)peb->ProcessHeap + 0x70);
if (*heapFlags & 0x50000062) exit(1);
```

**Các Cách tiếp cận Vượt qua:**

```assembly
; Trong trình gỡ lỗi, sửa đổi PEB trực tiếp
; x64dbg: dump tại gs:[60] (x64) hoặc fs:[30] (x86)
; Đặt BeingDebugged (offset 2) thành 0
; Xóa NtGlobalFlag (offset 0xBC cho x64)
```

#### Phát hiện dựa trên Thời gian

```c
// RDTSC timing
uint64_t start = __rdtsc();
// ... một số mã ...
uint64_t end = __rdtsc();
if ((end - start) > THRESHOLD) exit(1);

// QueryPerformanceCounter
LARGE_INTEGER start, end, freq;
QueryPerformanceFrequency(&freq);
QueryPerformanceCounter(&start);
// ... mã ...
QueryPerformanceCounter(&end);
double elapsed = (double)(end.QuadPart - start.QuadPart) / freq.QuadPart;
if (elapsed > 0.1) exit(1);  // Quá chậm = có trình gỡ lỗi

// GetTickCount
DWORD start = GetTickCount();
// ... mã ...
if (GetTickCount() - start > 1000) exit(1);
```

**Các Cách tiếp cận Vượt qua:**

```
- Sử dụng điểm ngắt phần cứng (hardware breakpoints) thay vì phần mềm
- Vá các kiểm tra thời gian
- Sử dụng VM với thời gian được kiểm soát
- Hook các API thời gian để trả về các giá trị nhất quán
```

#### Phát hiện dựa trên Ngoại lệ (Exception-Based Detection)

```c
// Phát hiện dựa trên SEH
__try {
    __asm { int 3 }  // Điểm ngắt phần mềm
}
__except(EXCEPTION_EXECUTE_HANDLER) {
    // Thực thi bình thường: ngoại lệ đã bắt được
    return;
}
// Trình gỡ lỗi đã "nuốt" ngoại lệ
exit(1);

// Phát hiện dựa trên VEH
LONG CALLBACK VectoredHandler(PEXCEPTION_POINTERS ep) {
    if (ep->ExceptionRecord->ExceptionCode == EXCEPTION_BREAKPOINT) {
        ep->ContextRecord->Rip++;  // Bỏ qua INT3
        return EXCEPTION_CONTINUE_EXECUTION;
    }
    return EXCEPTION_CONTINUE_SEARCH;
}
```

### Chống Gỡ lỗi trên Linux

```c
// ptrace self-trace
if (ptrace(PTRACE_TRACEME, 0, NULL, NULL) == -1) {
    // Đang bị theo dõi
    exit(1);
}

// /proc/self/status
FILE *f = fopen("/proc/self/status", "r");
char line[256];
while (fgets(line, sizeof(line), f)) {
    if (strncmp(line, "TracerPid:", 10) == 0) {
        int tracer_pid = atoi(line + 10);
        if (tracer_pid != 0) exit(1);
    }
}

// Kiểm tra quy trình cha
if (getppid() != 1 && strcmp(get_process_name(getppid()), "bash") != 0) {
    // Cha bất thường (có thể là trình gỡ lỗi)
}
```

**Các Cách tiếp cận Vượt qua:**

```bash
# LD_PRELOAD để hook ptrace
# Biên dịch: gcc -shared -fPIC -o hook.so hook.c
long ptrace(int request, ...) {
    return 0;  // Luôn thành công
}

# Sử dụng
LD_PRELOAD=./hook.so ./target
```

## Phát hiện Chống Máy ảo (Anti-VM Detection)

### Lấy dấu vân tay Phần cứng

```c
// Phát hiện dựa trên CPUID
int cpuid_info[4];
__cpuid(cpuid_info, 1);
// Kiểm tra bit hypervisor (bit 31 của ECX)
if (cpuid_info[2] & (1 << 31)) {
    // Đang chạy trong hypervisor
}

// Chuỗi thương hiệu CPUID
__cpuid(cpuid_info, 0x40000000);
char vendor[13] = {0};
memcpy(vendor, &cpuid_info[1], 12);
// "VMwareVMware", "Microsoft Hv", "KVMKVMKVM", "VBoxVBoxVBox"

// Tiền tố địa chỉ MAC
// VMware: 00:0C:29, 00:50:56
// VirtualBox: 08:00:27
// Hyper-V: 00:15:5D
```

### Phát hiện Registry/Tệp

```c
// Các khóa Windows registry
// HKLM\SOFTWARE\VMware, Inc.\VMware Tools
// HKLM\SOFTWARE\Oracle\VirtualBox Guest Additions
// HKLM\HARDWARE\ACPI\DSDT\VBOX__

// Các tệp
// C:\Windows\System32\drivers\vmmouse.sys
// C:\Windows\System32\drivers\vmhgfs.sys
// C:\Windows\System32\drivers\VBoxMouse.sys

// Các quy trình
// vmtoolsd.exe, vmwaretray.exe
// VBoxService.exe, VBoxTray.exe
```

### Phát hiện VM dựa trên Thời gian

```c
// Thoát VM (VM exits) gây ra sự bất thường về thời gian
uint64_t start = __rdtsc();
__cpuid(cpuid_info, 0);  // Gây ra VM exit
uint64_t end = __rdtsc();
if ((end - start) > 500) {
    // Có khả năng trong VM (CPUID mất nhiều thời gian hơn)
}
```

**Các Cách tiếp cận Vượt qua:**

```
- Sử dụng môi trường phân tích bare-metal
- Gia cố VM (xóa các công cụ khách, thay đổi MAC)
- Vá mã phát hiện
- Sử dụng các VM phân tích chuyên dụng (FLARE-VM)
```

## Làm rối Mã (Code Obfuscation)

### Làm rối Luồng Điều khiển

#### Làm phẳng Luồng Điều khiển (Control Flow Flattening)

```c
// Bản gốc
if (cond) {
    func_a();
} else {
    func_b();
}
func_c();

// Đã làm phẳng
int state = 0;
while (1) {
    switch (state) {
        case 0:
            state = cond ? 1 : 2;
            break;
        case 1:
            func_a();
            state = 3;
            break;
        case 2:
            func_b();
            state = 3;
            break;
        case 3:
            func_c();
            return;
    }
}
```

**Cách tiếp cận Phân tích:**

- Xác định biến trạng thái
- Ánh xạ các chuyển đổi trạng thái
- Tái tạo lại luồng gốc
- Công cụ: D-810 (IDA), SATURN

#### Vị từ Mờ (Opaque Predicates)

```c
// Luôn đúng, nhưng phức tạp để phân tích
int x = rand();
if ((x * x) >= 0) {  // Luôn đúng
    real_code();
} else {
    junk_code();  // Mã chết
}

// Luôn sai
if ((x * (x + 1)) % 2 == 1) {  // Tích của hai số liên tiếp = chẵn
    junk_code();
}
```

**Cách tiếp cận Phân tích:**

- Xác định các biểu thức hằng
- Thực thi ký hiệu (symbolic execution) để chứng minh vị từ
- Khớp mẫu cho các vị từ mờ đã biết

### Làm rối Dữ liệu

#### Mã hóa Chuỗi

```c
// Mã hóa XOR
char decrypt_string(char *enc, int len, char key) {
    char *dec = malloc(len + 1);
    for (int i = 0; i < len; i++) {
        dec[i] = enc[i] ^ key;
    }
    dec[len] = 0;
    return dec;
}

// Chuỗi Stack
char url[20];
url[0] = 'h'; url[1] = 't'; url[2] = 't'; url[3] = 'p';
url[4] = ':'; url[5] = '/'; url[6] = '/';
// ...
```

**Cách tiếp cận Phân tích:**

```python
# FLOSS để giải mã chuỗi tự động
floss malware.exe

# Giải mã chuỗi IDAPython
def decrypt_xor(ea, length, key):
    result = ""
    for i in range(length):
        byte = ida_bytes.get_byte(ea + i)
        result += chr(byte ^ key)
    return result
```

#### Làm rối API

```c
// Phân giải API động
typedef HANDLE (WINAPI *pCreateFileW)(LPCWSTR, DWORD, DWORD,
    LPSECURITY_ATTRIBUTES, DWORD, DWORD, HANDLE);

HMODULE kernel32 = LoadLibraryA("kernel32.dll");
pCreateFileW myCreateFile = (pCreateFileW)GetProcAddress(
    kernel32, "CreateFileW");

// Băm API (API hashing)
DWORD hash_api(char *name) {
    DWORD hash = 0;
    while (*name) {
        hash = ((hash >> 13) | (hash << 19)) + *name++;
    }
    return hash;
}
// Giải quyết bằng cách so sánh băm thay vì chuỗi
```

**Cách tiếp cận Phân tích:**

- Xác định thuật toán băm
- Xây dựng cơ sở dữ liệu băm của các API đã biết
- Sử dụng plugin HashDB cho IDA
- Phân tích động để giải quyết khi chạy

### Làm rối Mức Lệnh

#### Chèn Mã Chết

```asm
; Bản gốc
mov eax, 1

; Với mã chết
push ebx           ; Chết
mov eax, 1
pop ebx            ; Chết
xor ecx, ecx       ; Chết
add ecx, ecx       ; Chết
```

#### Thay thế Lệnh

```asm
; Bản gốc: xor eax, eax (đặt về 0)
; Thay thế:
sub eax, eax
mov eax, 0
and eax, 0
lea eax, [0]

; Bản gốc: mov eax, 1
; Thay thế:
xor eax, eax
inc eax

push 1
pop eax
```

## Đóng gói và Mã hóa

### Các Packer Phổ biến

```
UPX          - Mã nguồn mở, dễ giải nén
Themida      - Thương mại, bảo vệ dựa trên VM
VMProtect    - Thương mại, ảo hóa mã
ASPack       - Packer nén
PECompact    - Packer nén
Enigma       - Packer thương mại
```

### Phương pháp Giải nén

```
1. Xác định packer (DIE, Exeinfo PE, PEiD)

2. Giải nén tĩnh (nếu biết packer):
   - UPX: upx -d packed.exe
   - Sử dụng các trình giải nén hiện có

3. Giải nén động:
   a. Tìm Điểm Nhập Gốc (Original Entry Point - OEP)
   b. Đặt điểm ngắt tại OEP
   c. Dump bộ nhớ khi OEP đạt được
   d. Sửa bảng nhập (Scylla, ImpREC)

4. Các kỹ thuật tìm OEP:
   - Điểm ngắt phần cứng trên stack (thủ thuật ESP)
   - Ngắt tại các lệnh gọi API phổ biến (GetCommandLineA)
   - Theo dõi và tìm kiếm các mẫu nhập điển hình
```

### Ví dụ Giải nén Thủ công

```
1. Tải tệp nhị phân đã đóng gói vào x64dbg
2. Ghi chú điểm nhập (packer stub)
3. Sử dụng thủ thuật ESP:
   - Chạy đến điểm nhập
   - Đặt điểm ngắt phần cứng tại [ESP]
   - Chạy cho đến khi điểm ngắt chạm (sau PUSHAD/POPAD)
4. Tìm JMP đến OEP
5. Tại OEP, sử dụng Scylla để:
   - Dump quy trình
   - Tìm các imports (IAT autosearch)
   - Sửa bản dump
```

## Bảo vệ dựa trên Ảo hóa

### Ảo hóa Mã

```
Mã x86 gốc được chuyển đổi thành bytecode tùy chỉnh
được thông dịch bởi VM nhúng khi chạy.

Gốc:          VM Bảo vệ:
mov eax, 1    push vm_context
add eax, 2    call vm_entry
              ; VM thông dịch bytecode
              ; tương đương với gốc
```

### Các Cách tiếp cận Phân tích

```
1. Xác định các thành phần VM:
   - VM entry (dispatcher)
   - Bảng xử lý (Handler table)
   - Vị trí bytecode
   - Các thanh ghi/stack ảo

2. Theo dõi thực thi:
   - Ghi lại các cuộc gọi xử lý
   - Ánh xạ bytecode tới các hoạt động
   - Hiểu tập lệnh

3. Lifting/devirtualization:
   - Ánh xạ các lệnh VM trở lại native
   - Công cụ: VMAttack, SATURN, NoVmp

4. Thực thi ký hiệu:
   - Phân tích VM theo ngữ nghĩa
   - angr, Triton
```

## Tóm tắt Chiến lược Vượt qua

### Các Nguyên tắc Chung

1. **Hiểu sự bảo vệ**: Xác định kỹ thuật nào được sử dụng
2. **Tìm kiểm tra**: Định vị mã bảo vệ trong tệp nhị phân
3. **Vá hoặc hook**: Sửa đổi kiểm tra để luôn vượt qua
4. **Sử dụng các công cụ thích hợp**: ScyllaHide, các plugin x64dbg
5. **Ghi lại các phát hiện**: Giữ ghi chú về các bảo vệ đã vượt qua

### Khuyến nghị Công cụ

```
Vượt qua chống gỡ lỗi: ScyllaHide, TitanHide
Giải nén:            x64dbg + Scylla, OllyDumpEx
Gỡ làm rối:          D-810, SATURN, miasm
Phân tích VM:        VMAttack, NoVmp, theo dõi thủ công
Giải mã chuỗi:       FLOSS, các script tùy chỉnh
Thực thi ký hiệu:    angr, Triton
```

### Cân nhắc Đạo đức

Kiến thức này chỉ nên được sử dụng cho:

- Nghiên cứu bảo mật được ủy quyền
- Phân tích phần mềm độc hại (phòng thủ)
- Các cuộc thi CTF
- Hiểu các biện pháp bảo vệ cho các mục đích hợp pháp
- Mục đích giáo dục

Không bao giờ sử dụng để vượt qua các biện pháp bảo vệ cho:

- Vi phạm bản quyền phần mềm
- Truy cập trái phép
- Mục đích độc hại
