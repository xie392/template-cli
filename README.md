# 如何开发一个脚手架

你是否好奇如何通过命令 `npm create vite@latest` 来创建项目？使用脚手架可以是我们方便快捷的创建一些初始化的项目，这些基本都是一些常用的技术栈，比如 React、Vue、Angular、Node.js 等。那么，每次创建你是否都需要自己额外配置，如一些路由、存储、插件等？是否可以写一个脚手架专门用于创建属于自己常用的模板？下面，我将带你一起了解如何开发一个简单的脚手架。

# 设计原理

通过脚手架克隆远程自己仓库的模板到本地，在一个项目中创建多个不同分支的模板，例如本项目中我的模板有：

- react
- react-shadcn
- vue
- vue-shadcn

# 1、创建项目

创建一个空文件夹，然后在该文件夹下执行以下命令：

```shell
pnpm init
```

其 `package.json` 文件会自动生成，可以添加一些额外的配置，比如：

```json
{
  // 项目名称，格式： create-[脚手架名]
  "name": "create-xie392-template",
  "version": "0.0.0",
  // +
  "type": "module",
  "description": "",
  // + 等下会安装 tsup
  "scripts": {
    "dev": "tsup --watch",
    "build": "tsup"
  },
  // + 和包名相同，通过 `npm create [脚手架名]` 命令创建项目
  "bin": {
    "create-xie392-template": "./dist/index.js"
  },
  "keywords": [],
  // + npm publish 时，需要包含的文件
  "files": ["dist", "README.md", "package.json"],
  // + 导出的模块
  "exports": "./dist/index.js",
  "author": "",
  // + 远程仓库地址（可选）
  "repository": {
    "type": "git",
    "url": "https://github.com/xie392/npm-template.git"
  },
  "license": "MIT"
}
```

> 【注意】：如果你想使用 `npm create [脚手架名]`命令，你的包名必须是 `create-[脚手架名]`，比如 `create-vite`。

# 2、安装依赖

## 2.1. 安装 `tsup`：

> `tsup` 是一个 TypeScript 打包器，它可以将 TypeScript 编译成 JavaScript，并且可以将依赖项打包到一起。

```shell
pnpm add tsup -D
```

## 2.2、安装 `typescript`

> 本项目使用 `typescript`，所以需要安装 `typescript` 依赖。

```shell
pnpm add typescript -D
```

## 2.3、安装 `commander`

> `commander` 是一个 Node.js 命令行解析器，可以方便的解析命令行参数。

```shell
pnpm add commander
```

## 2.4、安装 `chalk`

> `chalk` 是一个用于在终端上输出颜色的库。

```shell
pnpm add chalk
```

## 2.5、安装 `inquirer`

> `inquirer` 是一个交互式命令行界面，可以让用户输入一些信息。如：输入、选择等

```shell
pnpm add inquirer
```

## 2.6、安装 `degit`（可选）

> `degit` 是用来克隆远程 Git 仓库的命令行工具。（注意：这个库克隆不了 `Gitee` 的仓库，如果想要克隆 `Gitee` 的仓库，可以使用 [giget](https://www.npmjs.com/package/giget)）

```shell
pnpm add degit
```

## 2.7、安装 `ora`（可选）

> `ora` 是一个用来显示命令行进度条的库。

```shell
pnpm add ora
```

# 3、创建文件

## 3.1、创建 `tsup.config.ts` 文件

```typescript
import { defineConfig } from "tsup";

const isDev = process.env.npm_lifecycle_event === "dev";

export default defineConfig({
  clean: true,
  entry: ["src/index.ts"],
  format: ["esm"],
  minify: !isDev,
  target: "esnext",
  outDir: "dist",
  // 如果是 dev 环境，则使用 node 运行打包后的文件
  onSuccess: isDev ? "node dist/index.js" : undefined,
});
```

## 3.2、创建 `tsconfig.json` 文件夹

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "lib": ["DOM", "DOM.Iterable", "ES2021"],
    "module": "Node16",
    "moduleResolution": "nodenext",
    "resolveJsonModule": true,
    "allowJs": true,
    "baseUrl": "./",
    "paths": {
      "@/*": ["./src/*"]
    },
    "checkJs": true
  },
  "include": ["src", "tsup.config.ts"]
}
```

## 3.3、创建 `src/index.ts` 文件

> 如果是可执行文件，必须在顶层加上 `#!/usr/bin/env node` 注释。

```typescript
#!/usr/bin/env node

console.log("hello world");
// ...more
```

## 3.4、添加启动和打包命令

以上添加完毕后记得在 `package.json` 文件中的 `scripts` 中添加 `"dev": "tsup --watch"` 和 `"build": "tsup"` 命令。

```json
{
  "scripts": {
    "dev": "tsup --watch",
    "build": "tsup"
  }
}
```

# 4、初始测试运行

```shell
pnpm dev
```

如果输出 `hello world` 就说明初步的配置已经成功了，那么下面开始编写逻辑代码。

# 5、编写逻辑代码

为了简洁明了，我将逻辑代码放在 `src/index.ts` 文件中，具体实现可以根据自己的需求进行修改。

