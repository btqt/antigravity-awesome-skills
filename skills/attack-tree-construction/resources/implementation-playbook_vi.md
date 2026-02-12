# Sổ tay Triển khai Xây dựng Cây Tấn công (Attack Tree Construction Implementation Playbook)

Tệp này chứa các mẫu, danh sách kiểm tra và các mẫu mã chi tiết được tham chiếu bởi kỹ năng.

## Các Khái niệm Cốt lõi

### 1. Cấu trúc Cây Tấn công

```
                    [Mục tiêu gốc]
                          |
             ┌────────────┴────────────┐
             │                         │
      [Mục tiêu phụ 1]          [Mục tiêu phụ 2]
         (Nút OR)                  (Nút AND)
             │                         │
       ┌─────┴─────┐             ┌─────┴─────┐
       │           │             │           │
   [Tấn công]  [Tấn công]    [Tấn công]  [Tấn công]
    (Nút lá)    (Nút lá)      (Nút lá)    (Nút lá)
```

### 2. Các loại Nút

| Loại          | Ký hiệu       | Mô tả                                |
| ------------- | ------------- | ------------------------------------ |
| **OR**        | Hình bầu dục  | Bất kỳ nút con nào đạt được mục tiêu |
| **AND**       | Hình chữ nhật | Tất cả các nút con đều bắt buộc      |
| **Lá (Leaf)** | Hình hộp      | Bước tấn công nguyên tử              |

### 3. Các Thuộc tính Tấn công

| Thuộc tính    | Mô tả                 | Giá trị               |
| ------------- | --------------------- | --------------------- |
| **Chi phí**   | Nguồn lực cần thiết   | $, $$, $$$            |
| **Thời gian** | Thời gian thực hiện   | Giờ, Ngày, Tuần       |
| **Kỹ năng**   | Chuyên môn bắt buộc   | Thấp, Trung bình, Cao |
| **Phát hiện** | Khả năng bị phát hiện | Thấp, Trung bình, Cao |

## Các khuôn mẫu (Templates)

### Khuôn mẫu 1: Mô hình Dữ liệu Cây Tấn công

