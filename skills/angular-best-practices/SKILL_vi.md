---
name: angular-best-practices
description: Hướng dẫn tối ưu hóa hiệu suất và các thực hành tốt nhất cho Angular. Sử dụng khi viết, xem xét hoặc tái cấu trúc mã Angular để đạt hiệu suất tối ưu, kích thước gói nhỏ và hiệu quả kết xuất.
risk: safe
source: self
---

# Các Thực hành tốt nhất cho Angular

Hướng dẫn tối ưu hóa hiệu suất toàn diện cho các ứng dụng Angular. Chứa các quy tắc được ưu tiên để loại bỏ các điểm nghẽn hiệu suất, tối ưu hóa kích thước gói và cải thiện khả năng kết xuất.

## Khi nào nên áp dụng

Tham khảo các hướng dẫn này khi:

- Viết các thành phần (components) hoặc trang Angular mới
- Triển khai các mẫu lấy dữ liệu (data fetching patterns)
- Xem xét mã (code review) để tìm các vấn đề hiệu suất
- Tái cấu trúc mã Angular hiện có
- Tối ưu hóa kích thước gói hoặc thời gian tải
- Cấu hình SSR/hydration

---

## Các danh mục Quy tắc theo Mức độ ưu tiên

| Mức độ ưu tiên | Danh mục              | Tác động        | Trọng tâm                                |
| -------------- | --------------------- | --------------- | ---------------------------------------- |
| 1              | Phát hiện Thay đổi    | NGHIÊM TRỌNG    | Signals, OnPush, Zoneless                |
| 2              | Thác nước Bất đồng bộ | NGHIÊM TRỌNG    | Các mẫu RxJS, tải trước SSR              |
| 3              | Tối ưu hóa Gói        | NGHIÊM TRỌNG    | Tải chậm (Lazy loading), tree shaking    |
| 4              | Hiệu suất Kết xuất    | CAO             | @defer, trackBy, ảo hóa (virtualization) |
| 5              | Kết xuất phía Máy chủ | CAO             | Hydration, prerendering                  |
| 6              | Tối ưu hóa Mẫu        | TRUNG BÌNH      | Luồng điều khiển, pipes                  |
| 7              | Quản lý Trạng thái    | TRUNG BÌNH      | Các mẫu Signal, selectors                |
| 8              | Quản lý Bộ nhớ        | THẤP-TRUNG BÌNH | Dọn dẹp, subscriptions                   |

---

## 1. Phát hiện Thay đổi (NGHIÊM TRỌNG)

### Sử dụng Phát hiện Thay đổi OnPush

```typescript
// ĐÚNG - OnPush với Signals
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `<div>{{ count() }}</div>`,
})
export class CounterComponent {
  count = signal(0);
}

// SAI - Phát hiện thay đổi mặc định
@Component({
  template: `<div>{{ count }}</div>`, // Được kiểm tra mỗi chu kỳ
})
export class CounterComponent {
  count = 0;
}
```

### Ưu tiên Signals hơn các Thuộc tính có thể thay đổi (Mutable Properties)

```typescript
// ĐÚNG - Signals kích hoạt cập nhật chính xác
@Component({
  template: `
    <h1>{{ title() }}</h1>
    <p>Count: {{ count() }}</p>
  `,
})
export class DashboardComponent {
  title = signal("Dashboard");
  count = signal(0);
}

// SAI - Thuộc tính có thể thay đổi yêu cầu zone.js kiểm tra
@Component({
  template: `
    <h1>{{ title }}</h1>
    <p>Count: {{ count }}</p>
  `,
})
export class DashboardComponent {
  title = "Dashboard";
  count = 0;
}
```

### Kích hoạt Zoneless cho các Dự án Mới

```typescript
// main.ts - Zoneless Angular (v20+)
bootstrapApplication(AppComponent, {
  providers: [provideZonelessChangeDetection()],
});
```

**Lợi ích:**

- Không có các bản vá zone.js trên các API bất đồng bộ
- Kích thước gói nhỏ hơn (tiết kiệm khoảng 15KB)
- Dấu vết ngăn xếp (stack traces) sạch để gỡ lỗi
- Khả năng tương thích micro-frontend tốt hơn

---

## 2. Các hoạt động Bất đồng bộ & Thác nước (NGHIÊM TRỌNG)

