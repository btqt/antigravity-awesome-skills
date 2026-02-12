---
name: angular
description: >-
  Chuyên gia Angular hiện đại (v20+) với kiến thức sâu rộng về Signals, Standalone Components, ứng dụng Zoneless, SSR/Hydration và các mẫu phản ứng (reactive patterns). Sử dụng CHỦ ĐỘNG cho phát triển Angular, kiến trúc thành phần (component architecture), quản lý trạng thái, tối ưu hóa hiệu suất và di chuyển sang các mẫu hiện đại.
risk: safe
source: self
---

# Chuyên gia Angular

Làm chủ phát triển Angular hiện đại với Signals, Standalone Components, ứng dụng Zoneless, SSR/Hydration và các mẫu phản ứng mới nhất.

## Khi nào nên sử dụng kỹ năng này

- Xây dựng các ứng dụng Angular mới (v20+)
- Triển khai các mẫu phản ứng dựa trên Signals
- Tạo các Standalone Components và di chuyển từ NgModules
- Cấu hình các ứng dụng Angular Zoneless
- Triển khai SSR, prerendering và hydration
- Tối ưu hóa hiệu suất Angular
- Áp dụng các mẫu Angular hiện đại và thực hành tốt nhất

## Không sử dụng kỹ năng này khi

- Di chuyển từ AngularJS (1.x) → sử dụng kỹ năng `angular-migration`
- Làm việc với các ứng dụng Angular cũ không thể nâng cấp
- Các vấn đề TypeScript chung → sử dụng kỹ năng `typescript-expert`

## Hướng dẫn

1. Đánh giá phiên bản Angular và cấu trúc dự án
2. Áp dụng các mẫu hiện đại (Signals, Standalone, Zoneless)
3. Triển khai với định kiểu và tính phản ứng phù hợp
4. Xác thực bằng bản dựng (build) và kiểm thử (tests)

## An toàn

- Luôn kiểm thử các thay đổi trong môi trường phát triển trước khi đưa vào sản xuất
- Di chuyển dần dần cho các ứng dụng hiện có (không tái cấu trúc ồ ạt)
- Giữ khả năng tương thích ngược trong quá trình chuyển đổi

---

## Dòng thời gian phiên bản Angular

| Phiên bản      | Phát hành | Các tính năng chính                                                             |
| -------------- | --------- | ------------------------------------------------------------------------------- |
| **Angular 20** | Q2 2025   | Signals ổn định, Zoneless ổn định, Hydration tăng cường (Incremental hydration) |
| **Angular 21** | Q4 2025   | Mặc định ưu tiên Signals (Signals-first default), SSR nâng cao                  |
| **Angular 22** | Q2 2026   | Signal Forms, Các thành phần không có Selector (Selectorless components)        |

---

## 1. Signals: Nguyên thủy Phản ứng Mới (The New Reactive Primitive)

Signals là hệ thống phản ứng chi tiết (fine-grained reactivity system) của Angular, thay thế việc phát hiện thay đổi dựa trên zone.js.

### Các khái niệm cốt lõi

```typescript
import { signal, computed, effect } from "@angular/core";

// Signal có thể ghi (Writable signal)
const count = signal(0);

// Đọc giá trị
console.log(count()); // 0

// Cập nhật giá trị
count.set(5); // Đặt trực tiếp
count.update((v) => v + 1); // Cập nhật bằng hàm

// Computed (derived) signal - Signal được tính toán
const doubled = computed(() => count() * 2);

// Effect (tác dụng phụ)
effect(() => {
  console.log(`Count changed to: ${count()}`);
});
```

### Inputs và Outputs dựa trên Signal

```typescript
import { Component, input, output, model } from "@angular/core";

@Component({
  selector: "app-user-card",
  standalone: true,
  template: `
    <div class="card">
      <h3>{{ name() }}</h3>
      <span>{{ role() }}</span>
      <button (click)="select.emit(id())">Select</button>
    </div>
  `,
})
export class UserCardComponent {
  // Signal inputs (chỉ đọc)
  id = input.required<string>();
  name = input.required<string>();
  role = input<string>("User"); // Có giá trị mặc định

  // Output
  select = output<string>();

  // Ràng buộc hai chiều (model)
  isSelected = model(false);
}

// Sử dụng:
// <app-user-card [id]="'123'" [name]="'John'" [(isSelected)]="selected" />
```

### Truy vấn Signal (ViewChild/ContentChild)