```python
from dataclasses import dataclass, field
from enum import Enum
from typing import List, Dict, Optional, Union
import json

class NodeType(Enum):
    OR = "or"
    AND = "and"
    LEAF = "leaf"


class Difficulty(Enum):
    TRIVIAL = 1
    LOW = 2
    MEDIUM = 3
    HIGH = 4
    EXPERT = 5


class Cost(Enum):
    FREE = 0
    LOW = 1
    MEDIUM = 2
    HIGH = 3
    VERY_HIGH = 4


class DetectionRisk(Enum):
    NONE = 0
    LOW = 1
    MEDIUM = 2
    HIGH = 3
    CERTAIN = 4


@dataclass
class AttackAttributes:
    difficulty: Difficulty = Difficulty.MEDIUM
    cost: Cost = Cost.MEDIUM
    detection_risk: DetectionRisk = DetectionRisk.MEDIUM
    time_hours: float = 8.0
    requires_insider: bool = False
    requires_physical: bool = False


@dataclass
class AttackNode:
    id: str
    name: str
    description: str
    node_type: NodeType
    attributes: AttackAttributes = field(default_factory=AttackAttributes)
    children: List['AttackNode'] = field(default_factory=list)
    mitigations: List[str] = field(default_factory=list)
    cve_refs: List[str] = field(default_factory=list)

    def add_child(self, child: 'AttackNode') -> None:
        self.children.append(child)

    def calculate_path_difficulty(self) -> float:
        """Tính toán độ khó tổng hợp cho đường dẫn này."""
        if self.node_type == NodeType.LEAF:
            return self.attributes.difficulty.value

        if not self.children:
            return 0

        child_difficulties = [c.calculate_path_difficulty() for c in self.children]

        if self.node_type == NodeType.OR:
            return min(child_difficulties)
        else:  # AND
            return max(child_difficulties)

    def calculate_path_cost(self) -> float:
        """Tính toán tổng chi phí cho đường dẫn này."""
        if self.node_type == NodeType.LEAF:
            return self.attributes.cost.value

        if not self.children:
            return 0

        child_costs = [c.calculate_path_cost() for c in self.children]

        if self.node_type == NodeType.OR:
            return min(child_costs)
        else:  # AND
            return sum(child_costs)

    def to_dict(self) -> Dict:
        """Chuyển đổi sang từ điển để tuần tự hóa."""
        return {
            "id": self.id,
            "name": self.name,
            "description": self.description,
            "type": self.node_type.value,
            "attributes": {
                "difficulty": self.attributes.difficulty.name,
                "cost": self.attributes.cost.name,
                "detection_risk": self.attributes.detection_risk.name,
                "time_hours": self.attributes.time_hours,
            },
            "mitigations": self.mitigations,
            "children": [c.to_dict() for c in self.children]
        }


@dataclass
class AttackTree:
    name: str
    description: str
    root: AttackNode
    version: str = "1.0"

    def find_easiest_path(self) -> List[AttackNode]:
        """Tìm đường dẫn có độ khó thấp nhất."""
        return self._find_path(self.root, minimize="difficulty")

    def find_cheapest_path(self) -> List[AttackNode]:
        """Tìm đường dẫn có chi phí thấp nhất."""
        return self._find_path(self.root, minimize="cost")

    def find_stealthiest_path(self) -> List[AttackNode]:
        """Tìm đường dẫn có rủi ro bị phát hiện thấp nhất."""
        return self._find_path(self.root, minimize="detection")

    def _find_path(
        self,
        node: AttackNode,
        minimize: str
    ) -> List[AttackNode]:
        """Tìm kiếm đường dẫn đệ quy."""
        if node.node_type == NodeType.LEAF:
            return [node]

        if not node.children:
            return [node]

        if node.node_type == NodeType.OR:
            # Chọn đường dẫn của nút con tốt nhất
            best_path = None
            best_score = float('inf')

            for child in node.children:
                child_path = self._find_path(child, minimize)
                score = self._path_score(child_path, minimize)
                if score < best_score:
                    best_score = score
                    best_path = child_path

            return [node] + (best_path or [])
        else:  # AND
            # Phải đi qua tất cả các nút con
            path = [node]
            for child in node.children:
                path.extend(self._find_path(child, minimize))
            return path

    def _path_score(self, path: List[AttackNode], metric: str) -> float:
        """Tính điểm cho một đường dẫn."""
        if metric == "difficulty":
            return sum(n.attributes.difficulty.value for n in path if n.node_type == NodeType.LEAF)
        elif metric == "cost":
            return sum(n.attributes.cost.value for n in path if n.node_type == NodeType.LEAF)
        elif metric == "detection":
            return sum(n.attributes.detection_risk.value for n in path if n.node_type == NodeType.LEAF)
        return 0

    def get_all_leaf_attacks(self) -> List[AttackNode]:
        """Lấy tất cả các nút lá tấn công."""
        leaves = []
        self._collect_leaves(self.root, leaves)
        return leaves

    def _collect_leaves(self, node: AttackNode, leaves: List[AttackNode]) -> None:
        if node.node_type == NodeType.LEAF:
            leaves.append(node)
        for child in node.children:
            self._collect_leaves(child, leaves)

    def get_unmitigated_attacks(self) -> List[AttackNode]:
        """Tìm các cuộc tấn công chưa có biện pháp giảm thiểu."""
        return [n for n in self.get_all_leaf_attacks() if not n.mitigations]

    def export_json(self) -> str:
        """Xuất cây ra định dạng JSON."""
        return json.dumps({
            "name": self.name,
            "description": self.description,
            "version": self.version,
            "root": self.root.to_dict()
        }, indent=2)
```

