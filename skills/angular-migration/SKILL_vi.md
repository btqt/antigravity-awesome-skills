---
name: angular-migration
description: Di chuyển từ AngularJS sang Angular sử dụng chế độ lai (hybrid mode), viết lại thành phần tăng dần và cập nhật dependency injection. Sử dụng khi nâng cấp các ứng dụng AngularJS, lập kế hoạch di chuyển framework hoặc hiện đại hóa mã Angular cũ.
---

# Di chuyển Angular

Làm chủ việc di chuyển từ AngularJS sang Angular, bao gồm các ứng dụng lai, chuyển đổi thành phần, thay đổi dependency injection và di chuyển định tuyến.

## Sử dụng kỹ năng này khi

- Di chuyển các ứng dụng AngularJS (1.x) sang Angular (2+)
- Chạy các ứng dụng lai AngularJS/Angular
- Chuyển đổi directives thành components
- Hiện đại hóa dependency injection
- Di chuyển hệ thống định tuyến (routing systems)
- Cập nhật lên các phiên bản Angular mới nhất
- Triển khai các thực hành tốt nhất cho Angular

## Không sử dụng kỹ năng này khi

- Bạn không di chuyển từ AngularJS sang Angular
- Ứng dụng đã ở phiên bản Angular hiện đại
- Bạn chỉ cần sửa lỗi giao diện nhỏ mà không thay đổi framework

## Hướng dẫn

1. Đánh giá cơ sở mã AngularJS, các phụ thuộc và rủi ro di chuyển.
2. Chọn chiến lược di chuyển (lai vs viết lại) và xác định các cột mốc.
3. Thiết lập ngUpgrade và di chuyển các modules, components và routing.
4. Xác thực bằng các bài kiểm tra và lập kế hoạch chuyển đổi an toàn.

## An toàn

- Tránh chuyển đổi kiểu "big-bang" mà không có xác thực trên môi trường staging và khả năng rollback.
- Duy trì kiểm thử tương thích lai trong quá trình di chuyển tăng dần.

## Các Chiến lược Di chuyển

### 1. Big Bang (Viết lại hoàn toàn)

- Viết lại toàn bộ ứng dụng bằng Angular
- Phát triển song song
- Chuyển đổi cùng một lúc
- **Tốt nhất cho:** Các ứng dụng nhỏ, dự án mới (green field)

### 2. Tăng dần (Phương pháp Lai)

- Chạy AngularJS và Angular song song
- Di chuyển từng tính năng một
- Sử dụng ngUpgrade để tương tác
- **Tốt nhất cho:** Các ứng dụng lớn, chuyển giao liên tục (continuous delivery)

### 3. Lát cắt Dọc (Vertical Slice)

- Di chuyển hoàn toàn một tính năng
- Tính năng mới bằng Angular, duy trì tính năng cũ bằng AngularJS
- Thay thế dần dần
- **Tốt nhất cho:** Các ứng dụng trung bình, các tính năng riêng biệt

## Thiết lập Ứng dụng Lai

```typescript
// main.ts - Bootstrap hybrid app
import { platformBrowserDynamic } from "@angular/platform-browser-dynamic";
import { UpgradeModule } from "@angular/upgrade/static";
import { AppModule } from "./app/app.module";

platformBrowserDynamic()
  .bootstrapModule(AppModule)
  .then((platformRef) => {
    const upgrade = platformRef.injector.get(UpgradeModule);
    // Bootstrap AngularJS
    upgrade.bootstrap(document.body, ["myAngularJSApp"], { strictDi: true });
  });
```

```typescript
// app.module.ts
import { NgModule } from "@angular/core";
import { BrowserModule } from "@angular/platform-browser";
import { UpgradeModule } from "@angular/upgrade/static";

@NgModule({
  imports: [BrowserModule, UpgradeModule],
})
export class AppModule {
  constructor(private upgrade: UpgradeModule) {}

  ngDoBootstrap() {
    // Được bootstrap thủ công trong main.ts
  }
}
```

## Di chuyển Thành phần (Component Migration)

### AngularJS Controller → Angular Component

```javascript
// Trước: AngularJS controller
angular
  .module("myApp")
  .controller("UserController", function ($scope, UserService) {
    $scope.user = {};

    $scope.loadUser = function (id) {
      UserService.getUser(id).then(function (user) {
        $scope.user = user;
      });
    };

    $scope.saveUser = function () {
      UserService.saveUser($scope.user);
    };
  });
```

