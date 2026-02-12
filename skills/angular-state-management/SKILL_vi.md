---
name: angular-state-management
description: Làm chủ quản lý trạng thái Angular hiện đại với Signals, NgRx và RxJS. Sử dụng khi thiết lập trạng thái toàn cục, quản lý kho lưu trữ thành phần (component stores), lựa chọn giữa các giải pháp trạng thái hoặc di chuyển từ các mẫu cũ.
risk: safe
source: self
---

# Quản lý Trạng thái Angular

Hướng dẫn toàn diện về các mẫu quản lý trạng thái Angular hiện đại, từ trạng thái cục bộ dựa trên Signal đến các kho lưu trữ toàn cục và đồng bộ hóa trạng thái máy chủ.

## Khi nào nên sử dụng kỹ năng này

- Thiết lập quản lý trạng thái toàn cục trong Angular
- Lựa chọn giữa Signals, NgRx hoặc Akita
- Quản lý kho lưu trữ cấp thành phần (component-level stores)
- Triển khai cập nhật lạc quan (optimistic updates)
- Gỡ lỗi các vấn đề liên quan đến trạng thái
- Di chuyển từ các mẫu trạng thái cũ

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến quản lý trạng thái Angular
- Bạn cần quản lý trạng thái React → sử dụng kỹ năng `react-state-management`

---

## Các Khái niệm Cốt lõi

### Các Danh mục Trạng thái

| Loại                    | Mô tả                                | Giải pháp                        |
| ----------------------- | ------------------------------------ | -------------------------------- |
| **Trạng thái Cục bộ**   | Cụ thể cho thành phần, trạng thái UI | Signals, `signal()`              |
| **Trạng thái Chia sẻ**  | Giữa các thành phần liên quan        | Dịch vụ Signal (Signal services) |
| **Trạng thái Toàn cục** | Toàn ứng dụng, phức tạp              | NgRx, Akita, Elf                 |
| **Trạng thái Máy chủ**  | Dữ liệu từ xa, bộ nhớ đệm            | NgRx Query, RxAngular            |
| **Trạng thái URL**      | Tham số định tuyến                   | ActivatedRoute                   |
| **Trạng thái Biểu mẫu** | Giá trị đầu vào, xác thực            | Reactive Forms                   |

### Tiêu chí Lựa chọn

```
Ứng dụng nhỏ, trạng thái đơn giản → Dịch vụ Signal
Ứng dụng vừa, trạng thái vừa phải → Kho lưu trữ Thành phần (Component Stores)
Ứng dụng lớn, trạng thái phức tạp → NgRx Store
Tương tác máy chủ nhiều → NgRx Query + Dịch vụ Signal
Cập nhật thời gian thực → RxAngular + Signals
```

---

## Bắt đầu Nhanh: Trạng thái Dựa trên Signal

### Mẫu 1: Dịch vụ Signal Đơn giản

```typescript
// services/counter.service.ts
import { Injectable, signal, computed } from "@angular/core";

@Injectable({ providedIn: "root" })
export class CounterService {
  // Writable signals riêng tư
  private _count = signal(0);

  // Chỉ đọc công khai
  readonly count = this._count.asReadonly();
  readonly doubled = computed(() => this._count() * 2);
  readonly isPositive = computed(() => this._count() > 0);

  increment() {
    this._count.update((v) => v + 1);
  }

  decrement() {
    this._count.update((v) => v - 1);
  }

  reset() {
    this._count.set(0);
  }
}

// Sử dụng trong thành phần
@Component({
  template: `
    <p>Count: {{ counter.count() }}</p>
    <p>Doubled: {{ counter.doubled() }}</p>
    <button (click)="counter.increment()">+</button>
  `,
})
export class CounterComponent {
  counter = inject(CounterService);
}
```

### Mẫu 2: Kho lưu trữ Signal Tính năng (Feature Signal Store)