### Khuôn mẫu 2: Trình dựng Cây Tấn công (Attack Tree Builder)

```python
class AttackTreeBuilder:
    """Fluent builder cho các cây tấn công."""

    def __init__(self, name: str, description: str):
        self.name = name
        self.description = description
        self._node_stack: List[AttackNode] = []
        self._root: Optional[AttackNode] = None

    def goal(self, id: str, name: str, description: str = "") -> 'AttackTreeBuilder':
        """Thiết lập mục tiêu gốc (mặc định là nút OR)."""
        self._root = AttackNode(
            id=id,
            name=name,
            description=description,
            node_type=NodeType.OR
        )
        self._node_stack = [self._root]
        return self

    def or_node(self, id: str, name: str, description: str = "") -> 'AttackTreeBuilder':
        """Thêm một mục tiêu phụ OR."""
        node = AttackNode(
            id=id,
            name=name,
            description=description,
            node_type=NodeType.OR
        )
        self._current().add_child(node)
        self._node_stack.append(node)
        return self

    def and_node(self, id: str, name: str, description: str = "") -> 'AttackTreeBuilder':
        """Thêm một mục tiêu phụ AND (bắt buộc tất cả nút con)."""
        node = AttackNode(
            id=id,
            name=name,
            description=description,
            node_type=NodeType.AND
        )
        self._current().add_child(node)
        self._node_stack.append(node)
        return self

    def attack(
        self,
        id: str,
        name: str,
        description: str = "",
        difficulty: Difficulty = Difficulty.MEDIUM,
        cost: Cost = Cost.MEDIUM,
        detection: DetectionRisk = DetectionRisk.MEDIUM,
        time_hours: float = 8.0,
        mitigations: List[str] = None
    ) -> 'AttackTreeBuilder':
        """Thêm một nút lá tấn công."""
        node = AttackNode(
            id=id,
            name=name,
            description=description,
            node_type=NodeType.LEAF,
            attributes=AttackAttributes(
                difficulty=difficulty,
                cost=cost,
                detection_risk=detection,
                time_hours=time_hours
            ),
            mitigations=mitigations or []
        )
        self._current().add_child(node)
        return self

    def end(self) -> 'AttackTreeBuilder':
        """Đóng nút hiện tại, quay lại nút cha."""
        if len(self._node_stack) > 1:
            self._node_stack.pop()
        return self

    def build(self) -> AttackTree:
        """Xây dựng cây tấn công."""
        if not self._root:
            raise ValueError("Chưa định nghĩa mục tiêu gốc")
        return AttackTree(
            name=self.name,
            description=self.description,
            root=self._root
        )

    def _current(self) -> AttackNode:
        if not self._node_stack:
            raise ValueError("Không có nút hiện tại")
        return self._node_stack[-1]


# Ví dụ sử dụng
def build_account_takeover_tree() -> AttackTree:
    """Xây dựng cây tấn công cho kịch bản chiếm đoạt tài khoản."""
    return (
        AttackTreeBuilder("Chiếm đoạt Tài khoản", "Truy cập trái phép vào tài khoản người dùng")
        .goal("G1", "Chiếm đoạt Tài khoản Người dùng")

        .or_node("S1", "Đánh cắp Thông tin đăng nhập")
            .attack(
                "A1", "Tấn công Phishing",
                difficulty=Difficulty.LOW,
                cost=Cost.LOW,
                detection=DetectionRisk.MEDIUM,
                mitigations=["Đào tạo nhận thức bảo mật", "Lọc email"]
            )
            .attack(
                "A2", "Credential Stuffing",
                difficulty=Difficulty.TRIVIAL,
                cost=Cost.LOW,
                detection=DetectionRisk.HIGH,
                mitigations=["Giới hạn tốc độ", "MFA", "Giám sát rò rỉ mật khẩu"]
            )
            .attack(
                "A3", "Mã độc Keylogger",
                difficulty=Difficulty.MEDIUM,
                cost=Cost.MEDIUM,
                detection=DetectionRisk.MEDIUM,
                mitigations=["Bảo vệ điểm cuối", "MFA"]
            )
        .end()

        .or_node("S2", "Vượt qua Xác thực")
            .attack(
                "A4", "Session Hijacking",
                difficulty=Difficulty.MEDIUM,
                cost=Cost.LOW,
                detection=DetectionRisk.LOW,
                mitigations=["Quản lý phiên an toàn", "Chỉ HTTPS"]
            )
            .attack(
                "A5", "Lỗ hổng Vượt qua Xác thực",
                difficulty=Difficulty.HIGH,
                cost=Cost.LOW,
                detection=DetectionRisk.LOW,
                mitigations=["Kiểm thử bảo mật", "Review mã nguồn", "WAF"]
            )
        .end()

        .or_node("S3", "Kỹ thuật Xã hội (Social Engineering)")
            .and_node("S3.1", "Tấn công Phục hồi Tài khoản")
                .attack(
                    "A6", "Thu thập Thông tin Cá nhân",
                    difficulty=Difficulty.LOW,
                    cost=Cost.FREE,
                    detection=DetectionRisk.NONE
                )
                .attack(
                    "A7", "Gọi điện cho Bộ phận Hỗ trợ",
                    difficulty=Difficulty.MEDIUM,
                    cost=Cost.FREE,
                    detection=DetectionRisk.MEDIUM,
                    mitigations=["Quy trình xác minh hỗ trợ", "Câu hỏi bảo mật"]
                )
            .end()
        .end()

        .build()
    )
```

