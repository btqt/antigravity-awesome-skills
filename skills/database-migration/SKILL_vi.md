---
name: database-migration
description: Thực hiện di chuyển cơ sở dữ liệu qua các ORM và nền tảng với chiến lược không thời gian chết, chuyển đổi dữ liệu và quy trình rollback. Sử dụng khi di chuyển cơ sở dữ liệu, thay đổi lược đồ, thực hiện chuyển đổi dữ liệu hoặc triển khai chiến lược triển khai không thời gian chết.
---

# Di Chuyển Cơ Sở Dữ Liệu (Database Migration)

Làm chủ việc di chuyển lược đồ và dữ liệu cơ sở dữ liệu qua các ORM (Sequelize, TypeORM, Prisma), bao gồm các chiến lược rollback và triển khai không thời gian chết.

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến di chuyển cơ sở dữ liệu
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc và đầu vào cần thiết.
- Áp dụng các thực tiễn tốt nhất có liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần các ví dụ chi tiết, hãy mở `resources/implementation-playbook.md`.

## Sử dụng kỹ năng này khi

- Di chuyển giữa các ORM khác nhau
- Thực hiện chuyển đổi lược đồ
- Di chuyển dữ liệu giữa các cơ sở dữ liệu
- Triển khai quy trình rollback
- Triển khai không thời gian chết
- Nâng cấp phiên bản cơ sở dữ liệu
- Tái cấu trúc mô hình dữ liệu

## Các Di chuyển ORM

### Sequelize Migrations

```javascript
// migrations/20231201-create-users.js
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.createTable("users", {
      id: {
        type: Sequelize.INTEGER,
        primaryKey: true,
        autoIncrement: true,
      },
      email: {
        type: Sequelize.STRING,
        unique: true,
        allowNull: false,
      },
      createdAt: Sequelize.DATE,
      updatedAt: Sequelize.DATE,
    });
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.dropTable("users");
  },
};

// Run: npx sequelize-cli db:migrate
// Rollback: npx sequelize-cli db:migrate:undo
```

### TypeORM Migrations

```typescript
// migrations/1701234567-CreateUsers.ts
import { MigrationInterface, QueryRunner, Table } from "typeorm";

export class CreateUsers1701234567 implements MigrationInterface {
  public async up(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.createTable(
      new Table({
        name: "users",
        columns: [
          {
            name: "id",
            type: "int",
            isPrimary: true,
            isGenerated: true,
            generationStrategy: "increment",
          },
          {
            name: "email",
            type: "varchar",
            isUnique: true,
          },
          {
            name: "created_at",
            type: "timestamp",
            default: "CURRENT_TIMESTAMP",
          },
        ],
      }),
    );
  }

  public async down(queryRunner: QueryRunner): Promise<void> {
    await queryRunner.dropTable("users");
  }
}

// Run: npm run typeorm migration:run
// Rollback: npm run typeorm migration:revert
```

### Prisma Migrations

```prisma
// schema.prisma
model User {
  id        Int      @id @default(autoincrement())
  email     String   @unique
  createdAt DateTime @default(now())
}

// Generate migration: npx prisma migrate dev --name create_users
// Apply: npx prisma migrate deploy
```

## Chuyển đổi Lược đồ

### Thêm Cột với Giá trị Mặc định

```javascript
// Safe migration: add column with default
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.addColumn("users", "status", {
      type: Sequelize.STRING,
      defaultValue: "active",
      allowNull: false,
    });
  },

  down: async (queryInterface) => {
    await queryInterface.removeColumn("users", "status");
  },
};
```

### Đổi tên Cột (Không Thời gian chết)

```javascript
// Step 1: Add new column
module.exports = {
  up: async (queryInterface, Sequelize) => {
    await queryInterface.addColumn("users", "full_name", {
      type: Sequelize.STRING,
    });

    // Copy data from old column
    await queryInterface.sequelize.query("UPDATE users SET full_name = name");
  },

  down: async (queryInterface) => {
    await queryInterface.removeColumn("users", "full_name");
  },
};

// Step 2: Update application to use new column

// Step 3: Remove old column
module.exports = {
  up: async (queryInterface) => {
    await queryInterface.removeColumn("users", "name");
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.addColumn("users", "name", {
      type: Sequelize.STRING,
    });
  },
};
```

### Thay đổi Loại Cột

```javascript
module.exports = {
  up: async (queryInterface, Sequelize) => {
    // For large tables, use multi-step approach

    // 1. Add new column
    await queryInterface.addColumn("users", "age_new", {
      type: Sequelize.INTEGER,
    });

    // 2. Copy and transform data
    await queryInterface.sequelize.query(`
      UPDATE users
      SET age_new = CAST(age AS INTEGER)
      WHERE age IS NOT NULL
    `);

    // 3. Drop old column
    await queryInterface.removeColumn("users", "age");

    // 4. Rename new column
    await queryInterface.renameColumn("users", "age_new", "age");
  },

  down: async (queryInterface, Sequelize) => {
    await queryInterface.changeColumn("users", "age", {
      type: Sequelize.STRING,
    });
  },
};
```

## Chuyển đổi Dữ liệu

### Di chuyển Dữ liệu Phức tạp