```typescript
// stores/user.store.ts
import { Injectable, signal, computed, inject } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { toSignal } from "@angular/core/rxjs-interop";

interface User {
  id: string;
  name: string;
  email: string;
}

interface UserState {
  user: User | null;
  loading: boolean;
  error: string | null;
}

@Injectable({ providedIn: "root" })
export class UserStore {
  private http = inject(HttpClient);

  // Các tín hiệu trạng thái
  private _user = signal<User | null>(null);
  private _loading = signal(false);
  private _error = signal<string | null>(null);

  // Selectors (computed chỉ đọc)
  readonly user = computed(() => this._user());
  readonly loading = computed(() => this._loading());
  readonly error = computed(() => this._error());
  readonly isAuthenticated = computed(() => this._user() !== null);
  readonly displayName = computed(() => this._user()?.name ?? "Guest");

  // Actions
  async loadUser(id: string) {
    this._loading.set(true);
    this._error.set(null);

    try {
      const user = await fetch(`/api/users/${id}`).then((r) => r.json());
      this._user.set(user);
    } catch (e) {
      this._error.set("Failed to load user");
    } finally {
      this._loading.set(false);
    }
  }

  updateUser(updates: Partial<User>) {
    this._user.update((user) => (user ? { ...user, ...updates } : null));
  }

  logout() {
    this._user.set(null);
    this._error.set(null);
  }
}
```

### Mẫu 3: SignalStore (NgRx Signals)

```typescript
// stores/products.store.ts
import {
  signalStore,
  withState,
  withMethods,
  withComputed,
  patchState,
} from "@ngrx/signals";
import { inject } from "@angular/core";
import { ProductService } from "./product.service";

interface ProductState {
  products: Product[];
  loading: boolean;
  filter: string;
}

const initialState: ProductState = {
  products: [],
  loading: false,
  filter: "",
};

export const ProductStore = signalStore(
  { providedIn: "root" },

  withState(initialState),

  withComputed((store) => ({
    filteredProducts: computed(() => {
      const filter = store.filter().toLowerCase();
      return store
        .products()
        .filter((p) => p.name.toLowerCase().includes(filter));
    }),
    totalCount: computed(() => store.products().length),
  })),

  withMethods((store, productService = inject(ProductService)) => ({
    async loadProducts() {
      patchState(store, { loading: true });

      try {
        const products = await productService.getAll();
        patchState(store, { products, loading: false });
      } catch {
        patchState(store, { loading: false });
      }
    },

    setFilter(filter: string) {
      patchState(store, { filter });
    },

    addProduct(product: Product) {
      patchState(store, ({ products }) => ({
        products: [...products, product],
      }));
    },
  })),
);

// Sử dụng
@Component({
  template: `
    <input (input)="store.setFilter($event.target.value)" />
    @if (store.loading()) {
      <app-spinner />
    } @else {
      @for (product of store.filteredProducts(); track product.id) {
        <app-product-card [product]="product" />
      }
    }
  `,
})
export class ProductListComponent {
  store = inject(ProductStore);

  ngOnInit() {
    this.store.loadProducts();
  }
}
```

---

## NgRx Store (Trạng thái Toàn cục)

### Thiết lập

```typescript
// store/app.state.ts
import { ActionReducerMap } from "@ngrx/store";

export interface AppState {
  user: UserState;
  cart: CartState;
}

export const reducers: ActionReducerMap<AppState> = {
  user: userReducer,
  cart: cartReducer,
};

// main.ts
bootstrapApplication(AppComponent, {
  providers: [
    provideStore(reducers),
    provideEffects([UserEffects, CartEffects]),
    provideStoreDevtools({ maxAge: 25 }),
  ],
});
```

### Mẫu Feature Slice

```typescript
// store/user/user.actions.ts
import { createActionGroup, props, emptyProps } from "@ngrx/store";

export const UserActions = createActionGroup({
  source: "User",
  events: {
    "Load User": props<{ userId: string }>(),
    "Load User Success": props<{ user: User }>(),
    "Load User Failure": props<{ error: string }>(),
    "Update User": props<{ updates: Partial<User> }>(),
    Logout: emptyProps(),
  },
});
```

```typescript
// store/user/user.reducer.ts
import { createReducer, on } from "@ngrx/store";
import { UserActions } from "./user.actions";

export interface UserState {
  user: User | null;
  loading: boolean;
  error: string | null;
}

const initialState: UserState = {
  user: null,
  loading: false,
  error: null,
};

export const userReducer = createReducer(
  initialState,

  on(UserActions.loadUser, (state) => ({
    ...state,
    loading: true,
    error: null,
  })),

  on(UserActions.loadUserSuccess, (state, { user }) => ({
    ...state,
    user,
    loading: false,
  })),

  on(UserActions.loadUserFailure, (state, { error }) => ({
    ...state,
    loading: false,
    error,
  })),

  on(UserActions.logout, () => initialState),
);
```

