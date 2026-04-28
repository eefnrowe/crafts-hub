# 前端项目语言参考资料

> 适用于：TypeScript/JavaScript 前端项目，覆盖 React / Vue / Svelte + Vite / Next.js / Nuxt

## 命令表

### 包管理器检测

| 锁文件 | 包管理器 | 安装依赖 | 添加依赖 | 运行 |
|--------|---------|---------|---------|------|
| package-lock.json | npm | `npm ci` | `npm install <pkg>` | `npx <cmd>` |
| pnpm-lock.yaml | pnpm | `pnpm install` | `pnpm add <pkg>` | `pnpm exec <cmd>` |
| bun.lockb | bun | `bun install` | `bun add <pkg>` | `bun run <cmd>` |
| yarn.lock | yarn | `yarn install` | `yarn add <pkg>` | `yarn <cmd>` |

### 工具链命令

| 操作 | 命令 |
|------|------|
| 开发服务器 | `npm run dev` / `pnpm dev` |
| 构建 | `npm run build` |
| 格式化 | `npx prettier --write src/` |
| Lint | `npx eslint src/ --fix` |
| 类型检查 | `npx tsc --noEmit` |
| 测试 | `npx vitest run` 或 `npx jest` |
| 测试（监听） | `npx vitest` |
| 测试覆盖率 | `npx vitest run --coverage` |
| 全部检查 | `npm run lint && npm run type-check && npm run test` |

## 工具链覆盖清单

| 工具 | 覆盖的规则 |
|------|-----------|
| **ESLint** | 代码质量：未使用变量、禁止 any、import 排序、React hooks 规则 |
| **Prettier** | 代码风格：缩进、引号、分号、行宽、尾随逗号 |
| **TypeScript strict** | 类型安全：strictNullChecks、noImplicitAny、strictFunctionTypes |
| **Vitest / Jest** | 测试框架：断言、mock、覆盖率 |
| **Stylelint** | CSS 规范：属性排序、禁止 !important |

## 优先级分层

生成 CLAUDE.md 时按以下优先级填充，空间不足时优先保证 Tier 1。

### Tier 1：核心规则（几乎所有项目）
- 组件模型：仅函数式组件 + hooks（禁止 class 组件）
- any 禁止：tsconfig.json strict + ESLint no-explicit-any
- key 约束：列表渲染使用稳定唯一 key，禁止数组 index

### Tier 2：推荐规则（取决于项目风格）
- 状态管理方案选择及使用边界
- 目录结构约定
- interface vs type 选择
- 命名导出 vs 默认导出
- 副作用管理（useEffect 依赖数组）
- SSR/SSG 及 server/client component 边界（仅 Next.js/Nuxt）

### Tier 3：边缘场景（特定条件才需要）
- 样式方案（Tailwind / CSS Modules / styled-components）
- Suspense / ErrorBoundary 模式
- 可空类型显式标注
- 禁止内联样式

## 前端特有约束（候选规则池）

### 架构类

- **组件模型**：仅函数式组件 + hooks（禁止 class 组件）
- **状态管理**：明确选择（Zustand / TanStack Query / Redux / Pinia）及使用边界
- **路由**：明确路由方案和约定（文件路由 vs 配置路由）
- **目录结构**：`src/components/` / `src/features/` / `src/hooks/` / `src/utils/` 等的组织方式
- **SSR/SSG**：是否使用服务端渲染，server component vs client component 边界

### TypeScript 类

- **interface vs type**：对象形状用 `interface`，联合类型/工具类型用 `type`
- **禁止 enum**：用字面量联合类型替代（`type Status = 'idle' | 'loading' | 'error'`）
- **命名导出 vs 默认导出**：统一选择，说明偏好
- **Props 定义**：interface 内联定义于组件同一文件
- **any 禁止**：`tsconfig.json` 开启 `strict` + ESLint `@typescript-eslint/no-explicit-any`
- **可空类型**：显式标注 `| null` / `| undefined`，不依赖隐式

### 组件规范类

- **文件结构**：组件 + Props interface + 样式在同一文件，或拆分
- **命名**：PascalCase 组件名、camelCase 事件处理器、`use*` hook 命名
- **组合优于 prop drilling**：Context / Composition / Slot 优先于深层 props
- **副作用**：useEffect 依赖数组必须完整，优先 useSyncExternalStore / useMemo
- **key 约束**：列表渲染使用稳定唯一 key，禁止数组 index

### 样式类

- **方案选择**：明确 Tailwind / CSS Modules / styled-components / Vanilla Extract
- **Tailwind**：类名排序（prettier-plugin-tailwindcss），避免过长组合
- **CSS Modules**：文件命名 `*.module.css`
- **禁止内联样式**：除动态计算外不使用 `style={{}}`

