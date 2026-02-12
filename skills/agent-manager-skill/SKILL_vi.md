---
name: agent-manager-skill
description: Quản lý nhiều đại lý CLI cục bộ thông qua các phiên tmux (bắt đầu/dừng/giám sát/giao việc) với khả năng lập lịch thân thiện với cron.
---

# Kỹ năng Quản lý Đại lý (Agent Manager Skill)

## Khi nào nên sử dụng

Sử dụng kỹ năng này khi bạn cần:

- Chạy nhiều đại lý CLI cục bộ một cách song song (trong các phiên tmux riêng biệt)
- Bắt đầu/dừng các đại lý và theo dõi (tail) nhật ký (logs) của họ
- Giao nhiệm vụ cho các đại lý và giám sát đầu ra
- Lập lịch cho các công việc định kỳ của đại lý (cron)

## Điều kiện tiên quyết

Cài đặt `agent-manager-skill` trong không gian làm việc của bạn:

```bash
git clone https://github.com/fractalmind-ai/agent-manager-skill.git
```

## Các lệnh thông dụng

```bash
python3 agent-manager/scripts/main.py doctor
python3 agent-manager/scripts/main.py list
python3 agent-manager/scripts/main.py start EMP_0001
python3 agent-manager/scripts/main.py monitor EMP_0001 --follow
python3 agent-manager/scripts/main.py assign EMP_0002 <<'EOF'
Làm theo quy trình công việc trong teams/fractalmind-ai-maintenance.md
EOF
```

## Ghi chú

- Yêu cầu `tmux` và `python3`.
- Các đại lý được cấu hình trong thư mục `agents/` (xem kho lưu trữ để biết các ví dụ).
