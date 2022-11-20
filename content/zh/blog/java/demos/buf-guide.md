---
title: "IDL管理工具——Buf"
linkTitle: "IDL管理工具——Buf"
date: 2022-11-12
description: >-
     与简单的 REST/JSON 服务相比，使用 IDL 定义 API 提供了许多好处，如今，Protobuf 是业内最稳定、被广泛采用的 IDL。但就目前情况而言，使用 Protobuf 比使用 JSON 作为数据传输格式要困难得多。
---

## 现状
使用 Protobuf 会在整个 API 生命周期中面临许多挑战：
* **API 设计问题**： 编写 Protobuf API 不像编写可维护的基于 REST/JSON 的 API 那样易于理解，如果没有标准， Protobuf API 设计可能出现不一致并影响未来的迭代。
* **缺乏依赖管理**： Protobuf 文件是从 GitHub 复制的可能包含错误内容的文件。在 Buf Schema Registry (BSR) 之前，没有尝试过统一的跟踪和管理跨文件依赖关系。这就像在没有 npm 的情况下编写 JavaScript，在没有cargo的情况下编写 Rust 等等。
* **兼容性问题**： 虽然 Protobuf 承诺向前和向后兼容，但实际上维护向后兼容的 Protobuf API 并没有得到广泛的实践，而且很难实施。
* **生态工具不完善**： 现在已有很多用户体验良好的用于 REST/JSON API 的工具，但是 Protobuf API 缺少这些工具。因此，团队需要定期造轮子并构建自定义工具来复制 JSON 生态。

## Buf 优势
Buf 有两大利器：`Buf CLI` 和 `Buf Schema Registry (BSR)`。`Buf CLI` 是一个开源的高性能 Protobuf 编译器、代码生成器和变更检测器；`BSR` 是一个提供一站式服务的 SAAS 平台，有免费和收费版本，`BSR` 使您能够集中维护兼容性和管理依赖性，同时使您的客户能够可靠、高效地使用 API。

Buf 解决了上述许多问题，Buf 可帮助您的团队在整个生命周期中使用 Protobuf API，无论您是为关键客户构建新 API 还是依赖另一个团队公开的 API，最终允许您将大部分时间和精力从管理 Protobuf 文件转移到实现核心功能和基础设施上来。

**For Producers**

* 一致地生成 Protocol Buffers API，利用 Buf 的直观工具集快速迭代并实施最佳实践。
* 可靠地向用户分发 API，使他们能够针对您的最新版本进行开发，无需昂贵的沟通成本。
* 通过可浏览的中央注册表、可显示的内容和生成的文档提高可发现性。

**For Consumers**

* 根据清晰的后端定义进行开发，使团队能够在通用模式上并行工作。
* 使用 Buf 的直观工具消除摩擦，替换自定义、复杂的构建脚本。
* 解锁生成的 CLI、运行时验证、自定义插件、模拟服务器、压力测试等功能。

## 最佳实践
### 规约
**Files and packages**
* 所有文件都应该定义一个包
* 同一个包的所有文件应该在同一个目录中，所有文件都应位于与其包名匹配的目录中。如下所示：
```
.
└── proto
    ├── buf.yaml
    └── foo
        └── bar
            ├── bat
            │   └── v1
            │       └── bat.proto // package foo.bar.bat.v1
            └── baz
                └── v1
                    ├── baz.proto         // package foo.bar.baz.v1
                    └── baz_service.proto // package foo.bar.baz.v1
```
* 包应遵循 `lower_snake_case` 风格
* 包的最后一个层级应该以版本命名
* 文件名应遵循 `lower_snake_case.proto` 风格
* 同一包下的所有文件，所有文件选项都应具有相同的值，或者全部不设置

**Imports**
* 不要将 Import 声明为 `public` 或 `weak` 类型的

**Enums**
* 枚举不应设置 `allow_alias` 选项
* 枚举名称应遵循 `PascalCase` 风格
* 枚举值名称遵循 `UPPER_SNAKE_CASE` 风格
* 枚举值名称应以枚举名称的 `UPPER_SNAKE_CASE` 为前缀，示例：枚举名叫 `FooBar`，枚举值应该以 `FOO_BAR_` 开头
* 所有枚举的零值应以 `_UNSPECIFIED` 为后缀，示例：枚举名叫 `FooBar`，0值应该定义为FOO_BAR_UNSPECIFIED = 0;

示例：
```
// PetType represents the different types of pets in the pet store.
enum PetType {
  PET_TYPE_UNSPECIFIED = 0;
  PET_TYPE_CAT = 1;
  PET_TYPE_DOG = 2;
  PET_TYPE_SNAKE = 3;
  PET_TYPE_HAMSTER = 4;
}
```

