# Các Mẫu Thiết kế Lược đồ GraphQL

## Tổ chức Lược đồ

### Cấu trúc Lược đồ Mô-đun

```graphql
# user.graphql
type User {
  id: ID!
  email: String!
  name: String!
  posts: [Post!]!
}

extend type Query {
  user(id: ID!): User
  users(first: Int, after: String): UserConnection!
}

extend type Mutation {
  createUser(input: CreateUserInput!): CreateUserPayload!
}

# post.graphql
type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
}

extend type Query {
  post(id: ID!): Post
}
```

## Các Mẫu Thiết kế Kiểu

### 1. Các Kiểu Không-Null (Non-Null Types)

```graphql
type User {
  id: ID! # Luôn bắt buộc
  email: String! # Bắt buộc
  phone: String # Tùy chọn (có thể null)
  posts: [Post!]! # Mảng không null của các bài viết không null
  tags: [String!] # Mảng có thể null của các chuỗi không null
}
```

### 2. Giao diện cho Đa hình (Interfaces for Polymorphism)

```graphql
interface Node {
  id: ID!
  createdAt: DateTime!
}

type User implements Node {
  id: ID!
  createdAt: DateTime!
  email: String!
}

type Post implements Node {
  id: ID!
  createdAt: DateTime!
  title: String!
}

type Query {
  node(id: ID!): Node
}
```

### 3. Union cho Kết quả Không đồng nhất

```graphql
union SearchResult = User | Post | Comment

type Query {
  search(query: String!): [SearchResult!]!
}

# Ví dụ truy vấn
{
  search(query: "graphql") {
    ... on User {
      name
      email
    }
    ... on Post {
      title
      content
    }
    ... on Comment {
      text
      author {
        name
      }
    }
  }
}
```

### 4. Các Kiểu Đầu vào (Input Types)

```graphql
input CreateUserInput {
  email: String!
  name: String!
  password: String!
  profileInput: ProfileInput
}

input ProfileInput {
  bio: String
  avatar: String
  website: String
}

input UpdateUserInput {
  id: ID!
  email: String
  name: String
  profileInput: ProfileInput
}
```

## Các Mẫu Phân trang

### Phân trang Con trỏ Relay (Được khuyến nghị)

```graphql
type UserConnection {
  edges: [UserEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type UserEdge {
  node: User!
  cursor: String!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

type Query {
  users(first: Int, after: String, last: Int, before: String): UserConnection!
}

# Sử dụng
{
  users(first: 10, after: "cursor123") {
    edges {
      cursor
      node {
        id
        name
      }
    }
    pageInfo {
      hasNextPage
      endCursor
    }
  }
}
```

### Phân trang Offset (Đơn giản hơn)

```graphql
type UserList {
  items: [User!]!
  total: Int!
  page: Int!
  pageSize: Int!
}

type Query {
  users(page: Int = 1, pageSize: Int = 20): UserList!
}
```

## Các Mẫu Thiết kế Đột biến (Mutation)

### 1. Mẫu Đầu vào/Payload

```graphql
input CreatePostInput {
  title: String!
  content: String!
  tags: [String!]
}

type CreatePostPayload {
  post: Post
  errors: [Error!]
  success: Boolean!
}

type Error {
  field: String
  message: String!
  code: String!
}

type Mutation {
  createPost(input: CreatePostInput!): CreatePostPayload!
}
```

### 2. Hỗ trợ Phản hồi Lạc quan

```graphql
type UpdateUserPayload {
  user: User
  clientMutationId: String
  errors: [Error!]
}

input UpdateUserInput {
  id: ID!
  name: String
  clientMutationId: String
}

type Mutation {
  updateUser(input: UpdateUserInput!): UpdateUserPayload!
}
```

### 3. Đột biến Hàng loạt

```graphql
input BatchCreateUserInput {
  users: [CreateUserInput!]!
}

type BatchCreateUserPayload {
  results: [CreateUserResult!]!
  successCount: Int!
  errorCount: Int!
}

type CreateUserResult {
  user: User
  errors: [Error!]
  index: Int!
}

type Mutation {
  batchCreateUsers(input: BatchCreateUserInput!): BatchCreateUserPayload!
}
```

## Thiết kế Trường

### Tham số và Lọc

```graphql
type Query {
  posts(
    # Phân trang
    first: Int = 20
    after: String

    # Lọc
    status: PostStatus
    authorId: ID
    tag: String

    # Sắp xếp
    orderBy: PostOrderBy = CREATED_AT
    orderDirection: OrderDirection = DESC

    # Tìm kiếm
    search: String
  ): PostConnection!
}

enum PostStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

enum PostOrderBy {
  CREATED_AT
  UPDATED_AT
  TITLE
}

enum OrderDirection {
  ASC
  DESC
}
```

### Các Trường Tính toán

```graphql
type User {
  firstName: String!
  lastName: String!
  fullName: String! # Được tính trong resolver
  posts: [Post!]!
  postCount: Int! # Được tính, không tải tất cả bài viết
}

type Post {
  likeCount: Int!
  commentCount: Int!
  isLikedByViewer: Boolean! # Phụ thuộc ngữ cảnh
}
```

## Đăng ký (Subscriptions)