### Loại bỏ việc Lấy dữ liệu Tuần tự (Sequential Data Fetching)

```typescript
// SAI - Subscriptions lồng nhau tạo ra thác nước
this.route.params.subscribe((params) => {
  // 1. Chờ params
  this.userService.getUser(params.id).subscribe((user) => {
    // 2. Chờ user
    this.postsService.getPosts(user.id).subscribe((posts) => {
      // 3. Chờ posts
    });
  });
});

// ĐÚNG - Thực thi song song với forkJoin
forkJoin({
  user: this.userService.getUser(id),
  posts: this.postsService.getPosts(id),
}).subscribe((data) => {
  // Được lấy song song
});

// ĐÚNG - Làm phẳng các cuộc gọi phụ thuộc với switchMap
this.route.params
  .pipe(
    map((p) => p.id),
    switchMap((id) => this.userService.getUser(id)),
  )
  .subscribe();
```

### Tránh Thác nước phía Máy khách trong SSR

```typescript
// ĐÚNG - Sử dụng resolvers hoặc chặn hydration cho dữ liệu quan trọng
export const route: Route = {
  path: "profile/:id",
  resolve: { data: profileResolver }, // Được lấy trên máy chủ trước khi điều hướng
  component: ProfileComponent,
};

// SAI - Component lấy dữ liệu khi khởi tạo (on init)
class ProfileComponent implements OnInit {
  ngOnInit() {
    // CHỈ bắt đầu sau khi JS tải và component được kết xuất
    this.http.get("/api/profile").subscribe();
  }
}
```

---

## 3. Tối ưu hóa Gói (NGHIÊM TRỌNG)

### Tải chậm các Tuyến đường (Lazy Load Routes)

```typescript
// ĐÚNG - Tải chậm các tuyến đường tính năng
export const routes: Routes = [
  {
    path: "admin",
    loadChildren: () =>
      import("./admin/admin.routes").then((m) => m.ADMIN_ROUTES),
  },
  {
    path: "dashboard",
    loadComponent: () =>
      import("./dashboard/dashboard.component").then(
        (m) => m.DashboardComponent,
      ),
  },
];

// SAI - Tải ngay (Eager loading) mọi thứ
import { AdminModule } from "./admin/admin.module";
export const routes: Routes = [
  { path: "admin", component: AdminComponent }, // Nằm trong gói chính (main bundle)
];
```

### Sử dụng @defer cho các Thành phần Nặng

```html
<!-- ĐÚNG - Thành phần nặng tải theo nhu cầu -->
@defer (on viewport) {
<app-analytics-chart [data]="data()" />
} @placeholder {
<div class="chart-skeleton"></div>
}

<!-- SAI - Thành phần nặng nằm trong gói ban đầu -->
<app-analytics-chart [data]="data()" />
```

### Tránh Re-exports từ Barrel File

```typescript
// SAI - Import toàn bộ barrel, phá vỡ tree-shaking
import { Button, Modal, Table } from "@shared/components";

// ĐÚNG - Import trực tiếp
import { Button } from "@shared/components/button/button.component";
import { Modal } from "@shared/components/modal/modal.component";
```

### Import Động các Thư viện Bên thứ ba

```typescript
// ĐÚNG - Tải thư viện nặng theo nhu cầu
async loadChart() {
  const { Chart } = await import('chart.js');
  this.chart = new Chart(this.canvas, config);
}

// SAI - Đóng gói Chart.js trong chunk chính
import { Chart } from 'chart.js';
```

---

## 4. Hiệu suất Kết xuất (CAO)

### Luôn sử dụng trackBy với @for

```html
<!-- ĐÚNG - Cập nhật DOM hiệu quả -->
@for (item of items(); track item.id) {
<app-item-card [item]="item" />
}

<!-- SAI - Toàn bộ danh sách kết xuất lại khi có bất kỳ thay đổi nào -->
@for (item of items(); track $index) {
<app-item-card [item]="item" />
}
```

### Sử dụng Cuộn ảo (Virtual Scrolling) cho Danh sách Lớn