```typescript
import {
  Component,
  viewChild,
  viewChildren,
  contentChild,
} from "@angular/core";

@Component({
  selector: "app-container",
  standalone: true,
  template: `
    <input #searchInput />
    <app-item *ngFor="let item of items()" />
  `,
})
export class ContainerComponent {
  // Truy vấn dựa trên Signal
  searchInput = viewChild<ElementRef>("searchInput");
  items = viewChildren(ItemComponent);
  projectedContent = contentChild(HeaderDirective);

  focusSearch() {
    this.searchInput()?.nativeElement.focus();
  }
}
```

### Khi nào dùng Signals so với RxJS

| Trường hợp sử dụng            | Signals         | RxJS                            |
| ----------------------------- | --------------- | ------------------------------- |
| Trạng thái thành phần cục bộ  | ✅ Ưu tiên      | Quá mức cần thiết (Overkill)    |
| Giá trị dẫn xuất/tính toán    | ✅ `computed()` | `combineLatest` hoạt động được  |
| Tác dụng phụ (Side effects)   | ✅ `effect()`   | Toán tử `tap`                   |
| Yêu cầu HTTP                  | ❌              | ✅ HttpClient trả về Observable |
| Luồng sự kiện (Event streams) | ❌              | ✅ `fromEvent`, toán tử         |
| Luồng bất đồng bộ phức tạp    | ❌              | ✅ `switchMap`, `mergeMap`      |

---

## 2. Standalone Components (Thành phần Độc lập)

Standalone components là các thành phần tự chứa và không yêu cầu khai báo NgModule.

### Tạo Standalone Components

```typescript
import { Component } from "@angular/core";
import { CommonModule } from "@angular/common";
import { RouterLink } from "@angular/router";

@Component({
  selector: "app-header",
  standalone: true,
  imports: [CommonModule, RouterLink], // Import trực tiếp
  template: `
    <header>
      <a routerLink="/">Home</a>
      <a routerLink="/about">About</a>
    </header>
  `,
})
export class HeaderComponent {}
```

### Khởi động (Bootstrapping) mà không cần NgModule

```typescript
// main.ts
import { bootstrapApplication } from "@angular/platform-browser";
import { provideRouter } from "@angular/router";
import { provideHttpClient } from "@angular/common/http";
import { AppComponent } from "./app/app.component";
import { routes } from "./app/app.routes";

bootstrapApplication(AppComponent, {
  providers: [provideRouter(routes), provideHttpClient()],
});
```

### Tải chậm (Lazy Loading) Standalone Components

```typescript
// app.routes.ts
import { Routes } from "@angular/router";

export const routes: Routes = [
  {
    path: "dashboard",
    loadComponent: () =>
      import("./dashboard/dashboard.component").then(
        (m) => m.DashboardComponent,
      ),
  },
  {
    path: "admin",
    loadChildren: () =>
      import("./admin/admin.routes").then((m) => m.ADMIN_ROUTES),
  },
];
```

---

## 3. Zoneless Angular (Angular Không Vùng)

Các ứng dụng Zoneless không sử dụng zone.js, giúp cải thiện hiệu suất và gỡ lỗi dễ dàng hơn.

### Kích hoạt Chế độ Zoneless

```typescript
// main.ts
import { bootstrapApplication } from "@angular/platform-browser";
import { provideZonelessChangeDetection } from "@angular/core";
import { AppComponent } from "./app/app.component";

bootstrapApplication(AppComponent, {
  providers: [provideZonelessChangeDetection()],
});
```

### Các mẫu Thành phần Zoneless

```typescript
import { Component, signal, ChangeDetectionStrategy } from "@angular/core";

@Component({
  selector: "app-counter",
  standalone: true,
  changeDetection: ChangeDetectionStrategy.OnPush,
  template: `
    <div>Count: {{ count() }}</div>
    <button (click)="increment()">+</button>
  `,
})
export class CounterComponent {
  count = signal(0);

  increment() {
    this.count.update((v) => v + 1);
    // Không cần zone.js - Signal kích hoạt phát hiện thay đổi
  }
}
```

### Lợi ích chính của Zoneless

- **Hiệu suất**: Không có các bản vá zone.js trên các API bất đồng bộ
- **Gỡ lỗi**: Dấu vết ngăn xếp (stack traces) sạch sẽ không có các trình bao bọc của zone
- **Kích thước gói (Bundle size)**: Nhỏ hơn vì không có zone.js (tiết kiệm khoảng 15KB)
- **Khả năng tương tác**: Tốt hơn với Web Components và micro-frontends

---

## 4. Kết xuất phía máy chủ (SSR) & Hydration

### Thiết lập SSR với Angular CLI

```bash
ng add @angular/ssr
```

