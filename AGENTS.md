# 开发者 Agent 指南 (AGENTS.md)

欢迎，Agent。本代码库遵循严格的模块化架构和高质量设计标准。在进行所有代码贡献时，请务必遵循以下规则。

## 1. 环境与技术栈

- **运行时/包管理器**: [Bun](https://bun.sh) (首选), Node.js (备选)。
- **框架**: Next.js (App Router)。
- **语言**: TypeScript (严格模式)。
- **样式**: Tailwind CSS, CSS Modules 或 Styled Components (根据特性选择)。
- **状态/逻辑**: 使用 Zod 进行数据校验，使用 Server Actions 处理数据变更 (mutations)。
- **数据库**: Prisma 或 Drizzle (请查看 `src/lib/db.ts`)。

## 2. 关键命令

由于本项目中存在 `pnpm-lock.yaml`，请始终使用 `pnpm`（除非明确指定使用 `bun`）。

### 开发模式 (至关重要)
- **启动开发服务器**: `sh scripts/dev.sh`
  *在开发过程中，请始终通过该脚本启动开发模式，以便用户能够持续观看开发效果。*

### 构建与代码检查
- **安装依赖**: `pnpm install`
- **构建项目**: `pnpm run build`
- **代码检查**: `pnpm run lint`
- **类型检查**: `pnpm tsc --noEmit`

### 测试
- **运行所有测试**: `pnpm test`
- **运行单个测试**: `pnpm test <path-to-file>`
- **监听模式**: `pnpm test --watch`

## 3. 架构设计

我们在 `src/` 中采用模块化结构。在开发过程中，必须准确遵循当前的目录结构设计进行开发，确保代码的组织清晰和职责单一。请避免将业务逻辑直接放在 `src/app/` 目录下。

### 就近原则 (Co-location)
逻辑相关的代码应该放在一起。如果一个组件仅被 `billing` 特性使用，它必须存放在 `src/features/billing/components/` 目录下。

## 4. 代码风格指南

### 命名规范
- **目录/文件**: `kebab-case` 短横线命名法 (例如：`user-profile`, `auth-service.ts`)。
- **React 组件**: `PascalCase` 大驼峰命名法 (例如：`UserProfile.tsx`)。
- **函数/变量**: `camelCase` 小驼峰命名法。
- **Server Actions**: 以 `Action` 为后缀 (例如：`createPostAction`)。
- **常量**: `UPPER_SNAKE_CASE` 全大写下划线命名法。

### 导入规范
- 使用带有 `@/` 别名的绝对路径 (例如：`import { db } from '@/lib/db'`)。
- 分组导入：内置模块 -> 外部依赖 -> 内部依赖 (@/) -> 相对路径。
- 避免使用过深的相对路径 (如 `../../../../`)。

### 类型与数据校验
- 对于跨越边界的输入 (Server Actions, API 路由)，**始终**使用 Zod 进行校验。
- 优先使用 `interface` 定义对象形状，使用 `type` 定义联合类型/类型别名。
- 避免使用 `any`。如果类型确实是动态的，请使用 `unknown`。

## 5. 后端组件化 (服务层)

- **核心规则**: 绝不要在 Route Handlers (路由处理程序) 或 Server Actions 中直接编写数据库查询。
- **服务层**: 所有的业务逻辑和数据库调用都必须进入 `src/services/`。
- **模式示例**:
  ```typescript
  // src/services/post-service.ts
  export const PostService = {
    async create(data: CreatePostDTO) {
      return await db.post.create({ data });
    }
  };
  ```

## 6. Server Actions & Mutations (数据变更)

- 所有的 mutations (POST/PUT/DELETE) 均使用 Server Actions (`'use server'`)。
- **工作流**:
  1. 使用 Zod 校验输入数据。
  2. 调用对应的 Service (服务)。
  3. 根据需要执行 `revalidatePath` 或 `redirect`。
  4. 返回标准化的响应对象：`{ success: boolean, data?: T, error?: string }`。

## 7. 前端设计与美学

避免“AI 生成感”的廉价视觉风格。每个 UI 都应感觉经过精心设计并具有视觉冲击力。

- **排版 (Typography)**: 
  - 选择有特色、有趣的字体。
  - 避免使用普通的字体栈如 Arial/Inter/Roboto。
  - 将独特的展示字体 (例如：Space Grotesk, Cal Sans) 与干净易读的正文字体搭配使用。
- **色彩与主题 (Color & Theme)**:
  - 所有的颜色都使用 CSS 变量。
  - 确定一个大胆的美学方向 (例如：粗野主义 Brutalist、极简主义 Minimalism、复古未来主义 Retro-futurism)。
  - 使用高对比度的强调色，避免使用胆怯、浑浊的调色板。
- **动效与交互 (Motion & Interaction)**: 
  - 在 React 中使用 `framer-motion`，或者在纯 HTML/CSS 中使用 CSS 过渡。
  - 专注于“令人愉悦的微交互”：交错展开、平滑的悬停状态以及有意义的过渡效果。
- **布局 (Layout)**: 
  - 在适当的时候打破常规网格。
  - 慷慨地使用留白，或有控制地保持内容密度。
  - 尝试不对称、元素重叠和对角线流向。
- **组件 (Components)**: 
  - 对于数据繁重的部分，优先使用 Server Components。
  - 仅在叶子节点严格使用 `'use client'` 以实现交互。

## 8. 错误处理与校验

- **输入校验**: 对每一个跨越边界的输入（Server Action、API、表单）都使用 Zod。
- **服务错误**: Services 应该抛出特定的错误，或返回 Result 类型。
- **Action 响应**: Server Actions 必须返回一致的数据形状：
  ```typescript
  export type ActionResponse<T = any> = 
    | { success: true; data: T }
    | { success: false; error: string; issues?: z.ZodIssue[] };
  ```
- **UI 反馈**: 使用 `sonner` 或类似的 toast 库来提供成功/错误反馈。始终向用户提供清晰、可操作的错误信息。

## 9. 实施工作流 (步骤指导)

当实施一项新特性时，请遵循以下顺序：

1.  **分析**: 确定业务领域，并在 `src/features/` 中创建一个新的特性目录。
2.  **脚手架**: 运行 `.opencode/skills/fullstack-architect/scripts/scaffold.sh` 生成目录结构。
3.  **模式**: 首先定义 Zod schemas 和数据库模型。
4.  **服务**: 在 `src/services/` 中实现业务逻辑。
5.  **Action**: 在 `src/features/<feature>/actions.ts` 中创建 Server Actions。
6.  **UI 组件**: 构建 UI 组件，尽可能从 Server Components 开始。
7.  **验证**: 运行 `pnpm test` 和 `pnpm run lint`。

## 10. Agent 安全性与主动性

- **迭代开发**: 所有的开发都**必须**是迭代式的，并且基于当前项目。不要在项目结构之外构建全新的独立应用程序；相反，应该持续演进现有的代码库。
- **目录验证**: 在创建文件之前，请先验证父目录是否存在。
- **不回退原则**: 除非更改导致了错误，否则不要回退 (revert) 更改。
- **自我纠正**: 如果 build/lint/test 失败，请在继续之前立即修复它。
- **文档**: 仅在用户明确要求时才创建文档。

---
*本文档是一个动态文档 (living document)。请随着项目约定的演进而更新它。*