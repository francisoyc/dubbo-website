---
title: "Protobuf 管理工具 —— Buf"
linkTitle: "Protobuf 管理工具 —— Buf"
date: 2022-11-12
description: >-
     与简单的 REST/JSON 服务相比，使用 IDL 定义 API 提供了许多好处，如今，Protobuf 是业内最稳定、被广泛采用的 IDL。但就目前情况而言，使用 Protobuf 比使用 JSON 作为数据传输格式要困难得多。
---

## 现状
使用 Protobuf 会在整个 API 生命周期中面临许多挑战：
* API 设计和兼容性： Protobuf API 不像 REST/JSON 的 API 那么好理解，如果没有标准或规范， Protobuf API 设计可能出现不一致并影响未来的迭代。
* 缺乏依赖管理： Protobuf 文件可以导入需要在编译时提供给编译器的依赖项。开发人员需要自己弄清楚如何找到现有的 Protobuf 定义、管理依赖项并确保它们是最新的。
* 代码生成&分发困难：使用 protoc 和相关插件生成代码学习成本高，同时开发人员需要用相同的配置确保环境配置正确，通常需要专门的团队负责这个事。
* 生态工具不完善：REST/JSON API 有很多用户体验不错的工具，但是 Protobuf API 比较缺少。因此，团队需要定期造轮子并构建自定义工具来复制 JSON 生态。

## Buf 介绍
Buf 有两大利器：`Buf CLI` 和 `Buf Schema Registry (BSR)`。`Buf CLI` 是一个开源的高性能 Protobuf 编译器、代码生成器和变更检测器；`BSR` 是一个提供一站式服务的 SAAS 平台，有免费和收费版本，`BSR` 使您能够集中维护兼容性和管理依赖性，同时使您的客户能够可靠、高效地使用 API。

### Buf CLI 优势
* 相比与 `protoc` 更容易上手，性能更好，更加稳定
* 提供了文档化的 `yaml` 配置文件，使用更加简单
* 提供了变更检测能力，防止 API 被改坏
* 与 `BSR` 无缝集成，远程管理依赖，实现 API 与他人的顺畅共享
* 易于与现有 CI 流程集成
* 由 `proto/gRPC` 生态系统专家构建和维护

### BSR 优势
* 提供了适用于 gRPC 和 Protobuf 服务的简洁、交互式 Web UI，功能强大
* 根据 Protobuf schema 自动生成易于阅读的文档，无需再单独编写文档
* 为 Protobuf 提供了依赖管理解决方案
* 为任何版本的 API 自动生成一致的客户端和服务端 libraries 

### 核心命令
```
# 生成 buf.yaml
buf mod init

# 编译 proto 文件
buf build

# 列出 proto 文件
buf ls-files

# 检查 proto 文件是否满足配置的 lint rules，不满足的文件会被输出到控制台
buf lint

# 检查文件变更
buf breaking --against 

# 生成代码
buf generate
```

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
// Copyright 2015 The gRPC Authors
//
// Licensed under the Apache License, Version 2.0 (the "License");
// you may not use this file except in compliance with the License.
// You may obtain a copy of the License at
//
//     http://www.apache.org/licenses/LICENSE-2.0
//
// Unless required by applicable law or agreed to in writing, software
// distributed under the License is distributed on an "AS IS" BASIS,
// WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
// See the License for the specific language governing permissions and
// limitations under the License.

// The canonical version of this proto can be found at
// https://github.com/grpc/grpc-proto/blob/master/grpc/health/v1/health.proto

syntax = "proto3";

package grpc.health.v1;

option csharp_namespace = "Grpc.Health.V1";
option go_package = "google.golang.org/grpc/health/grpc_health_v1";
option java_multiple_files = true;
option java_outer_classname = "HealthProto";
option java_package = "io.grpc.health.v1";

message HealthCheckRequest {
  string service = 1;
}

message HealthCheckResponse {
  enum ServingStatus {
    UNKNOWN = 0;
    SERVING = 1;
    NOT_SERVING = 2;
    SERVICE_UNKNOWN = 3;  // Used only by the Watch method.
  }
  ServingStatus status = 1;
}

service Health {
  // If the requested service is unknown, the call will fail with status
  // NOT_FOUND.
  rpc Check(HealthCheckRequest) returns (HealthCheckResponse);

  // Performs a watch for the serving status of the requested service.
  // The server will immediately send back a message indicating the current
  // serving status.  It will then subsequently send a new message whenever
  // the service's serving status changes.
  //
  // If the requested service is unknown when the call is received, the
  // server will send a message setting the serving status to
  // SERVICE_UNKNOWN but will *not* terminate the call.  If at some
  // future point, the serving status of the service becomes known, the
  // server will send a new message with the service's serving status.
  //
  // If the call terminates with status UNIMPLEMENTED, then clients
  // should assume this method is not supported and should not retry the
  // call.  If the call terminates with any other status (including OK),
  // clients should retry the call with appropriate exponential backoff.
  rpc Watch(HealthCheckRequest) returns (stream HealthCheckResponse);
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