### Khuôn mẫu 3: Trình tạo Sơ đồ Mermaid

```python
class MermaidExporter:
    """Xuất các cây tấn công sang định dạng sơ đồ Mermaid."""

    def __init__(self, tree: AttackTree):
        self.tree = tree
        self._lines: List[str] = []
        self._node_count = 0

    def export(self) -> str:
        """Xuất cây sang sơ đồ luồng Mermaid."""
        self._lines = ["flowchart TD"]
        self._export_node(self.tree.root, None)
        return "\n".join(self._lines)

    def _export_node(self, node: AttackNode, parent_id: Optional[str]) -> str:
        """Đệ quy xuất các nút."""
        node_id = f"N{self._node_count}"
        self._node_count += 1

        # Hình dáng nút dựa trên loại
        if node.node_type == NodeType.OR:
            shape = f"{node_id}(({node.name}))"
        elif node.node_type == NodeType.AND:
            shape = f"{node_id}[{node.name}]"
        else:  # LEAF
            # Màu dựa trên độ khó
            style = self._get_leaf_style(node)
            shape = f"{node_id}[/{node.name}/]"
            self._lines.append(f"    style {node_id} {style}")

        self._lines.append(f"    {shape}")

        if parent_id:
            connector = "-->" if node.node_type != NodeType.AND else "==>"
            self._lines.append(f"    {parent_id} {connector} {node_id}")

        for child in node.children:
            self._export_node(child, node_id)

        return node_id

    def _get_leaf_style(self, node: AttackNode) -> str:
        """Lấy kiểu dáng dựa trên các thuộc tính tấn công."""
        colors = {
            Difficulty.TRIVIAL: "fill:#ff6b6b",  # Đỏ - tấn công dễ
            Difficulty.LOW: "fill:#ffa06b",
            Difficulty.MEDIUM: "fill:#ffd93d",
            Difficulty.HIGH: "fill:#6bcb77",
            Difficulty.EXPERT: "fill:#4d96ff",  # Xanh dương - tấn công khó
        }
        color = colors.get(node.attributes.difficulty, "fill:#gray")
        return color


class PlantUMLExporter:
    """Xuất các cây tấn công sang định dạng PlantUML."""

    def __init__(self, tree: AttackTree):
        self.tree = tree

    def export(self) -> str:
        """Xuất cây sang PlantUML."""
        lines = [
            "@startmindmap",
            f"* {self.tree.name}",
        ]
        self._export_node(self.tree.root, lines, 1)
        lines.append("@endmindmap")
        return "\n".join(lines)

    def _export_node(self, node: AttackNode, lines: List[str], depth: int) -> None:
        """Đệ quy xuất các nút."""
        prefix = "*" * (depth + 1)

        if node.node_type == NodeType.OR:
            marker = "[OR]"
        elif node.node_type == NodeType.AND:
            marker = "[AND]"
        else:
            diff = node.attributes.difficulty.name
            marker = f"<<{diff}>>"

        lines.append(f"{prefix} {marker} {node.name}")

        for child in node.children:
            self._export_node(child, lines, depth + 1)
```