```graphql
type Subscription {
  postAdded: Post!

  postUpdated(postId: ID!): Post!

  userStatusChanged(userId: ID!): UserStatus!
}

type UserStatus {
  userId: ID!
  online: Boolean!
  lastSeen: DateTime!
}

# Sử dụng phía máy khách
subscription {
  postAdded {
    id
    title
    author {
      name
    }
  }
}
```

## Scalars Tùy chỉnh

```graphql
scalar DateTime
scalar Email
scalar URL
scalar JSON
scalar Money

type User {
  email: Email!
  website: URL
  createdAt: DateTime!
  metadata: JSON
}

type Product {
  price: Money!
}
```

## Chỉ thị (Directives)

### Chỉ thị Tích hợp sẵn

```graphql
type User {
  name: String!
  email: String! @deprecated(reason: "Use emails field instead")
  emails: [String!]!

  # Bao gồm có điều kiện
  privateData: PrivateData @include(if: $isOwner)
}

# Truy vấn
query GetUser($isOwner: Boolean!) {
  user(id: "123") {
    name
    privateData @include(if: $isOwner) {
      ssn
    }
  }
}
```

### Chỉ thị Tùy chỉnh

```graphql
directive @auth(requires: Role = USER) on FIELD_DEFINITION

enum Role {
  USER
  ADMIN
  MODERATOR
}

type Mutation {
  deleteUser(id: ID!): Boolean! @auth(requires: ADMIN)
  updateProfile(input: ProfileInput!): User! @auth
}
```

## Xử lý Lỗi

### Mẫu Union Error

```graphql
type User {
  id: ID!
  email: String!
}

type ValidationError {
  field: String!
  message: String!
}

type NotFoundError {
  message: String!
  resourceType: String!
  resourceId: ID!
}

type AuthorizationError {
  message: String!
}

union UserResult = User | ValidationError | NotFoundError | AuthorizationError

type Query {
  user(id: ID!): UserResult!
}

# Sử dụng
{
  user(id: "123") {
    ... on User {
      id
      email
    }
    ... on NotFoundError {
      message
      resourceType
    }
    ... on AuthorizationError {
      message
    }
  }
}
```

### Lỗi trong Payload

```graphql
type CreateUserPayload {
  user: User
  errors: [Error!]
  success: Boolean!
}

type Error {
  field: String
  message: String!
  code: ErrorCode!
}

enum ErrorCode {
  VALIDATION_ERROR
  UNAUTHORIZED
  NOT_FOUND
  INTERNAL_ERROR
}
```

## Các Giải pháp cho Vấn đề Truy vấn N+1

### Mẫu DataLoader

```python
from aiodataloader import DataLoader

class PostLoader(DataLoader):
    async def batch_load_fn(self, post_ids):
        posts = await db.posts.find({"id": {"$in": post_ids}})
        post_map = {post["id"]: post for post in posts}
        return [post_map.get(pid) for pid in post_ids]

# Resolver
@user_type.field("posts")
async def resolve_posts(user, info):
    loader = info.context["loaders"]["post"]
    return await loader.load_many(user["post_ids"])
```

### Giới hạn Độ sâu Truy vấn

```python
from graphql import GraphQLError

def depth_limit_validator(max_depth: int):
    def validate(context, node, ancestors):
        depth = len(ancestors)
        if depth > max_depth:
            raise GraphQLError(
                f"Query depth {depth} exceeds maximum {max_depth}"
            )
    return validate
```

### Phân tích Độ phức tạp Truy vấn

```python
def complexity_limit_validator(max_complexity: int):
    def calculate_complexity(node):
        # Mỗi trường = 1, danh sách nhân lên
        complexity = 1
        if is_list_field(node):
            complexity *= get_list_size_arg(node)
        return complexity

    return validate_complexity
```

## Đánh phiên bản Lược đồ

### Ngừng hỗ trợ Trường

```graphql
type User {
  name: String! @deprecated(reason: "Use firstName and lastName")
  firstName: String!
  lastName: String!
}
```

### Tiến hóa Lược đồ

```graphql
# v1 - Ban đầu
type User {
  name: String!
}

# v2 - Thêm trường tùy chọn (tương thích ngược)
type User {
  name: String!
  email: String
}

# v3 - Ngừng hỗ trợ và thêm trường mới
type User {
  name: String! @deprecated(reason: "Use firstName/lastName")
  firstName: String!
  lastName: String!
  email: String
}
```

## Tóm tắt Thực hành Tốt nhất

1. **Nullable vs Non-Null**: Bắt đầu với nullable, chuyển sang non-null khi được đảm bảo
2. **Kiểu Đầu vào**: Luôn sử dụng kiểu đầu vào cho đột biến
3. **Mẫu Payload**: Trả về lỗi trong payload của đột biến
4. **Phân trang**: Sử dụng dựa trên con trỏ cho cuộn vô hạn, offset cho các trường hợp đơn giản
5. **Đặt tên**: Sử dụng camelCase cho các trường, PascalCase cho các kiểu
6. **Ngừng hỗ trợ**: Sử dụng `@deprecated` thay vì xóa trường
7. **DataLoaders**: Luôn sử dụng cho các mối quan hệ để ngăn chặn N+1
8. **Giới hạn Phức tạp**: Bảo vệ chống lại các truy vấn đắt đỏ
9. **Scalars Tùy chỉnh**: Sử dụng cho các kiểu miền cụ thể (Email, DateTime)
10. **Tài liệu**: Viết tài liệu cho tất cả các trường với mô tả
