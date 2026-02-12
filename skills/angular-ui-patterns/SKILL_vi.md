---
name: angular-ui-patterns
description: Các mẫu UI Angular hiện đại cho các trạng thái đang tải, xử lý lỗi và hiển thị dữ liệu. Sử dụng khi xây dựng các thành phần UI, xử lý dữ liệu bất đồng bộ hoặc quản lý trạng thái thành phần.
risk: safe
source: self
---

# Các Mẫu UI Angular

## Các Nguyên tắc Cốt lõi

1. **Không bao giờ hiển thị UI cũ** - Chỉ hiển thị trạng thái đang tải khi thực sự đang tải
2. **Luôn hiển thị lỗi** - Người dùng phải biết khi nào có sự cố
3. **Cập nhật lạc quan (Optimistic updates)** - Làm cho UI có cảm giác tức thì
4. **Tiết lộ lũy tiến (Progressive disclosure)** - Sử dụng `@defer` để hiển thị nội dung khi có sẵn
5. **Suy giảm nhẹ nhàng (Graceful degradation)** - Dữ liệu một phần tốt hơn là không có dữ liệu

---

## Các Mẫu Trạng thái Đang tải

### Quy tắc Vàng

**CHỈ hiển thị chỉ báo đang tải khi không có dữ liệu để hiển thị.**

```typescript
@Component({
  template: `
    @if (error()) {
      <app-error-state [error]="error()" (retry)="load()" />
    } @else if (loading() && !items().length) {
      <app-skeleton-list />
    } @else if (!items().length) {
      <app-empty-state message="No items found" />
    } @else {
      <app-item-list [items]="items()" />
    }
  `,
})
export class ItemListComponent {
  private store = inject(ItemStore);

  items = this.store.items;
  loading = this.store.loading;
  error = this.store.error;
}
```

### Cây Quyết định Trạng thái Đang tải

```
Có lỗi không?
  → Có: Hiển thị trạng thái lỗi với tùy chọn thử lại
  → Không: Tiếp tục

Có đang tải VÀ chúng ta không có dữ liệu không?
  → Có: Hiển thị chỉ báo đang tải (spinner/skeleton)
  → Không: Tiếp tục

Chúng ta có dữ liệu không?
  → Có, với các mục: Hiển thị dữ liệu
  → Có, nhưng trống: Hiển thị trạng thái trống
  → Không: Hiển thị đang tải (dự phòng)
```

### Skeleton vs Spinner

| Sử dụng Skeleton Khi       | Sử dụng Spinner Khi          |
| -------------------------- | ---------------------------- |
| Hình dạng nội dung đã biết | Hình dạng nội dung chưa biết |
| Bố cục danh sách/thẻ       | Các hành động modal          |
| Tải trang ban đầu          | Gửi nút (Button submissions) |
| Chỗ dành sẵn cho nội dung  | Các hoạt động nội tuyến      |

---

## Các Mẫu Luồng Điều khiển

### @if/@else cho Kết xuất Có điều kiện

```html
@if (user(); as user) {
<span>Welcome, {{ user.name }}</span>
} @else if (loading()) {
<app-spinner size="small" />
} @else {
<a routerLink="/login">Sign In</a>
}
```

### @for với Track

```html
@for (item of items(); track item.id) {
<app-item-card [item]="item" (delete)="remove(item.id)" />
} @empty {
<app-empty-state
  icon="inbox"
  message="No items yet"
  actionLabel="Create Item"
  (action)="create()"
/>
}
```

### @defer cho Tải Tiến bộ (Progressive Loading)

```html
<!-- Nội dung quan trọng tải ngay lập tức -->
<app-header />
<app-hero-section />

<!-- Nội dung không quan trọng được trì hoãn -->
@defer (on viewport) {
<app-comments [postId]="postId()" />
} @placeholder {
<div class="h-32 bg-gray-100 animate-pulse"></div>
} @loading (minimum 200ms) {
<app-spinner />
} @error {
<app-error-state message="Failed to load comments" />
}
```

---

## Các Mẫu Xử lý Lỗi

### Phân cấp Xử lý Lỗi

```
1. Lỗi nội tuyến (mức trường) → Lỗi xác thực biểu mẫu
2. Thông báo Toast → Lỗi có thể phục hồi, người dùng có thể thử lại
3. Banner lỗi → Lỗi cấp trang, dữ liệu vẫn có thể sử dụng một phần
4. Màn hình lỗi đầy đủ → Không thể phục hồi, cần hành động từ người dùng
```

### Luôn Hiển thị Lỗi

**QUAN TRỌNG: Không bao giờ nuốt lỗi một cách im lặng.**