```typescript
#!/usr/bin/env node

import { Command } from "commander";
import degit from "degit";
import path from "path";
import chalk from "chalk";
import ora from "ora";
import inquirer from "inquirer";

// 终端显示命令行
async function askQuestion() {
  try {
    // 这里可以根据自己的需求添加更多的选项
    // 可以去看 inquirer 官方文档
    const answers = await inquirer.prompt([
      {
        type: "input",
        name: "name",
        message: "What is your name?",
        // 如果没有输入就是下面的默认值
        default: "new-project",
        validate: (fileName: string) => {
          // 可以校验输入的名称是否合法
          const isChinese = /[\u4e00-\u9fa5]/.test(fileName);
          // 如果是中文就提示
          if (isChinese) {
            return "Please enter a valid English name";
          }
          // 根据自己需要添加更多的校验规则
          return true;
        },
      },
      {
        type: "list",
        name: "branch",
        message: "What template do you want to use?",
        // 选择一个
        choices: [
          { name: "React", value: "react" },
          { name: "React-Shadcn", value: "react-shadcn" },
          { name: "Vue", value: "vue" },
          { name: "Vue-Shadcn", value: "vue-shadcn" },
        ],
      },
    ]);
    return answers;
  } catch (error) {
    if (error.name === "ExitPromptError") {
      console.log(chalk.red("Error creating project: User exited prompt"));
    }
    process.exit(1);
  }
}

// 这里写你自己的仓库地址
const REPO = "github:xie392/create-xie392-template";

// 创建项目
async function createProject(answers: { name: string; branch: string }) {
  // 结构项目名和分支名
  const { name, branch } = answers;

  // 使用 degit 工具从指定的 Git 仓库拉取项目模板
  const emitter = degit(`${REPO}#${branch}`, {
    cache: false,
    force: true,
  });

  // 解析目标目录的绝对路径
  const targetDir = path.resolve(process.cwd(), name);

  console.log(chalk.blue(`\nCreating project in ${chalk.green(targetDir)}...`));

  // 显示进度条
  const spinner = ora("Cloning repository...").start();

  try {
    // 克隆仓库到本地
    await emitter.clone(targetDir);
    spinner.succeed("Repository cloned successfully!");
    console.log(chalk.green(`\nProject created successfully!`));
    console.log(chalk.yellow(`\nNext steps:`));
    console.log(chalk.cyan(`  cd ${name}`));
    console.log(chalk.cyan(`  npm install`));
    console.log(chalk.cyan(`  npm run dev`));
  } catch (error) {
    spinner.fail("Error cloning repository.");
    console.error(chalk.red("Error creating project:"), error.message);
  }
}

async function main() {
  try {
    const program = new Command();

    program
      .name("create-xie392-template")
      .version("0.0.0")
      .description("这是一个脚手架创建项目的工具")
      .action(async () => {
        const answer = await askQuestion();
        await createProject(answer);
        process.exit(1);
      });

    program.parse(process.argv);
  } catch (error) {
    console.log(chalk.red(error.message));
  }
}

main();
```

# 6、发布 npm

## 6.1、发布前的检查

```shell
pnpm build
```

1. 检查 `dist` 文件夹是否存在，如果不存在，请先执行 `pnpm build` 命令。
2. 检查 `package.json` 文件中的 `name` 和 `version` 是否正确。
3. 检查 `package.json` 文件中的 `files` 是否正确。
4. 检查 `package.json` 文件中的 `bin` 是否正确。
5. 检查 `package.json` 文件中的 `exports` 是否正确。

## 6.2、发布 npm

这里我希望是通过 `Github action` 去自动的发布。

在这之前先查看一下注册源是否是: `https://registry.npmjs.org/`

查看当前镜像源：

```shell
npm config get registry
```

如果不是，则需要修改镜像源：

```shell
npm config set registry https://registry.npmjs.org/
```

## 6.3、登录 npm

```shell
npm login
```

## 6.4、生成 token

```
npm token create
```

生成成功后复制下来

## 6.5、将 Token 添加到 GitHub 项目中的 Secrets：

- 打开你的 `GitHub` 仓库。
- 进入 `Settings` > `Secrets and variables` > `Actions`。
- 点击 `New repository secret`。
- 添加一个名为 `NPM_TOKEN` 的 `Secret`，并将上一步生成的 Token 作为值保存。

## 6.6、编写工作流

创建 `.github/workflows/publish.yml`

```yaml
name: publish to npm

on:
  # 当创建标签时触发
  push:
    tags:
      - v* # 匹配所有以 `v` 开头的 tag

jobs:
  publish:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 20

      # 配置 npm 的 authToken
      - name: Configure npm
        run: |
          echo "//registry.npmjs.org/:_authToken=${{ secrets.NPM_TOKEN }}" > ~/.npmrc

      # 安装依赖
      - name: Install dependencies
        run: npm install

      # 打包
      - name: Build
        run: npm run build

      # 发布到 npm
      - name: publish to npm
        run: npm publish
        env:
          NODE_AUTH_TOKEN: ${{ secrets.NPM_TOKEN }}
```

## 6.7、测试发布

```shell
git init
git add .
git commit -m "feat: init"
# 注意：你这里还需要先关联你的远程仓库,根据自己需求更改
git remote add origin https://github.com/xie392/create-xie392-template.git
git tag v0.0.1
git push origin master --tags
```

> 如果发布成功或失败都会通过邮箱通知你，你自己也可以看看 `Action` 的进度。

# 7、使用脚手架

```shell
npm create xie392-template
```

# 总结

通过本文，你应该对如何开发一个脚手架有了一个大概的了解，并且学会了如何使用 `tsup` 打包 TypeScript 项目，以及如何发布 npm 包。当然，脚手架的功能远不止这些，还可以实现更多的功能，比如：自定义模板、自动化测试、自动化部署等。学习永无止境！希望这篇文章能帮助到你。

# 参考项目

- [create-t3-app](https://github.com/t3-oss/create-t3-app)