**Messages**
* message 名应遵循 `PascalCase` 风格
* 字段名应遵循 `lower_snake_case` 风格
* oneof 命名应遵循 `lower_snake_case` 风格，更多 oneof 信息请参考 [oneof](https://developers.google.com/protocol-buffers/docs/proto3#oneof)

示例：
```
// Pet represents a pet in the pet store.
message Pet {
  PetType pet_type = 1;
  string pet_id = 2;
  string name = 3;
  google.type.DateTime created_at = 4;
}

message GetPetRequest {
  string pet_id = 1;
}

message GetPetResponse {
  Pet pet = 1;
}

message SampleMessage {
  oneof test_oneof {
    string name = 4;
    SubMessage sub_message = 9;
  }
}
```

**Services**
* 服务名应遵循 `PascalCase` 风格
* 服务名应以 `Service` 为后缀
* RPC 名应遵循 `PascalCase` 风格
* 所有 RPC 请求和响应 message 在 Protobuf schema 中保持唯一
* 所有 RPC 请求和响应 message 都应以 RPC 命名，可以将它们命名为 MethodNameRequest、MethodNameResponse 或 ServiceNameMethodNameRequest、ServiceNameMethodNameResponse

```
service PetStoreService {
  rpc GetPet(GetPetRequest) returns (GetPetResponse) {}
  rpc PutPet(PutPetRequest) returns (PutPetResponse) {}
  rpc DeletePet(DeletePetRequest) returns (DeletePetResponse) {}
  rpc PurchasePet(PurchasePetRequest) returns (PurchasePetResponse) {}
}
```

buf 提供了很多可配置在 `buf.yaml` 的 lint rules，使用 `buf lint` 命令可以列出不符合以上规约的文件，更多详情请参考 [Rules](https://docs.buf.build/lint/rules)  


### 建议
* 为你的 Protobuf schema 设置变更检测，如何设置可参考 [breaking change detector documentation](https://docs.buf.build/breaking/overview)
* 使用 `//` 而不是 `\* *\` 进行注释
* 注释尽量完整，不要使用行内注释，而是注释在上面
* 命名时避免用到被广泛使用的关键字，尤其是包名。例如，如果包名是 `foo.internal.bar`，在 Go 语言里 `internal` 包将不能被其他包导入 
* 文件内容按照下面顺序排列：
     * License header (if applicable)
     * File overview
     * Syntax
     * Package
     * Imports (sorted)
     * File options
     * Everything else
 ```
 syntax = "proto3";

package pet.v1;

import "google/type/datetime.proto";
import "payment/v1alpha1/payment.proto";

// PetType represents the different types of pets in the pet store.
enum PetType {
  PET_TYPE_UNSPECIFIED = 0;
  PET_TYPE_CAT = 1;
  PET_TYPE_DOG = 2;
  PET_TYPE_SNAKE = 3;
  PET_TYPE_HAMSTER = 4;
}
 ```
 
 * 对重复字段使用复数名称
 * 尽可能以类型命名字段，例如：对于 `FooBar` 类型字段，没有特殊情况应将字段命名为 foo_bar 
 * 避免使用嵌套 `enum` 和嵌套 `message`  


### 插件集成
通过插件和编辑器的集成可以达到 `buf lint` 的效果，在保存件时输出错误信息。使用插件需要确保安装了 buf，目前支持以下编辑器的集成：

**Vim**

使用 [vim-plug](https://github.com/junegunn/vim-plug) 下载 [vim-buf](https://github.com/bufbuild/vim-buf) 插件，在 `.vimrc` 添加以下内容：
```
Plug 'dense-analysis/ale'
Plug 'bufbuild/vim-buf'
let g:ale_linters = {
\   'proto': ['buf-lint',],
\}
let g:ale_lint_on_text_changed = 'never'
let g:ale_linters_explicit = 1
```  


**Visual Studio Code**

可以在 Visual Studio Code 内置的插件市场里面下载名为 `Buf` 的插件，也可以从 [网页](https://marketplace.visualstudio.com/items?itemName=bufbuild.vscode-buf) 下载  


**JetBrains IDEs**

JetBrains IDEs(如 IntelliJ IDEA) 需要通过在IDE里配置一个 `File Watcher` 来达到效果，如何配置可参考 [File Watchers in IntelliJ IDEA](https://www.jetbrains.com/help/idea/tutorial-file-watchers-in-product.html) ，具体值如下：

|  Setting   | Value  |
|  ----      | ----   |
| Program           | `buf`                    |
| Arguments	     | `lint --path $FilePath$` |
| Working directory | `$ProjectFileDir$`       |
| Output filters    | `$FILE_PATH$:$LINE$:$COLUMN$:$MESSAGE$` |