## 框架特定规则

### React + Vite

| 规则 | 说明 |
|------|------|
| 组件 | 仅函数式组件 + hooks |
| 状态 | 明确选择：Zustand（全局）/ TanStack Query（服务端）/ useState（局部） |
| 副作用 | 优先 useSyncExternalStore > useMemo > useEffect |
| Suspense | 数据加载使用 Suspense 边界 |
| 错误处理 | ErrorBoundary 组件捕获渲染错误 |
| 测试 | Vitest + React Testing Library，测试用户行为而非实现细节 |

### Next.js (App Router)

| 规则 | 说明 |
|------|------|
| Server/Client | 默认 Server Component，`"use client"` 仅用于交互组件 |
| 数据获取 | Server Component 中直接 async/await，不使用 useEffect |
| 路由 | `app/` 目录文件路由，`layout.tsx` / `page.tsx` / `loading.tsx` |
| Metadata | `generateMetadata()` 替代 `<head>` |

### Vue 3 + Nuxt

| 规则 | 说明 |
|------|------|
| API | Composition API + `<script setup>`（禁止 Options API） |
| 状态 | Pinia（禁止 Vuex） |
| 组件 | `.vue` SFC：`<script setup>` + `<template>` + `<style scoped>` |
| Auto-import | Nuxt auto-imports 行为需在 CLAUDE.md 中说明 |
| 测试 | Vitest + Vue Test Utils |

### Svelte / SvelteKit

| 规则 | 说明 |
|------|------|
| Svelte 5 | Runes（`$state`、`$derived`）替代 Svelte 4 stores |
| 路由 | SvelteKit `src/routes/` 文件路由 |
| 组件 | PascalCase 命名，`.svelte` 文件 |
| 测试 | Vitest + Svelte Testing Library |

## 测试约定

- 框架：Vitest（优先）或 Jest
- 工具：React Testing Library / Vue Test Utils / Svelte Testing Library
- 目录：`__tests__/` 或 `*.test.ts` 同目录
- 命名：`<component>.test.tsx`，函数 `test('should <预期> when <条件>')`
- 原则：测试用户行为而非实现细节（点击、输入、断言 DOM）
- Mock：`vi.mock()` / `jest.mock()`，优先 MSW（Mock Service Worker）拦截网络
- **TDD 流程**（Tier 2，Q7=TDD优先时激活）
  - 红灯→绿灯→重构：先写失败测试 → 最小实现通过 → 重构
  - 使用 `vitest --watch` 或 `jest --watch` 自动运行
  - ✅ 组件测试优先测行为（用户交互），非实现细节
  - ✅ 使用 `testing-library` 查询（`getByRole`、`getByText`），不用 `querySelector`
  - ❌ 禁止先写组件再补测试
- **测试覆盖要求**（Tier 2，Q7=覆盖要求时激活）
  - 配置 `vitest`/`jest` 覆盖率收集，设置阈值（如 ≥ 80%）
  - 覆盖率关注组件和 hooks，不要求样式/配置文件覆盖
  - ✅ CI 中覆盖率检查作为门禁
  - ❌ 禁止为凑覆盖率测试内部实现细节
- **安全边界校验**（Tier 2，Q8=严格边界校验时激活）
  - XSS 防护：禁止 `dangerouslySetInnerHTML`（React）/ `v-html`（Vue），使用 DOMPurify 净化
  - 输入校验：表单提交前做客户端校验 + 服务端二次校验，客户端校验仅为 UX 优化
  - 路由守卫：敏感路由必须有 auth guard，未授权时重定向到登录页
  - ✅ 使用框架提供的安全 API（如 React JSX 自动转义）
  - ❌ 禁止将用户输入直接插入 URL 参数或 HTML 属性

## 命名约定

| 类型 | 约定 | 示例 |
|------|------|------|
| 组件文件 | PascalCase | `SkillRouter.tsx` |
| 工具/hook 文件 | camelCase | `useSkillRouter.ts` |
| 组件名 | PascalCase | `SkillRouter` |
| Props interface | `*Props` | `SkillRouterProps` |
| 事件处理器 | `on*` / `handle*` | `onClick`, `handleSubmit` |
| Hook | `use*` 前缀 | `useSkillRouter` |
| 常量 | UPPER_SNAKE_CASE | `MAX_RETRY_COUNT` |
| CSS 类名 | kebab-case 或 BEM | `skill-router` / `skill-router__item` |
| 测试文件 | `*.test.ts(x)` | `SkillRouter.test.tsx` |