```typescript
// store/user/user.selectors.ts
import { createFeatureSelector, createSelector } from "@ngrx/store";
import { UserState } from "./user.reducer";

export const selectUserState = createFeatureSelector<UserState>("user");

export const selectUser = createSelector(
  selectUserState,
  (state) => state.user,
);

export const selectUserLoading = createSelector(
  selectUserState,
  (state) => state.loading,
);

export const selectIsAuthenticated = createSelector(
  selectUser,
  (user) => user !== null,
);
```

```typescript
// store/user/user.effects.ts
import { Injectable, inject } from "@angular/core";
import { Actions, createEffect, ofType } from "@ngrx/effects";
import { switchMap, map, catchError, of } from "rxjs";

@Injectable()
export class UserEffects {
  private actions$ = inject(Actions);
  private userService = inject(UserService);

  loadUser$ = createEffect(() =>
    this.actions$.pipe(
      ofType(UserActions.loadUser),
      switchMap(({ userId }) =>
        this.userService.getUser(userId).pipe(
          map((user) => UserActions.loadUserSuccess({ user })),
          catchError((error) =>
            of(UserActions.loadUserFailure({ error: error.message })),
          ),
        ),
      ),
    ),
  );
}
```

### Sử dụng trong Thành phần

```typescript
@Component({
  template: `
    @if (loading()) {
      <app-spinner />
    } @else if (user(); as user) {
      <h1>Welcome, {{ user.name }}</h1>
      <button (click)="logout()">Logout</button>
    }
  `,
})
export class HeaderComponent {
  private store = inject(Store);

  user = this.store.selectSignal(selectUser);
  loading = this.store.selectSignal(selectUserLoading);

  logout() {
    this.store.dispatch(UserActions.logout());
  }
}
```

---

## Các mẫu Dựa trên RxJS

### Kho lưu trữ Thành phần (Component Store - Trạng thái Tính năng Cục bộ)

```typescript
// stores/todo.store.ts
import { Injectable } from "@angular/core";
import { ComponentStore } from "@ngrx/component-store";
import { switchMap, tap, catchError, EMPTY } from "rxjs";

interface TodoState {
  todos: Todo[];
  loading: boolean;
}

@Injectable()
export class TodoStore extends ComponentStore<TodoState> {
  constructor(private todoService: TodoService) {
    super({ todos: [], loading: false });
  }

  // Selectors
  readonly todos$ = this.select((state) => state.todos);
  readonly loading$ = this.select((state) => state.loading);
  readonly completedCount$ = this.select(
    this.todos$,
    (todos) => todos.filter((t) => t.completed).length,
  );

  // Updaters
  readonly addTodo = this.updater((state, todo: Todo) => ({
    ...state,
    todos: [...state.todos, todo],
  }));

  readonly toggleTodo = this.updater((state, id: string) => ({
    ...state,
    todos: state.todos.map((t) =>
      t.id === id ? { ...t, completed: !t.completed } : t,
    ),
  }));

  // Effects
  readonly loadTodos = this.effect<void>((trigger$) =>
    trigger$.pipe(
      tap(() => this.patchState({ loading: true })),
      switchMap(() =>
        this.todoService.getAll().pipe(
          tap({
            next: (todos) => this.patchState({ todos, loading: false }),
            error: () => this.patchState({ loading: false }),
          }),
          catchError(() => EMPTY),
        ),
      ),
    ),
  );
}
```

---

## Trạng thái Máy chủ với Signals

### Mẫu HTTP + Signals