### Cấu hình Hydration

```typescript
// app.config.ts
import { ApplicationConfig } from "@angular/core";
import {
  provideClientHydration,
  withEventReplay,
} from "@angular/platform-browser";

export const appConfig: ApplicationConfig = {
  providers: [provideClientHydration(withEventReplay())],
};
```

### Hydration Tăng cường (v20+)

```typescript
import { Component } from "@angular/core";

@Component({
  selector: "app-page",
  standalone: true,
  template: `
    <app-hero />

    @defer (hydrate on viewport) {
      <app-comments />
    }

    @defer (hydrate on interaction) {
      <app-chat-widget />
    }
  `,
})
export class PageComponent {}
```

### Các kích hoạt Hydration (Hydration Triggers)

| Kích hoạt        | Khi nào sử dụng                                  |
| ---------------- | ------------------------------------------------ |
| `on idle`        | Ưu tiên thấp, hydrate khi trình duyệt nhàn rỗi   |
| `on viewport`    | Hydrate khi phần tử đi vào khung nhìn (viewport) |
| `on interaction` | Hydrate khi người dùng tương tác lần đầu tiên    |
| `on hover`       | Hydrate khi người dùng di chuột qua              |
| `on timer(ms)`   | Hydrate sau một khoảng thời gian trễ xác định    |

---

## 5. Các mẫu Định tuyến Hiện đại

### Functional Route Guards

```typescript
// auth.guard.ts
import { inject } from "@angular/core";
import { Router, CanActivateFn } from "@angular/router";
import { AuthService } from "./auth.service";

export const authGuard: CanActivateFn = (route, state) => {
  const auth = inject(AuthService);
  const router = inject(Router);

  if (auth.isAuthenticated()) {
    return true;
  }

  return router.createUrlTree(["/login"], {
    queryParams: { returnUrl: state.url },
  });
};

// Sử dụng trong routes
export const routes: Routes = [
  {
    path: "dashboard",
    loadComponent: () => import("./dashboard.component"),
    canActivate: [authGuard],
  },
];
```

### Route-Level Data Resolvers

```typescript
import { inject } from '@angular/core';
import { ResolveFn } from '@angular/router';
import { UserService } from './user.service';
import { User } from './user.model';

export const userResolver: ResolveFn<User> = (route) => {
  const userService = inject(UserService);
  return userService.getUser(route.paramMap.get('id')!);
};

// Trong routes
{
  path: 'user/:id',
  loadComponent: () => import('./user.component'),
  resolve: { user: userResolver }
}

// Trong component
export class UserComponent {
  private route = inject(ActivatedRoute);
  user = toSignal(this.route.data.pipe(map(d => d['user'])));
}
```

---

## 6. Các mẫu Dependency Injection

### Hàm inject() Hiện đại

```typescript
import { Component, inject } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { UserService } from './user.service';

@Component({...})
export class UserComponent {
  // inject() hiện đại - không cần constructor
  private http = inject(HttpClient);
  private userService = inject(UserService);

  // Hoạt động trong bất kỳ ngữ cảnh injection nào
  users = toSignal(this.userService.getUsers());
}
```

### Injection Tokens cho Cấu hình

```typescript
import { InjectionToken, inject } from "@angular/core";

// Định nghĩa token
export const API_BASE_URL = new InjectionToken<string>("API_BASE_URL");

// Cung cấp trong config
bootstrapApplication(AppComponent, {
  providers: [{ provide: API_BASE_URL, useValue: "https://api.example.com" }],
});

// Inject trong service
@Injectable({ providedIn: "root" })
export class ApiService {
  private baseUrl = inject(API_BASE_URL);

  get(endpoint: string) {
    return this.http.get(`${this.baseUrl}/${endpoint}`);
  }
}
```

---

## 7. Thành phần & Khả năng Tái sử dụng (Component Composition & Reusability)

### Content Projection (Slots)

```typescript
@Component({
  selector: 'app-card',
  template: `
    <div class="card">
      <div class="header">
        <!-- Chọn theo thuộc tính -->
        <ng-content select="[card-header]"></ng-content>
      </div>
      <div class="body">
        <!-- Slot mặc định -->
        <ng-content></ng-content>
      </div>
    </div>
  `
})
export class CardComponent {}

// Sử dụng
<app-card>
  <h3 card-header>Title</h3>
  <p>Body content</p>
</app-card>
```

### Host Directives (Composition)