```typescript
// ĐÚNG - Lỗi luôn được hiển thị cho người dùng
@Component({...})
export class CreateItemComponent {
  private store = inject(ItemStore);
  private toast = inject(ToastService);

  async create(data: CreateItemDto) {
    try {
      await this.store.create(data);
      this.toast.success('Item created successfully');
      this.router.navigate(['/items']);
    } catch (error) {
      console.error('createItem failed:', error);
      this.toast.error('Failed to create item. Please try again.');
    }
  }
}

// SAI - Lỗi bị bắt im lặng
async create(data: CreateItemDto) {
  try {
    await this.store.create(data);
  } catch (error) {
    console.error(error); // Người dùng không thấy gì cả!
  }
}
```

### Mẫu Thành phần Trạng thái Lỗi

```typescript
@Component({
  selector: "app-error-state",
  standalone: true,
  imports: [NgOptimizedImage],
  template: `
    <div class="error-state">
      <img ngSrc="/assets/error-icon.svg" width="64" height="64" alt="" />
      <h3>{{ title() }}</h3>
      <p>{{ message() }}</p>
      @if (retry.observed) {
        <button (click)="retry.emit()" class="btn-primary">Try Again</button>
      }
    </div>
  `,
})
export class ErrorStateComponent {
  title = input("Something went wrong");
  message = input("An unexpected error occurred");
  retry = output<void>();
}
```

---

## Các Mẫu Trạng thái Nút

### Trạng thái Đang tải của Nút

```html
<button
  (click)="handleSubmit()"
  [disabled]="isSubmitting() || !form.valid"
  class="btn-primary"
>
  @if (isSubmitting()) {
  <app-spinner size="small" class="mr-2" />
  Saving... } @else { Save Changes }
</button>
```

### Vô hiệu hóa Trong các Hoạt động

**QUAN TRỌNG: Luôn vô hiệu hóa các trình kích hoạt trong các hoạt động bất đồng bộ.**

```typescript
// ĐÚNG - Nút bị vô hiệu hóa khi đang tải
@Component({
  template: `
    <button
      [disabled]="saving()"
      (click)="save()"
    >
      @if (saving()) {
        <app-spinner size="sm" /> Saving...
      } @else {
        Save
      }
    </button>
  `
})
export class SaveButtonComponent {
  saving = signal(false);

  async save() {
    this.saving.set(true);
    try {
      await this.service.save();
    } finally {
      this.saving.set(false);
    }
  }
}

// SAI - Người dùng có thể nhấp nhiều lần
<button (click)="save()">
  {{ saving() ? 'Saving...' : 'Save' }}
</button>
```

---

## Trạng thái Trống (Empty States)

### Yêu cầu về Trạng thái Trống

Mọi danh sách/bộ sưu tập PHẢI có một trạng thái trống:

```html
@for (item of items(); track item.id) {
<app-item-card [item]="item" />
} @empty {
<app-empty-state
  icon="folder-open"
  title="No items yet"
  description="Create your first item to get started"
  actionLabel="Create Item"
  (action)="openCreateDialog()"
/>
}
```

### Trạng thái Trống theo Ngữ cảnh

```typescript
@Component({
  selector: "app-empty-state",
  template: `
    <div class="empty-state">
      <span class="icon" [class]="icon()"></span>
      <h3>{{ title() }}</h3>
      <p>{{ description() }}</p>
      @if (actionLabel()) {
        <button (click)="action.emit()" class="btn-primary">
          {{ actionLabel() }}
        </button>
      }
    </div>
  `,
})
export class EmptyStateComponent {
  icon = input("inbox");
  title = input.required<string>();
  description = input("");
  actionLabel = input<string | null>(null);
  action = output<void>();
}
```

---

## Các Mẫu Biểu mẫu

### Biểu mẫu với Đang tải và Xác thực

