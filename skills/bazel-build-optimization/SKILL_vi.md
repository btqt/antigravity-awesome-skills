---
name: bazel-build-optimization
description: Tối ưu hóa các bản build Bazel cho monorepos quy mô lớn. Sử dụng khi cấu hình Bazel, triển khai thực thi từ xa (remote execution), hoặc tối ưu hóa hiệu năng build cho các codebase doanh nghiệp.
---

# Tối ưu hóa Build Bazel

Các mẫu production cho Bazel trong monorepos quy mô lớn.

## Không sử dụng kỹ năng này khi

- Nhiệm vụ không liên quan đến tối ưu hóa build bazel
- Bạn cần một miền hoặc công cụ khác ngoài phạm vi này

## Hướng dẫn

- Làm rõ mục tiêu, ràng buộc, và đầu vào cần thiết.
- Áp dụng các thực hành tốt nhất liên quan và xác thực kết quả.
- Cung cấp các bước có thể hành động và xác minh.
- Nếu cần ví dụ chi tiết, mở `resources/implementation-playbook.md`.

## Sử dụng kỹ năng này khi

- Thiết lập Bazel cho monorepos
- Cấu hình remote caching/execution
- Tối ưu hóa thời gian build
- Viết các quy tắc Bazel tùy chỉnh
- Gỡ lỗi các vấn đề build
- Di chuyển sang Bazel

## Các Khái niệm Cốt lõi

### 1. Kiến trúc Bazel

```
workspace/
├── WORKSPACE.bazel       # Các phụ thuộc bên ngoài
├── .bazelrc              # Cấu hình Build
├── .bazelversion         # Phiên bản Bazel
├── BUILD.bazel           # Tệp build gốc
├── apps/
├── webs/
│   └── BUILD.bazel
├── libs/
│   └── utils/
│       └── BUILD.bazel
└── tools/
    └── bazel/
        └── rules/
```

### 2. Các Khái niệm Chính

| Khái niệm   | Mô tả                                          |
| ----------- | ---------------------------------------------- |
| **Target**  | Đơn vị có thể build (thư viện, nhị phân, test) |
| **Package** | Thư mục có tệp BUILD                           |
| **Label**   | Định danh mục tiêu `//path/to:target`          |
| **Rule**    | Định nghĩa cách build một mục tiêu             |
| **Aspect**  | Hành vi build cắt ngang (Cross-cutting)        |

## Mẫu (Templates)

### Mẫu 1: Cấu hình WORKSPACE

```python
# WORKSPACE.bazel
workspace(name = "myproject")

load("@bazel_tools//tools/build_defs/repo:http.bzl", "http_archive")

# Các quy tắc cho JavaScript/TypeScript
http_archive(
    name = "aspect_rules_js",
    sha256 = "...",
    strip_prefix = "rules_js-1.34.0",
    url = "https://github.com/aspect-build/rules_js/releases/download/v1.34.0/rules_js-v1.34.0.tar.gz",
)

load("@aspect_rules_js//js:repositories.bzl", "rules_js_dependencies")
rules_js_dependencies()

load("@rules_nodejs//nodejs:repositories.bzl", "nodejs_register_toolchains")
nodejs_register_toolchains(
    name = "nodejs",
    node_version = "20.9.0",
)

load("@aspect_rules_js//npm:repositories.bzl", "npm_translate_lock")
npm_translate_lock(
    name = "npm",
    pnpm_lock = "//:pnpm-lock.yaml",
    verify_node_modules_ignored = "//:.bazelignore",
)

load("@npm//:repositories.bzl", "npm_repositories")
npm_repositories()

# Các quy tắc cho Python
http_archive(
    name = "rules_python",
    sha256 = "...",
    strip_prefix = "rules_python-0.27.0",
    url = "https://github.com/bazelbuild/rules_python/releases/download/0.27.0/rules_python-0.27.0.tar.gz",
)

load("@rules_python//python:repositories.bzl", "py_repositories")
py_repositories()
```

### Mẫu 2: Cấu hình .bazelrc

