# 第0章 - Hello World

让我们从一个基本的TypeScript设置和一个hello world程序开始。

本指南使用TypeScript编写（是的，Python版本即将推出）

在每个工作坊步骤的文件编辑之间有许多检查点，
因此即使你对TypeScript不太熟悉，
也应该能够跟上并运行每个示例。

要运行本指南，你需要安装较新版本的nodejs和npm

你可以使用任何你想要的nodejs版本管理器，[homebrew](https://formulae.brew.sh/formula/node)就可以。

    brew install node@20

你应该能看到node版本。

    node --version

复制初始package.json。

    cp ./walkthrough/00-package.json package.json

<details>
<summary>显示文件</summary>

```json
// ./walkthrough/00-package.json
{
    "name": "my-agent",
    "version": "0.1.0",
    "private": true,
    "scripts": {
      "dev": "tsx src/index.ts",
      "build": "tsc"
    },
    "dependencies": {
      "tsx": "^4.15.0",
      "typescript": "^5.0.0"
    },
    "devDependencies": {
      "@types/node": "^20.0.0",
      "@typescript-eslint/eslint-plugin": "^6.0.0",
      "@typescript-eslint/parser": "^6.0.0",
      "eslint": "^8.0.0"
    }
  }
```

</details>

安装依赖。

    npm install

复制tsconfig.json。

    cp ./walkthrough/00-tsconfig.json tsconfig.json

<details>
<summary>显示文件</summary>

```json
// ./walkthrough/00-tsconfig.json
{
    "compilerOptions": {
      "target": "ES2017",
      "lib": ["esnext"],
      "allowJs": true,
      "skipLibCheck": true,
      "strict": true,
      "noEmit": true,
      "esModuleInterop": true,
      "module": "esnext",
      "moduleResolution": "bundler",
      "resolveJsonModule": true,
      "isolatedModules": true,
      "jsx": "preserve",
      "incremental": true,
      "plugins": [],
      "paths": {
        "@/*": ["./*"]
      }
    },
    "include": ["next-env.d.ts", "**/*.ts", "**/*.tsx", ".next/types/**/*.ts"],
    "exclude": ["node_modules", "walkthrough"]
  }
```

</details>

添加.gitignore。

    cp ./walkthrough/00-.gitignore .gitignore

<details>
<summary>显示文件</summary>

```gitignore
// ./walkthrough/00-.gitignore
baml_client/
node_modules/
```

</details>

创建src文件夹。

    mkdir -p src

添加一个简单的hello world index.ts。

    cp ./walkthrough/00-index.ts src/index.ts

<details>
<summary>显示文件</summary>

```ts
// ./walkthrough/00-index.ts
async function hello(): Promise<void> {
    console.log('hello, world!')
}

async function main() {
    await hello()
}

main().catch(console.error)
```

</details>

运行它来验证。

    npx tsx src/index.ts

你应该看到：

    hello, world!