```typescript
// Các hành vi tái sử dụng được mà không cần kế thừa
@Directive({
  standalone: true,
  selector: '[appTooltip]',
  inputs: ['tooltip'] // Signal input alias
})
export class TooltipDirective { ... }

@Component({
  selector: 'app-button',
  standalone: true,
  hostDirectives: [
    {
      directive: TooltipDirective,
      inputs: ['tooltip: title'] // Ánh xạ input
    }
  ],
  template: `<ng-content />`
})
export class ButtonComponent {}
```

---

## 8. Các mẫu Quản lý Trạng thái

### Dịch vụ Trạng thái Dựa trên Signal (Signal-Based State Service)

```typescript
import { Injectable, signal, computed } from "@angular/core";

interface AppState {
  user: User | null;
  theme: "light" | "dark";
  notifications: Notification[];
}

@Injectable({ providedIn: "root" })
export class StateService {
  // Các writable signals riêng tư
  private _user = signal<User | null>(null);
  private _theme = signal<"light" | "dark">("light");
  private _notifications = signal<Notification[]>([]);

  // Computed công khai chỉ đọc
  readonly user = computed(() => this._user());
  readonly theme = computed(() => this._theme());
  readonly notifications = computed(() => this._notifications());
  readonly unreadCount = computed(
    () => this._notifications().filter((n) => !n.read).length,
  );

  // Actions
  setUser(user: User | null) {
    this._user.set(user);
  }

  toggleTheme() {
    this._theme.update((t) => (t === "light" ? "dark" : "light"));
  }

  addNotification(notification: Notification) {
    this._notifications.update((n) => [...n, notification]);
  }
}
```

### Mẫu Component Store với Signals

```typescript
import { Injectable, signal, computed, inject } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { toSignal } from "@angular/core/rxjs-interop";

@Injectable()
export class ProductStore {
  private http = inject(HttpClient);

  // Trạng thái
  private _products = signal<Product[]>([]);
  private _loading = signal(false);
  private _filter = signal("");

  // Selectors
  readonly products = computed(() => this._products());
  readonly loading = computed(() => this._loading());
  readonly filteredProducts = computed(() => {
    const filter = this._filter().toLowerCase();
    return this._products().filter((p) =>
      p.name.toLowerCase().includes(filter),
    );
  });

  // Actions
  loadProducts() {
    this._loading.set(true);
    this.http.get<Product[]>("/api/products").subscribe({
      next: (products) => {
        this._products.set(products);
        this._loading.set(false);
      },
      error: () => this._loading.set(false),
    });
  }

  setFilter(filter: string) {
    this._filter.set(filter);
  }
}
```

---

## 9. Biểu mẫu với Signals (Sắp ra mắt trong v22+)

### Reactive Forms Hiện tại

```typescript
import { Component, inject } from "@angular/core";
import { FormBuilder, Validators, ReactiveFormsModule } from "@angular/forms";

@Component({
  selector: "app-user-form",
  standalone: true,
  imports: [ReactiveFormsModule],
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <input formControlName="name" placeholder="Name" />
      <input formControlName="email" type="email" placeholder="Email" />
      <button [disabled]="form.invalid">Submit</button>
    </form>
  `,
})
export class UserFormComponent {
  private fb = inject(FormBuilder);

  form = this.fb.group({
    name: ["", Validators.required],
    email: ["", [Validators.required, Validators.email]],
  });

  onSubmit() {
    if (this.form.valid) {
      console.log(this.form.value);
    }
  }
}
```

### Các mẫu Form Nhận biết Signal (Xem trước)

```typescript
// API Signal Forms Tương lai (đang thử nghiệm)
import { Component, signal } from '@angular/core';

@Component({...})
export class SignalFormComponent {
  name = signal('');
  email = signal('');

  // Xác thực được tính toán (Computed validation)
  isValid = computed(() =>
    this.name().length > 0 &&
    this.email().includes('@')
  );

  submit() {
    if (this.isValid()) {
      console.log({ name: this.name(), email: this.email() });
    }
  }
}
```

---

## 10. Tối ưu hóa Hiệu suất

### Chiến lược Phát hiện Thay đổi

```typescript
@Component({
  changeDetection: ChangeDetectionStrategy.OnPush,
  // Chỉ kiểm tra khi:
  // 1. Input signal/tham chiếu thay đổi
  // 2. Trình xử lý sự kiện chạy
  // 3. Async pipe phát ra tín hiệu
  // 4. Giá trị Signal thay đổi
})
```

### Defer Blocks để Tải chậm (Lazy Loading)

```typescript
@Component({
  template: `
    <!-- Tải ngay lập tức -->
    <app-header />

    <!-- Tải chậm khi hiển thị -->
    @defer (on viewport) {
      <app-heavy-chart />
    } @placeholder {
      <div class="skeleton" />
    } @loading (minimum 200ms) {
      <app-spinner />
    } @error {
      <p>Failed to load chart</p>
    }
  `
})
```

### NgOptimizedImage

```typescript
import { NgOptimizedImage } from '@angular/common';

