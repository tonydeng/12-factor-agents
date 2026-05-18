# Walkthroughgen

Walkthroughgen是一个用于创建演练、教程、readme和文档的工具。它通过从简单的YAML配置生成markdown和工作目录来帮助您维护分步指南。

## 特性

- 📝 **Markdown生成**：创建包含差异、代码块和可折叠部分的精美markdown文件
- 📁 **工作目录**：为演练的每个部分生成单独的目录
- 🔄 **增量更改**：跟踪和显示步骤之间的更改
- 🎯 **多个目标**：输出到markdown、按部分文件夹和最终项目状态
- 📦 **文件管理**：复制文件、创建目录和运行命令
- 🔍 **丰富差异**：显示文件版本之间的有意义差异
- 📚 **部分README**：生成每个部分的文档

## 安装

```bash
npm install -g walkthroughgen
```

## 快速开始

1. 创建一个`walkthrough.yaml`文件：

```yaml
title: "My Tutorial"
text: "A step-by-step guide"
targets:
  - markdown: "./walkthrough.md"
    onChange:
      diff: true
      cp: true
  - folders:
      path: "./by-section"
      final:
        dirName: "final"
sections:
  - name: setup
    title: "Initial Setup"
    steps:
      - file: {src: ./files/package.json, dest: package.json}
      - command: "npm install"
```

2. 运行生成器：

```bash
walkthroughgen generate walkthrough.yaml
```

## 目录结构

一个典型的演练项目如下所示：

```
my-tutorial/
├── walkthrough/          # 每个步骤的源文件
│   ├── 00-package.json
│   ├── 01-index.ts
│   └── 02-config.ts
├── walkthrough.yaml     # 演练配置
└── build/              # 生成的输出
    ├── by-section/    # 按部分的工作目录
    │   ├── 00-setup/
    │   └── 01-config/
    ├── final/         # 最终项目状态
    └── walkthrough.md # 生成的markdown
```

## Walkthrough.yaml配置

### 顶层字段

- `title`：演练的标题
- `text`：介绍文字
- `targets`：输出配置
- `sections`：教程部分

### 目标

#### Markdown目标

```yaml
targets:
  - markdown: "./output.md"
    onChange:
      diff: true  # 显示更改文件的差异
      cp: true    # 显示cp命令
    newFiles:
      cat: false  # 不显示文件内容
      cp: true    # 显示cp命令
```

#### 文件夹目标

```yaml
targets:
  - folders:
      path: "./by-section"        # 部分文件夹的基础路径
      skip: ["cleanup"]          # 要跳过的部分
      final:
        dirName: "final"        # 最终状态目录的名称
```

### 部分

每个部分代表教程中的一个逻辑步骤：

```yaml
sections:
  - name: setup              # 用于文件夹命名和跳过数组
    title: "Initial Setup"   # 显示标题
    text: "Setup steps..."   # 部分描述
    steps:
      # ... 步骤 ...
```

### 步骤

步骤定义要采取的操作：

#### 文件复制
```yaml
steps:
  - text: "Copy package.json"
    file:
      src: ./files/package.json
      dest: package.json
```

#### 目录创建
```yaml
steps:
  - text: "Create src directory"
    dir:
      create: true
      path: src
```

#### 命令执行
```yaml
steps:
  - text: "Install dependencies"
    command: "npm install"
    incremental: true  # 在构建文件夹目标时运行
```

#### 命令结果
```yaml
steps:
  - command: "npm run test"
    results:
      - text: "You should see:"
        code: |
          All tests passed!
```

## 生成的输出

### Markdown特性

- **文件差异**：显示版本之间的更改
- **复制命令**：易于遵循的文件复制说明
- **可折叠部分**：隐藏/显示文件内容
- **代码高亮**：各种语言的语法高亮

示例markdown输出：

~~~markdown
# Initial Setup

Copy the package.json:

    cp ./files/package.json package.json

<details>
<summary>show file</summary>

```json
{
  "name": "my-project",
  "version": "1.0.0"
}
```
</details>

Install dependencies:

    npm install

You should see:

    added 123 packages
~~~

### 部分文件夹

`folders`目标创建：

1. 每个部分的目录
2. 特定部分的README.md文件
3. 工作项目状态
4. 可选的最终状态目录

## 示例

有关完整示例，请参阅[examples](./examples)目录：

- [TypeScript CLI](./examples/typescript)：基本TypeScript项目设置
- [Walkthroughgen](./examples/walkthroughgen)：自文档化示例

## 技巧

1. 使用有意义的部分名称 - 它们会成为文件夹名称
2. 在步骤文本中包含上下文
3. 对修改状态的命令使用`incremental: true`
4. 利用差异来突出重要更改
5. 使用`skip`数组从输出中排除设置/清理部分

## 贡献

欢迎贡献！请阅读[CONTRIBUTING.md](./CONTRIBUTING.md)了解详情。

## 许可证

本项目根据MIT许可证获得许可 - 有关详细信息，请参阅[LICENSE](./LICENSE)文件。