### Khuôn mẫu 4: Phân tích Đường dẫn Tấn công (Attack Path Analysis)

```python
from typing import Set, Tuple

class AttackPathAnalyzer:
    """Phân tích các đường dẫn tấn công và độ bao phủ."""

    def __init__(self, tree: AttackTree):
        self.tree = tree

    def get_all_paths(self) -> List[List[AttackNode]]:
        """Lấy tất cả các đường dẫn tấn công có thể."""
        paths = []
        self._collect_paths(self.tree.root, [], paths)
        return paths

    def _collect_paths(
        self,
        node: AttackNode,
        current_path: List[AttackNode],
        all_paths: List[List[AttackNode]]
    ) -> None:
        """Đệ quy thu thập tất cả các đường dẫn."""
        current_path = current_path + [node]

        if node.node_type == NodeType.LEAF:
            all_paths.append(current_path)
            return

        if not node.children:
            all_paths.append(current_path)
            return

        if node.node_type == NodeType.OR:
            # Mỗi nút con là một đường dẫn riêng biệt
            for child in node.children:
                self._collect_paths(child, current_path, all_paths)
        else:  # AND
            # Phải kết hợp tất cả các nút con
            child_paths = []
            for child in node.children:
                child_sub_paths = []
                self._collect_paths(child, [], child_sub_paths)
                child_paths.append(child_sub_paths)

            # Kết hợp các đường dẫn từ tất cả các nút con của AND
            combined = self._combine_and_paths(child_paths)
            for combo in combined:
                all_paths.append(current_path + combo)

    def _combine_and_paths(
        self,
        child_paths: List[List[List[AttackNode]]]
    ) -> List[List[AttackNode]]:
        """Kết hợp các đường dẫn từ các nút con của nút AND."""
        if not child_paths:
            return [[]]

        if len(child_paths) == 1:
            return [path for paths in child_paths for path in paths]

        # Tích Descartes của tất cả các tổ hợp đường dẫn con
        result = [[]]
        for paths in child_paths:
            new_result = []
            for existing in result:
                for path in paths:
                    new_result.append(existing + path)
            result = new_result
        return result

    def calculate_path_metrics(self, path: List[AttackNode]) -> Dict:
        """Tính toán các chỉ số cho một đường dẫn cụ thể."""
        leaves = [n for n in path if n.node_type == NodeType.LEAF]

        total_difficulty = sum(n.attributes.difficulty.value for n in leaves)
        total_cost = sum(n.attributes.cost.value for n in leaves)
        total_time = sum(n.attributes.time_hours for n in leaves)
        max_detection = max((n.attributes.detection_risk.value for n in leaves), default=0)

        return {
            "steps": len(leaves),
            "total_difficulty": total_difficulty,
            "avg_difficulty": total_difficulty / len(leaves) if leaves else 0,
            "total_cost": total_cost,
            "total_time_hours": total_time,
            "max_detection_risk": max_detection,
            "requires_insider": any(n.attributes.requires_insider for n in leaves),
            "requires_physical": any(n.attributes.requires_physical for n in leaves),
        }

    def identify_critical_nodes(self) -> List[Tuple[AttackNode, int]]:
        """Tìm các nút xuất hiện trong nhiều đường dẫn nhất."""
        paths = self.get_all_paths()
        node_counts: Dict[str, Tuple[AttackNode, int]] = {}

        for path in paths:
            for node in path:
                if node.id not in node_counts:
                    node_counts[node.id] = (node, 0)
                node_counts[node.id] = (node, node_counts[node.id][1] + 1)

        return sorted(
            node_counts.values(),
            key=lambda x: x[1],
            reverse=True
        )

    def coverage_analysis(self, mitigated_attacks: Set[str]) -> Dict:
        """Phân tích cách các biện pháp giảm thiểu ảnh hưởng đến độ bao phủ tấn công."""
        all_paths = self.get_all_paths()
        blocked_paths = []
        open_paths = []

        for path in all_paths:
            path_attacks = {n.id for n in path if n.node_type == NodeType.LEAF}
            if path_attacks & mitigated_attacks:
                blocked_paths.append(path)
            else:
                open_paths.append(path)

        return {
            "total_paths": len(all_paths),
            "blocked_paths": len(blocked_paths),
            "open_paths": len(open_paths),
            "coverage_percentage": len(blocked_paths) / len(all_paths) * 100 if all_paths else 0,
            "open_path_details": [
                {"path": [n.name for n in p], "metrics": self.calculate_path_metrics(p)}
                for p in open_paths[:5]  # Top 5 đường dẫn vẫn hở
            ]
        }

    def prioritize_mitigations(self) -> List[Dict]:
        """Ưu tiên các biện pháp giảm thiểu theo tác động."""
        critical_nodes = self.identify_critical_nodes()
        paths = self.get_all_paths()
        total_paths = len(paths)

        recommendations = []
        for node, count in critical_nodes:
            if node.node_type == NodeType.LEAF and node.mitigations:
                recommendations.append({
                    "attack": node.name,
                    "attack_id": node.id,
                    "paths_blocked": count,
                    "coverage_impact": count / total_paths * 100,
                    "difficulty": node.attributes.difficulty.name,
                    "mitigations": node.mitigations,
                })

        return sorted(recommendations, key=lambda x: x["coverage_impact"], reverse=True)
```