```typescript
// Sau: Angular component
import { Component, OnInit } from "@angular/core";
import { UserService } from "./user.service";

@Component({
  selector: "app-user",
  template: `
    <div>
      <h2>{{ user.name }}</h2>
      <button (click)="saveUser()">Save</button>
    </div>
  `,
})
export class UserComponent implements OnInit {
  user: any = {};

  constructor(private userService: UserService) {}

  ngOnInit() {
    this.loadUser(1);
  }

  loadUser(id: number) {
    this.userService.getUser(id).subscribe((user) => {
      this.user = user;
    });
  }

  saveUser() {
    this.userService.saveUser(this.user);
  }
}
```

### AngularJS Directive → Angular Component

```javascript
// Trước: AngularJS directive
angular.module("myApp").directive("userCard", function () {
  return {
    restrict: "E",
    scope: {
      user: "=",
      onDelete: "&",
    },
    template: `
      <div class="card">
        <h3>{{ user.name }}</h3>
        <button ng-click="onDelete()">Delete</button>
      </div>
    `,
  };
});
```

```typescript
// Sau: Angular component
import { Component, Input, Output, EventEmitter } from "@angular/core";

@Component({
  selector: "app-user-card",
  template: `
    <div class="card">
      <h3>{{ user.name }}</h3>
      <button (click)="delete.emit()">Delete</button>
    </div>
  `,
})
export class UserCardComponent {
  @Input() user: any;
  @Output() delete = new EventEmitter<void>();
}

// Sử dụng: <app-user-card [user]="user" (delete)="handleDelete()"></app-user-card>
```

## Di chuyển Dịch vụ (Service Migration)

```javascript
// Trước: AngularJS service
angular.module("myApp").factory("UserService", function ($http) {
  return {
    getUser: function (id) {
      return $http.get("/api/users/" + id);
    },
    saveUser: function (user) {
      return $http.post("/api/users", user);
    },
  };
});
```

```typescript
// Sau: Angular service
import { Injectable } from "@angular/core";
import { HttpClient } from "@angular/common/http";
import { Observable } from "rxjs";

@Injectable({
  providedIn: "root",
})
export class UserService {
  constructor(private http: HttpClient) {}

  getUser(id: number): Observable<any> {
    return this.http.get(`/api/users/${id}`);
  }

  saveUser(user: any): Observable<any> {
    return this.http.post("/api/users", user);
  }
}
```

## Thay đổi Dependency Injection

### Hạ cấp (Downgrading) Angular → AngularJS

```typescript
// Angular service
import { Injectable } from "@angular/core";

@Injectable({ providedIn: "root" })
export class NewService {
  getData() {
    return "data from Angular";
  }
}

// Cung cấp cho AngularJS
import { downgradeInjectable } from "@angular/upgrade/static";

angular.module("myApp").factory("newService", downgradeInjectable(NewService));

// Sử dụng trong AngularJS
angular.module("myApp").controller("OldController", function (newService) {
  console.log(newService.getData());
});
```

### Nâng cấp (Upgrading) AngularJS → Angular

```typescript
// AngularJS service
angular.module('myApp').factory('oldService', function() {
  return {
    getData: function() {
      return 'data from AngularJS';
    }
  };
});

// Cung cấp cho Angular
import { InjectionToken } from '@angular/core';

export const OLD_SERVICE = new InjectionToken<any>('oldService');

@NgModule({
  providers: [
    {
      provide: OLD_SERVICE,
      useFactory: (i: any) => i.get('oldService'),
      deps: ['$injector']
    }
  ]
})

// Sử dụng trong Angular
@Component({...})
export class NewComponent {
  constructor(@Inject(OLD_SERVICE) private oldService: any) {
    console.log(this.oldService.getData());
  }
}
```

## Di chuyển Định tuyến (Routing Migration)

```javascript
// Trước: AngularJS routing
angular.module("myApp").config(function ($routeProvider) {
  $routeProvider
    .when("/users", {
      template: "<user-list></user-list>",
    })
    .when("/users/:id", {
      template: "<user-detail></user-detail>",
    });
});
```