```javascript
module.exports = {
  up: async (queryInterface, Sequelize) => {
    // Get all records
    const [users] = await queryInterface.sequelize.query(
      "SELECT id, address_string FROM users",
    );

    // Transform each record
    for (const user of users) {
      const addressParts = user.address_string.split(",");

      await queryInterface.sequelize.query(
        `UPDATE users
         SET street = :street,
             city = :city,
             state = :state
         WHERE id = :id`,
        {
          replacements: {
            id: user.id,
            street: addressParts[0]?.trim(),
            city: addressParts[1]?.trim(),
            state: addressParts[2]?.trim(),
          },
        },
      );
    }

    // Drop old column
    await queryInterface.removeColumn("users", "address_string");
  },

  down: async (queryInterface, Sequelize) => {
    // Reconstruct original column
    await queryInterface.addColumn("users", "address_string", {
      type: Sequelize.STRING,
    });

    await queryInterface.sequelize.query(`
      UPDATE users
      SET address_string = CONCAT(street, ', ', city, ', ', state)
    `);

    await queryInterface.removeColumn("users", "street");
    await queryInterface.removeColumn("users", "city");
    await queryInterface.removeColumn("users", "state");
  },
};
```

## Chiến lược Rollback

### Di chuyển Dựa trên Giao dịch

```javascript
module.exports = {
  up: async (queryInterface, Sequelize) => {
    const transaction = await queryInterface.sequelize.transaction();

    try {
      await queryInterface.addColumn(
        "users",
        "verified",
        { type: Sequelize.BOOLEAN, defaultValue: false },
        { transaction },
      );

      await queryInterface.sequelize.query(
        "UPDATE users SET verified = true WHERE email_verified_at IS NOT NULL",
        { transaction },
      );

      await transaction.commit();
    } catch (error) {
      await transaction.rollback();
      throw error;
    }
  },

  down: async (queryInterface) => {
    await queryInterface.removeColumn("users", "verified");
  },
};
```

### Rollback Dựa trên Checkpoint

```javascript
module.exports = {
  up: async (queryInterface, Sequelize) => {
    // Create backup table
    await queryInterface.sequelize.query(
      "CREATE TABLE users_backup AS SELECT * FROM users",
    );

    try {
      // Perform migration
      await queryInterface.addColumn("users", "new_field", {
        type: Sequelize.STRING,
      });

      // Verify migration
      const [result] = await queryInterface.sequelize.query(
        "SELECT COUNT(*) as count FROM users WHERE new_field IS NULL",
      );

      if (result[0].count > 0) {
        throw new Error("Migration verification failed");
      }

      // Drop backup
      await queryInterface.dropTable("users_backup");
    } catch (error) {
      // Restore from backup
      await queryInterface.sequelize.query("DROP TABLE users");
      await queryInterface.sequelize.query(
        "CREATE TABLE users AS SELECT * FROM users_backup",
      );
      await queryInterface.dropTable("users_backup");
      throw error;
    }
  },
};
```

## Di chuyển Không Thời gian chết

### Chiến lược Triển khai Xanh-Lục (Blue-Green)

```javascript
// Phase 1: Make changes backward compatible
module.exports = {
  up: async (queryInterface, Sequelize) => {
    // Add new column (both old and new code can work)
    await queryInterface.addColumn("users", "email_new", {
      type: Sequelize.STRING,
    });
  },
};

// Phase 2: Deploy code that writes to both columns

// Phase 3: Backfill data
module.exports = {
  up: async (queryInterface) => {
    await queryInterface.sequelize.query(`
      UPDATE users
      SET email_new = email
      WHERE email_new IS NULL
    `);
  },
};

// Phase 4: Deploy code that reads from new column

// Phase 5: Remove old column
module.exports = {
  up: async (queryInterface) => {
    await queryInterface.removeColumn("users", "email");
  },
};
```

## Di chuyển Chéo Cơ Sở Dữ Liệu

### PostgreSQL sang MySQL

```javascript
// Handle differences
module.exports = {
  up: async (queryInterface, Sequelize) => {
    const dialectName = queryInterface.sequelize.getDialect();

    if (dialectName === "mysql") {
      await queryInterface.createTable("users", {
        id: {
          type: Sequelize.INTEGER,
          primaryKey: true,
          autoIncrement: true,
        },
        data: {
          type: Sequelize.JSON, // MySQL JSON type
        },
      });
    } else if (dialectName === "postgres") {
      await queryInterface.createTable("users", {
        id: {
          type: Sequelize.INTEGER,
          primaryKey: true,
          autoIncrement: true,
        },
        data: {
          type: Sequelize.JSONB, // PostgreSQL JSONB type
        },
      });
    }
  },
};
```

## Tài nguyên

- **references/orm-switching.md**: Hướng dẫn di chuyển ORM
- **references/schema-migration.md**: Các mẫu chuyển đổi lược đồ
- **references/data-transformation.md**: Các tập lệnh di chuyển dữ liệu
- **references/rollback-strategies.md**: Quy trình rollback
- **assets/schema-migration-template.sql**: Mẫu di chuyển SQL
- **assets/data-migration-script.py**: Tiện ích di chuyển dữ liệu
- **scripts/test-migration.sh**: Tập lệnh kiểm thử di chuyển

## Thực tiễn Tốt nhất

1. **Luôn cung cấp Rollback**: Mọi up() đều cần một down()
2. **Kiểm thử Di chuyển**: Kiểm thử trên staging trước
3. **Sử dụng Giao dịch**: Di chuyển nguyên tử khi có thể
4. **Sao lưu Trước**: Luôn sao lưu trước khi di chuyển
5. **Thay đổi Nhỏ**: Chia thành các bước nhỏ, tăng dần
6. **Giám sát**: Theo dõi lỗi trong quá trình triển khai
7. **Tài liệu**: Giải thích tại sao và như thế nào
8. **Lũy đẳng (Idempotent)**: Di chuyển nên có thể chạy lại

## Các Cạm bẫy Phổ biến

- Không kiểm thử quy trình rollback
- Thực hiện các thay đổi phá vỡ mà không có chiến lược không thời gian chết
- Quên xử lý các giá trị NULL
- Không xem xét hiệu suất chỉ mục
- Bỏ qua các ràng buộc khóa ngoại
- Di chuyển quá nhiều dữ liệu cùng lúc