```typescript
import { CdkVirtualScrollViewport, CdkFixedSizeVirtualScroll } from '@angular/cdk/scrolling';

@Component({
  imports: [CdkVirtualScrollViewport, CdkFixedSizeVirtualScroll],
  template: `
    <cdk-virtual-scroll-viewport itemSize="50" class="viewport">
      <div *cdkVirtualFor="let item of items" class="item">
        {{ item.name }}
      </div>
    </cdk-virtual-scroll-viewport>
  `
})
```

### Ưu tiên Pure Pipes hơn Methods

```typescript
// ĐÚNG - Pure pipe, được ghi nhớ (memoized)
@Pipe({ name: 'filterActive', standalone: true, pure: true })
export class FilterActivePipe implements PipeTransform {
  transform(items: Item[]): Item[] {
    return items.filter(i => i.active);
  }
}

// Template
@for (item of items() | filterActive; track item.id) { ... }

// SAI - Method được gọi mỗi khi phát hiện thay đổi
@for (item of getActiveItems(); track item.id) { ... }
```

### Sử dụng computed() cho Dữ liệu Dẫn xuất

```typescript
// ĐÚNG - Computed, được lưu trong bộ nhớ cache cho đến khi các phụ thuộc thay đổi
export class ProductStore {
  products = signal<Product[]>([]);
  filter = signal('');

  filteredProducts = computed(() => {
    const f = this.filter().toLowerCase();
    return this.products().filter(p =>
      p.name.toLowerCase().includes(f)
    );
  });
}

// SAI - Tính toán lại mỗi khi truy cập
get filteredProducts() {
  return this.products.filter(p =>
    p.name.toLowerCase().includes(this.filter)
  );
}
```

---

## 5. Kết xuất phía Máy chủ (CAO)

### Cấu hình Hydration Tăng cường

```typescript
// app.config.ts
import {
  provideClientHydration,
  withIncrementalHydration,
} from "@angular/platform-browser";

export const appConfig: ApplicationConfig = {
  providers: [
    provideClientHydration(withIncrementalHydration(), withEventReplay()),
  ],
};
```

### Trì hoãn Nội dung Không quan trọng

```html
<!-- Nội dung quan trọng trong màn hình đầu tiên (above-the-fold) -->
<app-header />
<app-hero />

<!-- Nội dung bên dưới được trì hoãn với các kích hoạt hydration -->
@defer (hydrate on viewport) {
<app-product-grid />
} @defer (hydrate on interaction) {
<app-chat-widget />
}
```

### Sử dụng TransferState cho Dữ liệu SSR

```typescript
@Injectable({ providedIn: "root" })
export class DataService {
  private http = inject(HttpClient);
  private transferState = inject(TransferState);
  private platformId = inject(PLATFORM_ID);

  getData(key: string): Observable<Data> {
    const stateKey = makeStateKey<Data>(key);

    if (isPlatformBrowser(this.platformId)) {
      const cached = this.transferState.get(stateKey, null);
      if (cached) {
        this.transferState.remove(stateKey);
        return of(cached);
      }
    }

    return this.http.get<Data>(`/api/${key}`).pipe(
      tap((data) => {
        if (isPlatformServer(this.platformId)) {
          this.transferState.set(stateKey, data);
        }
      }),
    );
  }
}
```

---

## 6. Tối ưu hóa Mẫu (TRUNG BÌNH)

### Sử dụng Cú pháp Luồng Điều khiển Mới

```html
<!-- ĐÚNG - Luồng điều khiển mới (nhanh hơn, gói nhỏ hơn) -->
@if (user()) {
<span>{{ user()!.name }}</span>
} @else {
<span>Guest</span>
} @for (item of items(); track item.id) {
<app-item [item]="item" />
} @empty {
<p>No items</p>
}

<!-- SAI - Các chỉ thị rẽ nhánh cũ -->
<span *ngIf="user; else guest">{{ user.name }}</span>
<ng-template #guest><span>Guest</span></ng-template>
```

### Tránh các Biểu thức Mẫu Phức tạp

```typescript
// ĐÚNG - Tính toán trước trong component
class Component {
  items = signal<Item[]>([]);
  sortedItems = computed(() =>
    [...this.items()].sort((a, b) => a.name.localeCompare(b.name))
  );
}

// Template
@for (item of sortedItems(); track item.id) { ... }

// SAI - Sắp xếp trong mẫu mỗi khi kết xuất
@for (item of items() | sort:'name'; track item.id) { ... }
```

---