```bash
# .bazelrc

# Cài đặt Build
build --enable_platform_specific_config
build --incompatible_enable_cc_toolchain_resolution
build --experimental_strict_conflict_checks

# Hiệu năng
build --jobs=auto
build --local_cpu_resources=HOST_CPUS*.75
build --local_ram_resources=HOST_RAM*.75

# Caching
build --disk_cache=~/.cache/bazel-disk
build --repository_cache=~/.cache/bazel-repo

# Remote caching (tùy chọn)
build:remote-cache --remote_cache=grpcs://cache.example.com
build:remote-cache --remote_upload_local_results=true
build:remote-cache --remote_timeout=3600

# Remote execution (tùy chọn)
build:remote-exec --remote_executor=grpcs://remote.example.com
build:remote-exec --remote_instance_name=projects/myproject/instances/default
build:remote-exec --jobs=500

# Cấu hình Nền tảng
build:linux --platforms=//platforms:linux_x86_64
build:macos --platforms=//platforms:macos_arm64

# Cấu hình CI
build:ci --config=remote-cache
build:ci --build_metadata=ROLE=CI
build:ci --bes_results_url=https://results.example.com/invocation/
build:ci --bes_backend=grpcs://bes.example.com

# Cài đặt Test
test --test_output=errors
test --test_summary=detailed

# Coverage
coverage --combined_report=lcov
coverage --instrumentation_filter="//..."

# Aliases tiện lợi
build:opt --compilation_mode=opt
build:dbg --compilation_mode=dbg

# Nhập cài đặt người dùng
try-import %workspace%/user.bazelrc
```

### Mẫu 3: TypeScript Library BUILD

```python
# libs/utils/BUILD.bazel
load("@aspect_rules_ts//ts:defs.bzl", "ts_project")
load("@aspect_rules_js//js:defs.bzl", "js_library")
load("@npm//:defs.bzl", "npm_link_all_packages")

npm_link_all_packages(name = "node_modules")

ts_project(
    name = "utils_ts",
    srcs = glob(["src/**/*.ts"]),
    declaration = True,
    source_map = True,
    tsconfig = "//:tsconfig.json",
    deps = [
        ":node_modules/@types/node",
    ],
)

js_library(
    name = "utils",
    srcs = [":utils_ts"],
    visibility = ["//visibility:public"],
)

# Tests
load("@aspect_rules_jest//jest:defs.bzl", "jest_test")

jest_test(
    name = "utils_test",
    config = "//:jest.config.js",
    data = [
        ":utils",
        "//:node_modules/jest",
    ],
    node_modules = "//:node_modules",
)
```

### Mẫu 4: Python Library BUILD

```python
# libs/ml/BUILD.bazel
load("@rules_python//python:defs.bzl", "py_library", "py_test", "py_binary")
load("@pip//:requirements.bzl", "requirement")

py_library(
    name = "ml",
    srcs = glob(["src/**/*.py"]),
    deps = [
        requirement("numpy"),
        requirement("pandas"),
        requirement("scikit-learn"),
        "//libs/utils:utils_py",
    ],
    visibility = ["//visibility:public"],
)

py_test(
    name = "ml_test",
    srcs = glob(["tests/**/*.py"]),
    deps = [
        ":ml",
        requirement("pytest"),
    ],
    size = "medium",
    timeout = "moderate",
)

py_binary(
    name = "train",
    srcs = ["train.py"],
    deps = [":ml"],
    data = ["//data:training_data"],
)
```

### Mẫu 5: Quy tắc Tùy chỉnh cho Docker

```python
# tools/bazel/rules/docker.bzl
def _docker_image_impl(ctx):
    dockerfile = ctx.file.dockerfile
    base_image = ctx.attr.base_image
    layers = ctx.files.layers

    # Build the image
    output = ctx.actions.declare_file(ctx.attr.name + ".tar")

    args = ctx.actions.args()
    args.add("--dockerfile", dockerfile)
    args.add("--output", output)
    args.add("--base", base_image)
    args.add_all("--layer", layers)

    ctx.actions.run(
        inputs = [dockerfile] + layers,
        outputs = [output],
        executable = ctx.executable._builder,
        arguments = [args],
        mnemonic = "DockerBuild",
        progress_message = "Building Docker image %s" % ctx.label,
    )

    return [DefaultInfo(files = depset([output]))]

docker_image = rule(
    implementation = _docker_image_impl,
    attrs = {
        "dockerfile": attr.label(
            allow_single_file = [".dockerfile", "Dockerfile"],
            mandatory = True,
        ),
        "base_image": attr.string(mandatory = True),
        "layers": attr.label_list(allow_files = True),
        "_builder": attr.label(
            default = "//tools/docker:builder",
            executable = True,
            cfg = "exec",
        ),
    },
)
```