@Component({
  imports: [NgOptimizedImage],
  template: `
    <img
      ngSrc="hero.jpg"
      width="800"
      height="600"
      priority
    />

    <img
      ngSrc="thumbnail.jpg"
      width="200"
      height="150"
      loading="lazy"
      placeholder="blur"
    />
  `
})
```

---

## 11. Kiểm thử Angular Hiện đại

### Kiểm thử Các thành phần Signal

```typescript
import { ComponentFixture, TestBed } from "@angular/core/testing";
import { CounterComponent } from "./counter.component";

describe("CounterComponent", () => {
  let component: CounterComponent;
  let fixture: ComponentFixture<CounterComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [CounterComponent], // Standalone import
    }).compileComponents();

    fixture = TestBed.createComponent(CounterComponent);
    component = fixture.componentInstance;
    fixture.detectChanges();
  });

  it("should increment count", () => {
    expect(component.count()).toBe(0);

    component.increment();

    expect(component.count()).toBe(1);
  });

  it("should update DOM on signal change", () => {
    component.count.set(5);
    fixture.detectChanges();

    const el = fixture.nativeElement.querySelector(".count");
    expect(el.textContent).toContain("5");
  });
});
```

### Kiểm thử với Signal Inputs

```typescript
import { ComponentFixture, TestBed } from "@angular/core/testing";
import { ComponentRef } from "@angular/core";
import { UserCardComponent } from "./user-card.component";

describe("UserCardComponent", () => {
  let fixture: ComponentFixture<UserCardComponent>;
  let componentRef: ComponentRef<UserCardComponent>;

  beforeEach(async () => {
    await TestBed.configureTestingModule({
      imports: [UserCardComponent],
    }).compileComponents();

    fixture = TestBed.createComponent(UserCardComponent);
    componentRef = fixture.componentRef;

    // Đặt signal inputs thông qua setInput
    componentRef.setInput("id", "123");
    componentRef.setInput("name", "John Doe");

    fixture.detectChanges();
  });

  it("should display user name", () => {
    const el = fixture.nativeElement.querySelector("h3");
    expect(el.textContent).toContain("John Doe");
  });
});
```

---

## Tóm tắt Thực hành Tốt nhất

| Mẫu                    | ✅ Nên làm                            | ❌ Không nên                            |
| ---------------------- | ------------------------------------- | --------------------------------------- |
| **Trạng thái**         | Sử dụng Signals cho trạng thái cục bộ | Lạm dụng RxJS cho trạng thái đơn giản   |
| **Thành phần**         | Standalone với import trực tiếp       | SharedModules cồng kềnh                 |
| **Phát hiện thay đổi** | OnPush + Signals                      | CD mặc định ở mọi nơi                   |
| **Tải chậm**           | `@defer` và `loadComponent`           | Tải ngay (Eager load) mọi thứ           |
| **DI**                 | Hàm `inject()`                        | Constructor injection (dài dòng)        |
| **Inputs**             | Hàm `input()` signal                  | Decorator `@Input()` (cũ)               |
| **Zoneless**           | Kích hoạt cho dự án mới               | Ép buộc trên dự án cũ mà không kiểm thử |

---

## Tài nguyên

- [Tài liệu Angular.dev](https://angular.dev)
- [Hướng dẫn Angular Signals](https://angular.dev/guide/signals)
- [Hướng dẫn Angular SSR](https://angular.dev/guide/ssr)
- [Hướng dẫn Cập nhật Angular](https://angular.dev/update-guide)
- [Blog Angular](https://blog.angular.dev)

---

## Khắc phục sự cố thường gặp

| Vấn đề                               | Giải pháp                                                          |
| ------------------------------------ | ------------------------------------------------------------------ |
| Signal không cập nhật UI             | Đảm bảo dùng `OnPush` + gọi signal như một hàm `count()`           |
| Hydration không khớp                 | Kiểm tra tính nhất quán nội dung giữa máy chủ/máy khách            |
| Phụ thuộc vòng (Circular dependency) | Sử dụng `inject()` với `forwardRef`                                |
| Zoneless không phát hiện thay đổi    | Kích hoạt qua cập nhật signal, không phải qua biến đổi (mutations) |
| SSR fetch thất bại                   | Sử dụng `TransferState` hoặc `withFetch()`                         |
