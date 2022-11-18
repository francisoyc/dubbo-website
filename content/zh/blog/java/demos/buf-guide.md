---
title: "IDL管理工具——Buf"
linkTitle: "IDL管理工具——Buf"
date: 2022-11-12
description: >-
     与简单的 REST/JSON 服务相比，使用 IDL 定义 API 提供了许多好处，如今，Protobuf 是业内最稳定、被广泛采用的 IDL。但就目前情况而言，使用 Protobuf 比使用 JSON 作为数据传输格式要困难得多。
---

## 背景
使用Protobuf会在整个API生命周期中面临许多挑战：
* **API 设计问题：** 编写 Protobuf API 不像编写可维护的基于 REST/JSON 的 API 那样易于理解，如果没有标准， Protobuf API 设计可能出现不一致并影响未来的迭代。
* **缺乏依赖管理：** Protobuf 文件是从 GitHub 复制的可能包含错误内容的文件。在Buf Schema Registry (BSR)之前，没有尝试过统一的跟踪和管理跨文件依赖关系。这就像在没有 npm 的情况下编写 JavaScript，在没有cargo的情况下编写 Rust，在没有modules的情况下编写 Go，以及我们都已经习惯的所有其他编程语言依赖管理器。
* **兼容性问题：** 虽然 Protobuf 承诺向前和向后兼容，但实际上维护向后兼容的 Protobuf API 并没有得到广泛的实践，而且很难实施。
* **生态工具不完善：** 现在已有很多用户体验良好的用于 REST/JSON API 的工具，但是 Protobuf API 缺少这些工具。因此，团队需要定期造轮子并构建自定义工具来复制 JSON 生态。

## Buf 优势
Buf有两大利器：Buf CLI和Buf Schema Registry (BSR)。Buf CLI是一个开源的高性能Protobuf编译器、代码生成器和变更检测器；BSR是一个提供一站式服务的SAAS平台，有免费和收费版本，BSR 使您能够集中维护兼容性和管理依赖性，同时使您的客户能够可靠、高效地使用 API。

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
### 风格指南
#### 要求
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
* 包应该遵循 lower_snake_case 风格
* 包的最后一个层级应该以版本命名
* 文件名应该遵循 lower_snake_case.proto 风格
* 同一包下的所有文件，所有文件选项都应具有相同的值，或者全部未设置

**Imports**