### Mẫu 6: Truy vấn và Phân tích Phụ thuộc

```bash
# Tìm tất cả phụ thuộc của một mục tiêu
bazel query "deps(//apps/web:web)"

# Tìm phụ thuộc ngược (cái gì phụ thuộc vào cái này)
bazel query "rdeps(//..., //libs/utils:utils)"

# Tìm tất cả các mục tiêu trong một gói
bazel query "//libs/..."

# Tìm các mục tiêu đã thay đổi kể từ commit
bazel query "rdeps(//..., set($(git diff --name-only HEAD~1 | sed 's/.*/"&"/' | tr '\n' ' ')))"

# Tạo đồ thị phụ thuộc
bazel query "deps(//apps/web:web)" --output=graph | dot -Tpng > deps.png

# Tìm tất cả các mục tiêu test
bazel query "kind('.*_test', //...)"

# Tìm các mục tiêu với thẻ cụ thể
bazel query "attr(tags, 'integration', //...)"

# Tính toán kích thước đồ thị build
bazel query "deps(//...)" --output=package | wc -l
```

### Mẫu 7: Thiết lập Remote Execution

```python
# platforms/BUILD.bazel
platform(
    name = "linux_x86_64",
    constraint_values = [
        "@platforms//os:linux",
        "@platforms//cpu:x86_64",
    ],
    exec_properties = {
        "container-image": "docker://gcr.io/myproject/bazel-worker:latest",
        "OSFamily": "Linux",
    },
)

platform(
    name = "remote_linux",
    parents = [":linux_x86_64"],
    exec_properties = {
        "Pool": "default",
        "dockerNetwork": "standard",
    },
)

# toolchains/BUILD.bazel
toolchain(
    name = "cc_toolchain_linux",
    exec_compatible_with = [
        "@platforms//os:linux",
        "@platforms//cpu:x86_64",
    ],
    target_compatible_with = [
        "@platforms//os:linux",
        "@platforms//cpu:x86_64",
    ],
    toolchain = "@remotejdk11_linux//:jdk",
    toolchain_type = "@bazel_tools//tools/jdk:runtime_toolchain_type",
)
```

## Tối ưu hóa Hiệu năng

```bash
# Profile build
bazel build //... --profile=profile.json
bazel analyze-profile profile.json

# Xác định các hành động chậm
bazel build //... --execution_log_json_file=exec_log.json

# Memory profiling
bazel build //... --memory_profile=memory.json

# Bỏ qua bộ nhớ cache phân tích
bazel build //... --notrack_incremental_state
```

## Thực hành Tốt nhất

### Nên (Do's)

- **Sử dụng các mục tiêu chi tiết (fine-grained)** - Caching tốt hơn
- **Ghim các phụ thuộc (Pin dependencies)** - Build có thể tái tạo
- **Bật remote caching** - Chia sẻ các tạo phẩm build
- **Sử dụng visibility một cách khôn ngoan** - Thực thi kiến trúc
- **Viết tệp BUILD cho mỗi thư mục** - Quy ước chuẩn

### Không nên (Don'ts)

- **Đừng sử dụng glob cho deps** - Rõ ràng là tốt hơn
- **Đừng commit các thư mục bazel-\*** - Thêm vào .gitignore
- **Đừng bỏ qua thiết lập WORKSPACE** - Nền tảng của build
- **Đừng bỏ qua cảnh báo build** - Nợ kỹ thuật

## Tài nguyên

- [Tài liệu Bazel](https://bazel.build/docs)
- [Bazel Remote Execution](https://bazel.build/docs/remote-execution)
- [rules_js](https://github.com/aspect-build/rules_js)
