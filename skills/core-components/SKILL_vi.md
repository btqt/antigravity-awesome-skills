---
name: core-components
description: Thư viện thành phần cốt lõi và các mẫu hệ thống thiết kế. Sử dụng khi xây dựng UI, sử dụng mã thông báo thiết kế hoặc làm việc với thư viện thành phần.
---

# Các Thành Phần Cốt Lõi (Core Components)

## Tổng quan Hệ thống Thiết kế

Sử dụng các thành phần từ thư viện cốt lõi của bạn thay vì các thành phần nền tảng thô. Điều này đảm bảo kiểu dáng và hành vi nhất quán.

## Mã thông báo Thiết kế (Design Tokens)

**KHÔNG BAO GIỜ mã hóa cứng các giá trị. Luôn sử dụng mã thông báo thiết kế.**

### Mã thông báo Khoảng cách

```tsx
// CORRECT - Use tokens
<Box padding="$4" marginBottom="$2" />

// WRONG - Hard-coded values
<Box padding={16} marginBottom={8} />
```

| Token | Value |
| ----- | ----- |
| `$1`  | 4px   |
| `$2`  | 8px   |
| `$3`  | 12px  |
| `$4`  | 16px  |
| `$6`  | 24px  |
| `$8`  | 32px  |

### Mã thông báo Màu sắc

```tsx
// CORRECT - Semantic tokens
<Text color="$textPrimary" />
<Box backgroundColor="$backgroundSecondary" />

// WRONG - Hard-coded colors
<Text color="#333333" />
<Box backgroundColor="rgb(245, 245, 245)" />
```

| Semantic Token   | Use For                      |
| ---------------- | ---------------------------- |
| `$textPrimary`   | Văn bản chính                |
| `$textSecondary` | Văn bản hỗ trợ               |
| `$textTertiary`  | Văn bản bị vô hiệu hóa/gợi ý |
| `$primary500`    | Màu thương hiệu/điểm nhấn    |
| `$statusError`   | Trạng thái lỗi               |
| `$statusSuccess` | Trạng thái thành công        |

### Mã thông báo Kiểu chữ

```tsx
<Text fontSize="$lg" fontWeight="$semibold" />
```

| Token  | Size |
| ------ | ---- |
| `$xs`  | 12px |
| `$sm`  | 14px |
| `$md`  | 16px |
| `$lg`  | 18px |
| `$xl`  | 20px |
| `$2xl` | 24px |

## Các Thành Phần Cốt Lõi

### Box

Thành phần bố cục cơ sở với hỗ trợ mã thông báo:

```tsx
<Box padding="$4" backgroundColor="$backgroundPrimary" borderRadius="$lg">
  {children}
</Box>
```

### HStack / VStack

Bố cục flex ngang và dọc:

```tsx
<HStack gap="$3" alignItems="center">
  <Icon name="user" />
  <Text>Username</Text>
</HStack>

<VStack gap="$4" padding="$4">
  <Heading>Title</Heading>
  <Text>Content</Text>
</VStack>
```

### Text

Kiểu chữ với hỗ trợ mã thông báo:

```tsx
<Text fontSize="$lg" fontWeight="$semibold" color="$textPrimary">
  Hello World
</Text>
```

### Button

Nút tương tác với các biến thể:

```tsx
<Button
  onPress={handlePress}
  variant="solid"
  size="md"
  isLoading={loading}
  isDisabled={disabled}
>
  Click Me
</Button>
```

| Variant   | Use For                  |
| --------- | ------------------------ |
| `solid`   | Hành động chính          |
| `outline` | Hành động phụ            |
| `ghost`   | Hành động cấp ba/tinh tế |
| `link`    | Hành động nội tuyến      |

### Input

Đầu vào biểu mẫu với xác thực:

```tsx
<Input
  value={value}
  onChangeText={setValue}
  placeholder="Enter text"
  error={touched ? errors.field : undefined}
  label="Field Name"
>
```

### Card

Vùng chứa nội dung:

```tsx
<Card padding="$4" gap="$3">
  <CardHeader>
    <Heading size="sm">Card Title</Heading>
  </CardHeader>
  <CardBody>
    <Text>Card content</Text>
  </CardBody>
</Card>
```

## Các Mẫu Bố cục

### Bố cục Màn hình

```tsx
const MyScreen = () => (
  <Screen>
    <ScreenHeader title="Page Title" />
    <ScreenContent padding="$4">{/* Content */}</ScreenContent>
  </Screen>
);
```

### Bố cục Biểu mẫu

```tsx
<VStack gap="$4" padding="$4">
  <Input label="Name" {...nameProps} />
  <Input label="Email" {...emailProps} />
  <Button isLoading={loading}>Submit</Button>
</VStack>
```

### Bố cục Mục Danh sách

```tsx
<HStack
  padding="$4"
  gap="$3"
  alignItems="center"
  borderBottomWidth={1}
  borderColor="$borderLight"
>
  <Avatar source={{ uri: imageUrl }} size="md" />
  <VStack flex={1}>
    <Text fontWeight="$semibold">{title}</Text>
    <Text color="$textSecondary" fontSize="$sm">
      {subtitle}
    </Text>
  </VStack>
  <Icon name="chevron-right" color="$textTertiary" />
</HStack>
```

## Các Chống Mẫu

```tsx
// WRONG - Hard-coded values
<View style={{ padding: 16, backgroundColor: '#fff' }}>

// CORRECT - Design tokens
<Box padding="$4" backgroundColor="$backgroundPrimary">


// WRONG - Raw platform components
import { View, Text } from 'react-native';

// CORRECT - Core components
import { Box, Text } from 'components/core';


// WRONG - Inline styles
<Text style={{ fontSize: 18, fontWeight: '600' }}>

// CORRECT - Token props
<Text fontSize="$lg" fontWeight="$semibold">
```

## Mẫu Props Thành phần

Khi tạo các thành phần, sử dụng props dựa trên mã thông báo:

```tsx
interface CardProps {
  padding?: "$2" | "$4" | "$6";
  variant?: "elevated" | "outlined" | "filled";
  children: React.ReactNode;
}

const Card = ({
  padding = "$4",
  variant = "elevated",
  children,
}: CardProps) => (
  <Box
    padding={padding}
    backgroundColor="$backgroundPrimary"
    borderRadius="$lg"
    {...variantStyles[variant]}
  >
    {children}
  </Box>
);
```

## Tích hợp với Các Kỹ năng Khác

- **react-ui-patterns**: Sử dụng các thành phần cốt lõi cho các trạng thái UI
- **testing-patterns**: Mock các thành phần cốt lõi trong các thử nghiệm
- **storybook**: Tài liệu hóa các biến thể thành phần