```typescript
// services/api.service.ts
import { Injectable, signal, inject } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { toSignal } from "@angular/core/rxjs-interop";

interface ApiState<T> {
  data: T | null;
  loading: boolean;
  error: string | null;
}

@Injectable({ providedIn: "root" })
export class ProductApiService {
  private http = inject(HttpClient);

  private _state = signal<ApiState<Product[]>>({
    data: null,
    loading: false,
    error: null,
  });

  readonly products = computed(() => this._state().data ?? []);
  readonly loading = computed(() => this._state().loading);
  readonly error = computed(() => this._state().error);

  async fetchProducts(): Promise<void> {
    this._state.update((s) => ({ ...s, loading: true, error: null }));

    try {
      const data = await firstValueFrom(
        this.http.get<Product[]>("/api/products"),
      );
      this._state.update((s) => ({ ...s, data, loading: false }));
    } catch (e) {
      this._state.update((s) => ({
        ...s,
        loading: false,
        error: "Failed to fetch products",
      }));
    }
  }

  // Cập nhật lạc quan (Optimistic update)
  async deleteProduct(id: string): Promise<void> {
    const previousData = this._state().data;

    // Xóa lạc quan
    this._state.update((s) => ({
      ...s,
      data: s.data?.filter((p) => p.id !== id) ?? null,
    }));

    try {
      await firstValueFrom(this.http.delete(`/api/products/${id}`));
    } catch {
      // Khôi phục khi lỗi
      this._state.update((s) => ({ ...s, data: previousData }));
    }
  }
}
```

---

## Thực hành Tốt nhất

### Nên làm

| Thực hành                                 | Tại sao                                     |
| ----------------------------------------- | ------------------------------------------- |
| Sử dụng Signals cho trạng thái cục bộ     | Đơn giản, phản ứng, không cần subscriptions |
| Sử dụng `computed()` cho dữ liệu dẫn xuất | Tự động cập nhật, được ghi nhớ (memoized)   |
| Đặt trạng thái cùng vị trí với tính năng  | Dễ bảo trì hơn                              |
| Sử dụng NgRx cho các luồng phức tạp       | Actions, effects, devtools                  |
| Ưu tiên `inject()` hơn constructor        | Sạch hơn, hoạt động trong factories         |

### Không nên làm

| Phản mẫu (Anti-Pattern)                      | Thay vào đó                                       |
| -------------------------------------------- | ------------------------------------------------- |
| Lưu trữ dữ liệu dẫn xuất                     | Sử dụng `computed()`                              |
| Biến đổi trực tiếp signals                   | Sử dụng `set()` hoặc `update()`                   |
| Toàn cục hóa trạng thái quá mức              | Giữ cục bộ khi có thể                             |
| Trộn lẫn RxJS và Signals lộn xộn             | Chọn chính, bắc cầu với `toSignal`/`toObservable` |
| Subscribe trong thành phần để lấy trạng thái | Sử dụng template với signals                      |

---

## Lộ trình Di chuyển

### Từ BehaviorSubject sang Signals

```typescript
// Trước: Dựa trên RxJS
@Injectable({ providedIn: "root" })
export class OldUserService {
  private userSubject = new BehaviorSubject<User | null>(null);
  user$ = this.userSubject.asObservable();

  setUser(user: User) {
    this.userSubject.next(user);
  }
}

// Sau: Dựa trên Signal
@Injectable({ providedIn: "root" })
export class UserService {
  private _user = signal<User | null>(null);
  readonly user = this._user.asReadonly();

  setUser(user: User) {
    this._user.set(user);
  }
}
```

### Bắc cầu giữa Signals và RxJS

```typescript
import { toSignal, toObservable } from '@angular/core/rxjs-interop';

// Observable → Signal
@Component({...})
export class ExampleComponent {
  private route = inject(ActivatedRoute);

  // Chuyển đổi Observable sang Signal
  userId = toSignal(
    this.route.params.pipe(map(p => p['id'])),
    { initialValue: '' }
  );
}

// Signal → Observable
export class DataService {
  private filter = signal('');

  // Chuyển đổi Signal sang Observable
  filter$ = toObservable(this.filter);

  filteredData$ = this.filter$.pipe(
    debounceTime(300),
    switchMap(filter => this.http.get(`/api/data?q=${filter}`))
  );
}
```

---

## Tài nguyên

- [Hướng dẫn Angular Signals](https://angular.dev/guide/signals)
- [Tài liệu NgRx](https://ngrx.io/)
- [NgRx SignalStore](https://ngrx.io/guide/signals)
- [RxAngular](https://www.rx-angular.io/)
