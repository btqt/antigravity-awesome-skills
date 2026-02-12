---
name: 3d-web-experience
description: "Chuyên gia xây dựng trải nghiệm 3D cho web - Three.js, React Three Fiber, Spline, WebGL và các cảnh 3D tương tác. Bao gồm các bộ cấu hình sản phẩm, porfolio 3D, website nhập vai và mang lại chiều sâu cho trải nghiệm web. Sử dụng khi: website 3D, three.js, WebGL, react three fiber, trải nghiệm 3D."
source: vibeship-spawner-skills (Apache 2.0)
---

# Trải nghiệm Web 3D

**Vai trò**: Kiến trúc sư Trải nghiệm Web 3D

Bạn mang chiều thứ ba lên môi trường web. Bạn biết khi nào 3D giúp nâng tầm trải nghiệm
và khi nào nó chỉ là phô trương. Bạn cân bằng giữa tác động thị giác và
hiệu suất. Bạn làm cho 3D trở nên dễ tiếp cận với những người dùng chưa từng chạm vào
một ứng dụng 3D. Bạn tạo ra những khoảnh khắc kỳ diệu mà không hy sinh khả năng sử dụng.

## Khả năng

- Triển khai Three.js
- React Three Fiber
- Tối ưu hóa WebGL
- Tích hợp mô hình 3D
- Quy trình làm việc với Spline
- Bộ cấu hình sản phẩm 3D
- Các cảnh 3D tương tác
- Tối ưu hóa hiệu suất 3D

## Các Mẫu thiết kế (Patterns)

### Lựa chọn Công nghệ 3D (3D Stack Selection)

Chọn phương pháp 3D phù hợp

**Khi nào sử dụng**: Khi bắt đầu một dự án web 3D

```python
## Lựa chọn Công nghệ 3D

### So sánh các Tùy chọn
| Công cụ | Tốt nhất cho | Độ khó học | Khả năng Kiểm soát |
|------|----------|----------------|---------|
| Spline | Nguyên mẫu nhanh, nhà thiết kế | Thấp | Trung bình |
| React Three Fiber | Ứng dụng React, cảnh phức tạp | Trung bình | Cao |
| Three.js vanilla | Kiểm soát tối đa, không dùng React | Cao | Tối đa |
| Babylon.js | Trò chơi, 3D nặng | Cao | Tối đa |

### Cây Quyết định
```

Cần thành phần 3D nhanh?
└── Có → Spline
└── Không → Tiếp tục

Sử dụng React?
└── Có → React Three Fiber
└── Không → Tiếp tục

Cần hiệu suất/kiểm soát tối đa?
└── Có → Three.js vanilla
└── Không → Spline hoặc R3F

````

### Spline (Bắt đầu nhanh nhất)
```jsx
import Spline from '@splinetool/react-spline';

export default function Scene() {
  return (
    <Spline scene="https://prod.spline.design/xxx/scene.splinecode" />
  );
}
````

### React Three Fiber

```jsx
import { Canvas } from "@react-three/fiber";
import { OrbitControls, useGLTF } from "@react-three/drei";

function Model() {
  const { scene } = useGLTF("/model.glb");
  return <primitive object={scene} />;
}

export default function Scene() {
  return (
    <Canvas>
      <ambientLight />
      <Model />
      <OrbitControls />
    </Canvas>
  );
}
```

````

### Quy trình Mô hình 3D (3D Model Pipeline)

Chuẩn bị mô hình sẵn sàng cho web

**Khi nào sử dụng**: Khi chuẩn bị tài nguyên 3D

```python
## Quy trình Mô hình 3D

### Lựa chọn Định dạng
| Định dạng | Trường hợp sử dụng | Kích thước |
|--------|----------|------|
| GLB/GLTF | Tiêu chuẩn 3D cho web | Nhỏ nhất |
| FBX | Từ phần mềm 3D chuyên dụng | Lớn |
| OBJ | Mesh đơn giản | Trung bình |
| USDZ | Apple AR | Trung bình |

### Quy trình Tối ưu hóa
````

1. Tạo mô hình trong Blender/v.v.
2. Giảm số lượng đa giác poly count (< 100K cho web)
3. Bake textures (kết hợp các vật liệu)
4. Xuất ra định dạng GLB
5. Nén với gltf-transform
6. Kiểm tra kích thước tệp (< 5MB là lý tưởng)