## 7. Quản lý Trạng thái (TRUNG BÌNH)

### Sử dụng Selectors để Ngăn chặn Kết xuất lại

```typescript
// ĐÚNG - Đăng ký có chọn lọc
@Component({
  template: `<span>{{ userName() }}</span>`,
})
class HeaderComponent {
  private store = inject(Store);
  // Chỉ kết xuất lại khi userName thay đổi
  userName = this.store.selectSignal(selectUserName);
}

// SAI - Đăng ký vào toàn bộ trạng thái
@Component({
  template: `<span>{{ state().user.name }}</span>`,
})
class HeaderComponent {
  private store = inject(Store);
  // Kết xuất lại khi CÓ BẤT KỲ thay đổi trạng thái nào
  state = toSignal(this.store);
}
```

### Đặt Trạng thái cùng vị trí với Tính năng

```typescript
// ĐÚNG - Store phạm vi tính năng (feature-scoped)
@Injectable() // KHÔNG providedIn: 'root'
export class ProductStore { ... }

@Component({
  providers: [ProductStore], // Phạm vi cây component
})
export class ProductPageComponent {
  store = inject(ProductStore);
}

// SAI - Mọi thứ trong store toàn cục
@Injectable({ providedIn: 'root' })
export class GlobalStore {
  // Chứa TẤT CẢ trạng thái ứng dụng - khó để tree-shake
}
```

---

## 8. Quản lý Bộ nhớ (THẤP-TRUNG BÌNH)

### Sử dụng takeUntilDestroyed cho Subscriptions

```typescript
import { takeUntilDestroyed } from '@angular/core/rxjs-interop';

@Component({...})
export class DataComponent {
  private destroyRef = inject(DestroyRef);

  constructor() {
    this.data$.pipe(
      takeUntilDestroyed(this.destroyRef)
    ).subscribe(data => this.process(data));
  }
}

// SAI - Quản lý subscription thủ công
export class DataComponent implements OnDestroy {
  private subscription!: Subscription;

  ngOnInit() {
    this.subscription = this.data$.subscribe(...);
  }

  ngOnDestroy() {
    this.subscription.unsubscribe(); // Dễ quên
  }
}
```

### Ưu tiên Signals hơn Subscriptions

```typescript
// ĐÚNG - Không cần subscription
@Component({
  template: `<div>{{ data().name }}</div>`,
})
export class Component {
  data = toSignal(this.service.data$, { initialValue: null });
}

// SAI - Subscription thủ công
@Component({
  template: `<div>{{ data?.name }}</div>`,
})
export class Component implements OnInit, OnDestroy {
  data: Data | null = null;
  private sub!: Subscription;

  ngOnInit() {
    this.sub = this.service.data$.subscribe((d) => (this.data = d));
  }

  ngOnDestroy() {
    this.sub.unsubscribe();
  }
}
```

---

## Danh sách Kiểm tra Tham khảo Nhanh

### Component Mới

- [ ] `changeDetection: ChangeDetectionStrategy.OnPush`
- [ ] `standalone: true`
- [ ] Signals cho trạng thái (`signal()`, `input()`, `output()`)
- [ ] `inject()` cho các phụ thuộc
- [ ] `@for` với biểu thức `track`

### Xem xét Hiệu suất

- [ ] Không có methods trong templates (sử dụng pipes hoặc computed)
- [ ] Danh sách lớn được ảo hóa (virtualized)
- [ ] Các thành phần nặng được trì hoãn (deferred)
- [ ] Các tuyến đường được tải chậm (lazy-loaded)
- [ ] Thư viện bên thứ ba được import động

### Kiểm tra SSR

- [ ] Hydration được cấu hình
- [ ] Nội dung quan trọng kết xuất trước
- [ ] Nội dung không quan trọng sử dụng `@defer (hydrate on ...)`
- [ ] TransferState cho dữ liệu được lấy từ máy chủ

---

## Tài nguyên

- [Hướng dẫn Hiệu suất Angular](https://angular.dev/best-practices/performance)
- [Angular Zoneless](https://angular.dev/guide/experimental/zoneless)
- [Hướng dẫn Angular SSR](https://angular.dev/guide/ssr)
- [Tìm hiểu sâu về Phát hiện Thay đổi](https://angular.dev/guide/change-detection)