## Thực hành Tốt nhất

### Nên làm (Do's)

- **Bắt đầu với mục tiêu rõ ràng** - Xác định những gì kẻ tấn công muốn.
- **Tính toán kỹ lưỡng** - Xem xét tất cả các vector tấn công.
- **Gán thuộc tính cho các cuộc tấn công** - Chi phí, kỹ năng và khả năng phát hiện.
- **Cập nhật thường xuyên** - Các mối đe dọa mới luôn xuất hiện.
- **Xác thực với các chuyên gia** - Review từ đội Red Team.

### Không nên làm (Don'ts)

- **Đừng đơn giản hóa quá mức** - Các cuộc tấn công thực tế rất phức tạp.
- **Đừng bỏ qua các phụ thuộc** - Các nút AND rất quan trọng.
- **Đừng quên các mối đe dọa từ bên trong** - Không phải tất cả kẻ tấn công đều từ bên ngoài.
- **Đừng bỏ qua các biện pháp giảm thiểu** - Cây tấn công được dùng để lập kế hoạch phòng thủ.
- **Đừng làm cho nó trở nên tĩnh** - Bối cảnh đe dọa luôn phát triển.

## Tài nguyên

- [Attack Trees của Bruce Schneier](https://www.schneier.com/academic/archives/1999/12/attack_trees.html)
- [Khung công tác MITRE ATT&CK](https://attack.mitre.org/)
- [Phân tích Bề mặt Tấn công của OWASP](https://owasp.org/www-community/controls/Attack_Surface_Analysis_Cheat_Sheet)