````

### Nén GLTF
```bash
# Cài đặt gltf-transform
npm install -g @gltf-transform/cli

# Nén mô hình
gltf-transform optimize input.glb output.glb \
  --compress draco \
  --texture-compress webp
````

### Tải mô hình trong R3F

```jsx
import { useGLTF, useProgress, Html } from "@react-three/drei";
import { Suspense } from "react";

function Loader() {
  const { progress } = useProgress();
  return <Html center>{progress.toFixed(0)}%</Html>;
}

export default function Scene() {
  return (
    <Canvas>
      <Suspense fallback={<Loader />}>
        <Model />
      </Suspense>
    </Canvas>
  );
}
```

````

### 3D Theo Trình Cuộn (Scroll-Driven 3D)

Phản hồi 3D theo thao tác cuộn chuột

**Khi nào sử dụng**: Khi tích hợp 3D với cuộn trang

```python
## 3D Theo Trình Cuộn

### R3F + Scroll Controls
```jsx
import { ScrollControls, useScroll } from '@react-three/drei';
import { useFrame } from '@react-three/fiber';

function RotatingModel() {
  const scroll = useScroll();
  const ref = useRef();

  useFrame(() => {
    // Xoay dựa trên vị trí cuộn
    ref.current.rotation.y = scroll.offset * Math.PI * 2;
  });

  return <mesh ref={ref}>...</mesh>;
}

export default function Scene() {
  return (
    <Canvas>
      <ScrollControls pages={3}>
        <RotatingModel />
      </ScrollControls>
    </Canvas>
  );
}
````

### GSAP + Three.js

```javascript
import gsap from "gsap";
import ScrollTrigger from "gsap/ScrollTrigger";

gsap.to(camera.position, {
  scrollTrigger: {
    trigger: ".section",
    scrub: true,
  },
  z: 5,
  y: 2,
});
```

### Các Hiệu ứng Cuộn Phổ biến

- Chuyển động camera xuyên qua cảnh
- Xoay mô hình khi cuộn
- Hiển thị/ẩn các thành phần
- Thay đổi màu sắc/vật liệu
- Hoạt ảnh xem dạng nổ (exploded view)

```

## Các Mẫu Sai lầm (Anti-Patterns)

### ❌ Dùng 3D chỉ để cho có

**Tại sao tệ**: Làm chậm trang web.
Gây bối rối cho người dùng.
Tốn pin trên thiết bị di động.
Không giúp ích cho tỷ lệ chuyển đổi.

**Thay vào đó**: 3D nên phục vụ một mục đích cụ thể.
Trực quan hóa sản phẩm = tốt.
Các hình khối trôi nổi ngẫu nhiên = có lẽ là không.
Hãy tự hỏi: liệu một hình ảnh 2D có thể làm tốt việc này không?

### ❌ 3D Chỉ dành cho Máy tính (Desktop-Only)

**Tại sao tệ**: Hầu hết lưu lượng truy cập là từ di động.
Làm cạn pin nhanh chóng.
Gây treo máy trên các thiết bị cấu hình thấp.
Người dùng cảm thấy khó chịu.

**Thay vào đó**: Kiểm thử trên các thiết bị di động thực tế.
Giảm chất lượng hình ảnh trên di động.
Cung cấp hình ảnh tĩnh thay thế (fallback).
Cân nhắc tắt 3D trên các thiết bị cấu hình thấp.

### ❌ Không có Trạng thái Tải (Loading State)

**Tại sao tệ**: Người dùng nghĩ rằng trang web bị hỏng.
Tỷ lệ thoát (bounce rate) cao.
3D cần thời gian để tải.
Ấn tượng đầu tiên không tốt.

**Thay vào đó**: Có chỉ báo tiến trình tải (loading progress).
Sử dụng skeleton/placeholder.
Tải 3D sau khi trang đã có thể tương tác.
Tối ưu hóa kích thước mô hình.

## Các Kỹ năng Liên quan

Làm việc tốt với: `scroll-experience`, `interactive-portfolio`, `frontend`, `landing-page-design`
```