```typescript
@Component({
  template: `
    <form [formGroup]="form" (ngSubmit)="onSubmit()">
      <div class="form-field">
        <label for="name">Name</label>
        <input
          id="name"
          formControlName="name"
          [class.error]="isFieldInvalid('name')"
        />
        @if (isFieldInvalid("name")) {
          <span class="error-text">
            {{ getFieldError("name") }}
          </span>
        }
      </div>

      <div class="form-field">
        <label for="email">Email</label>
        <input id="email" type="email" formControlName="email" />
        @if (isFieldInvalid("email")) {
          <span class="error-text">
            {{ getFieldError("email") }}
          </span>
        }
      </div>

      <button type="submit" [disabled]="form.invalid || submitting()">
        @if (submitting()) {
          <app-spinner size="sm" /> Submitting...
        } @else {
          Submit
        }
      </button>
    </form>
  `,
})
export class UserFormComponent {
  private fb = inject(FormBuilder);

  submitting = signal(false);

  form = this.fb.group({
    name: ["", [Validators.required, Validators.minLength(2)]],
    email: ["", [Validators.required, Validators.email]],
  });

  isFieldInvalid(field: string): boolean {
    const control = this.form.get(field);
    return control ? control.invalid && control.touched : false;
  }

  getFieldError(field: string): string {
    const control = this.form.get(field);
    if (control?.hasError("required")) return "This field is required";
    if (control?.hasError("email")) return "Invalid email format";
    if (control?.hasError("minlength")) return "Too short";
    return "";
  }

  async onSubmit() {
    if (this.form.invalid) return;

    this.submitting.set(true);
    try {
      await this.service.submit(this.form.value);
      this.toast.success("Submitted successfully");
    } catch {
      this.toast.error("Submission failed");
    } finally {
      this.submitting.set(false);
    }
  }
}
```

---

## Các Mẫu Hộp thoại/Modal

### Hộp thoại Xác nhận

```typescript
// dialog.service.ts
@Injectable({ providedIn: 'root' })
export class DialogService {
  private dialog = inject(Dialog); // CDK Dialog hoặc tùy chỉnh

  async confirm(options: {
    title: string;
    message: string;
    confirmText?: string;
    cancelText?: string;
  }): Promise<boolean> {
    const dialogRef = this.dialog.open(ConfirmDialogComponent, {
      data: options,
    });

    return await firstValueFrom(dialogRef.closed) ?? false;
  }
}

// Sử dụng
async deleteItem(item: Item) {
  const confirmed = await this.dialog.confirm({
    title: 'Delete Item',
    message: `Are you sure you want to delete "${item.name}"?`,
    confirmText: 'Delete',
  });

  if (confirmed) {
    await this.store.delete(item.id);
  }
}
```

---

## Các Phản mẫu (Anti-Patterns)

### Trạng thái Đang tải

```typescript
// SAI - Spinner khi dữ liệu tồn tại (gây nhấp nháy khi lấy lại dữ liệu)
@if (loading()) {
  <app-spinner />
}

// ĐÚNG - Chỉ hiển thị đang tải khi không có dữ liệu
@if (loading() && !items().length) {
  <app-spinner />
}
```

### Xử lý Lỗi

```typescript
// SAI - Lỗi bị nuốt
try {
  await this.service.save();
} catch (e) {
  console.log(e); // Người dùng không biết gì cả!
}

// ĐÚNG - Lỗi được hiển thị
try {
  await this.service.save();
} catch (e) {
  console.error("Save failed:", e);
  this.toast.error("Failed to save. Please try again.");
}
```

### Trạng thái Nút

```html
<!-- SAI - Nút không bị vô hiệu hóa trong khi gửi -->
<button (click)="submit()">Submit</button>

<!-- ĐÚNG - Bị vô hiệu hóa và hiển thị đang tải -->
<button (click)="submit()" [disabled]="loading()">
  @if (loading()) {
  <app-spinner size="sm" />
  } Submit
</button>
```

---

## Danh sách Kiểm tra Trạng thái UI

Trước khi hoàn thành bất kỳ thành phần UI nào:

### Trạng thái UI

- [ ] Trạng thái lỗi được xử lý và hiển thị cho người dùng
- [ ] Trạng thái đang tải chỉ hiển thị khi không có dữ liệu tồn tại
- [ ] Trạng thái trống được cung cấp cho các bộ sưu tập (khối `@empty`)
- [ ] Các nút bị vô hiệu hóa trong các hoạt động bất đồng bộ
- [ ] Các nút hiển thị chỉ báo đang tải khi thích hợp

### Dữ liệu & Biến đổi (Mutations)

- [ ] Tất cả các hoạt động bất đồng bộ đều có xử lý lỗi
- [ ] Tất cả các hành động của người dùng đều có phản hồi (toast/hình ảnh)
- [ ] Cập nhật lạc quan (Optimistic updates) khôi phục lại khi thất bại

### Khả năng truy cập (Accessibility)

- [ ] Trạng thái đang tải được thông báo cho trình đọc màn hình
- [ ] Thông báo lỗi được liên kết với các trường biểu mẫu
- [ ] Quản lý tiêu điểm (focus management) sau khi thay đổi trạng thái

---

## Tích hợp với Các Kỹ năng Khác

- **angular-state-management**: Sử dụng Signal stores cho trạng thái
- **angular**: Áp dụng các mẫu hiện đại (Signals, @defer)
- **testing-patterns**: Kiểm thử tất cả các trạng thái UI