```typescript
// Sau: Angular routing
import { NgModule } from "@angular/core";
import { RouterModule, Routes } from "@angular/router";

const routes: Routes = [
  { path: "users", component: UserListComponent },
  { path: "users/:id", component: UserDetailComponent },
];

@NgModule({
  imports: [RouterModule.forRoot(routes)],
  exports: [RouterModule],
})
export class AppRoutingModule {}
```

## Di chuyển Biểu mẫu (Forms Migration)

```html
<!-- Trước: AngularJS -->
<form name="userForm" ng-submit="saveUser()">
  <input type="text" ng-model="user.name" required />
  <input type="email" ng-model="user.email" required />
  <button ng-disabled="userForm.$invalid">Save</button>
</form>
```

```typescript
// Sau: Angular (Template-driven)
@Component({
  template: `
    <form #userForm="ngForm" (ngSubmit)="saveUser()">
      <input type="text" [(ngModel)]="user.name" name="name" required>
      <input type="email" [(ngModel)]="user.email" name="email" required>
      <button [disabled]="userForm.invalid">Save</button>
    </form>
  `
})

// Hoặc Reactive Forms (ưu tiên)
import { FormBuilder, FormGroup, Validators } from '@angular/forms';

@Component({
  template: `
    <form [formGroup]="userForm" (ngSubmit)="saveUser()">
      <input formControlName="name">
      <input formControlName="email">
      <button [disabled]="userForm.invalid">Save</button>
    </form>
  `
})
export class UserFormComponent {
  userForm: FormGroup;

  constructor(private fb: FormBuilder) {
    this.userForm = this.fb.group({
      name: ['', Validators.required],
      email: ['', [Validators.required, Validators.email]]
    });
  }

  saveUser() {
    console.log(this.userForm.value);
  }
}
```

## Dòng thời gian Di chuyển

```
Giai đoạn 1: Thiết lập (1-2 tuần)
- Cài đặt Angular CLI
- Thiết lập ứng dụng lai
- Cấu hình công cụ build
- Thiết lập kiểm thử

Giai đoạn 2: Cơ sở hạ tầng (2-4 tuần)
- Di chuyển các dịch vụ (services)
- Di chuyển các tiện ích (utilities)
- Thiết lập định tuyến
- Di chuyển các thành phần chung (shared components)

Giai đoạn 3: Di chuyển Tính năng (thay đổi tùy theo dự án)
- Di chuyển từng tính năng một
- Kiểm thử kỹ lưỡng
- Triển khai tăng dần

Giai đoạn 4: Dọn dẹp (1-2 tuần)
- Xóa mã AngularJS
- Xóa ngUpgrade
- Tối ưu hóa gói
- Kiểm thử cuối cùng
```

## Tài nguyên

- **references/hybrid-mode.md**: Các mẫu ứng dụng lai
- **references/component-migration.md**: Hướng dẫn chuyển đổi thành phần
- **references/dependency-injection.md**: Chiến lược di chuyển DI
- **references/routing.md**: Di chuyển định tuyến
- **assets/hybrid-bootstrap.ts**: Mẫu ứng dụng lai
- **assets/migration-timeline.md**: Lập kế hoạch dự án
- **scripts/analyze-angular-app.sh**: Script phân tích ứng dụng

## Thực hành Tốt nhất

1. **Bắt đầu với Dịch vụ**: Di chuyển các dịch vụ trước (dễ hơn)
2. **Phương pháp Tăng dần**: Di chuyển từng tính năng
3. **Kiểm thử Liên tục**: Kiểm thử ở mọi bước
4. **Sử dụng TypeScript**: Di chuyển sang TypeScript sớm
5. **Tuân thủ Style Guide**: Style guide của Angular ngay từ ngày đầu
6. **Tối ưu hóa sau**: Làm cho nó hoạt động, sau đó tối ưu hóa
7. **Ghi chép tài liệu**: Giữ các ghi chú di chuyển

## Các Cạm bẫy Phổ biến

- Không thiết lập ứng dụng lai một cách chính xác
- Di chuyển giao diện người dùng trước logic
- Bỏ qua sự khác biệt về phát hiện thay đổi
- Không xử lý scope đúng cách
- Trộn lẫn các mẫu (AngularJS + Angular)
- Kiểm thử không đầy đủ
