# 状态栏前端开发

**仅使用 MVU 时需要**。打包后，`正则/状态栏界面.html` 仍为占位符，需完成状态栏前端开发。

## 设计原则

- **使用 ICON，不使用 emoji**：前端版面向复杂界面，ICON 在不同主题/尺寸下表现一致，且支持 CSS 调色，与状态栏视觉风格统一
- **尽量解耦**：每个组件只负责一件事，变量读取集中在 store，业务逻辑抽离为独立模块，确保组件可复用、可单独测试

## 开发方式

使用 [tavern_helper_template](https://github.com/StageDog/tavern_helper_template) 开发状态栏前端界面。

## 开发流程

### 1. 准备开发环境

先询问用户是否已克隆过 tavern_helper_template 仓库。如已有，确认仓库位置后直接使用；如没有，执行以下步骤：

```bash
# 克隆模板仓库
git clone https://github.com/StageDog/tavern_helper_template.git
cd tavern_helper_template

# 安装依赖
pnpm install
```

### 2. 复制变量结构

将项目的 `schema.ts` 复制到模板仓库的 `src/{ProjectName}/schema.ts`。

**注意**：两个仓库的 schema.ts 结构互通，可直接复用。

### 3. 开发状态栏界面

在 `src/{ProjectName}/界面/状态栏/` 中开发：

```
界面/状态栏/
├── index.ts        # 入口文件，等待 MVU 初始化
├── index.html      # HTML 模板
├── App.vue         # 主组件
├── store.ts        # 数据存储，使用 defineMvuDataStore
├── global.css      # 全局样式
└── components/     # 子组件
```

**关键代码**：

```typescript
// store.ts - 连接 MVU 变量
import { defineMvuDataStore } from '@util/mvu';
import { Schema } from '../../schema';

export const useDataStore = defineMvuDataStore(Schema, {
  type: 'message',
  message_id: getCurrentMessageId()
});
```

```vue
<!-- App.vue - 读取变量 -->
<script setup lang="ts">
import { useDataStore } from './store';

const store = useDataStore();
// store.stat_data 即为 schema.ts 定义的变量结构
</script>
```

### 4. CSS 色彩变量命名规范

CSS 色彩变量必须使用**功能语义**命名，禁止使用视觉描述命名。

```css
/* ✓ 正确：功能语义 */
:root {
  --c-primary: #4a90d9;
  --c-danger: #e74c3c;
  --c-surface: #f5f5f5;
  --c-text-muted: #888;
  --c-border: #ddd;
}

/* ✗ 错误：视觉描述 */
:root {
  --c-blue: #4a90d9;
  --c-red: #e74c3c;
  --c-light-gray: #f5f5f5;
  --c-gray: #888;
  --c-light-border: #ddd;
}
```

### 5. 本地测试

先询问用户是否安装了 VS Code 的 Live Server 扩展。

如已安装，指导用户在 tavern_helper_template 根目录右键，选择"Open with Live Server"。Live Server 自动注入 CORS 头并启动 HTTP 服务，默认端口 `5500`。

如未安装，自行启动一个带 CORS 头的静态文件服务器，工作目录为 tavern_helper_template 根目录。

同时运行以下命令监听文件改动并自动重新编译：

```bash
pnpm watch
```

#### 配置实时预览

将项目中的 `正则/状态栏界面.html` 临时改为加载本地服务器：

```html
<body>
<script>
$('body').load('http://localhost:5500/dist/{ProjectName}/界面/状态栏/index.html')
</script>
</body>
```

> `dist/{ProjectName}/` 是 `pnpm watch` 的编译输出路径。端口号按实际使用的服务器端口调整。

修改预览正则后，按 `references/packaging.md` 的打包流程生成预览版角色卡，指导用户导入酒馆并在酒馆助手的 `开发` 选项中勾选 `允许监听`，即可实时预览修改效果。

> **开发/生产切换**：本地测试时使用上述 localhost 地址；部署前需将 `正则/状态栏界面.html` 改回 CDN 地址（见步骤 7）。

### 6. 前端打包与部署

```bash
# 打包到 dist 文件夹
pnpm build
```

**部署方式**：

- **GitHub + jsdelivr**（推荐）：推送到 GitHub 仓库，通过 jsdelivr CDN 访问
- **自托管**：部署到任意可访问的 URL

### 7. 更新占位符

部署完成后，修改项目中的 `正则/状态栏界面.html`：

```html
<body>
<script>
$('body').load('https://testingcf.jsdelivr.net/gh/{GH_USER}/{GH_REPO}/dist/{ProjectName}/界面/状态栏/index.html')
</script>
</body>
```

替换：
- `{GH_USER}`：GitHub 用户名
- `{GH_REPO}`：仓库名
- `{ProjectName}`：项目名称

### 8. 重新打包

按 `references/packaging.md` 的打包流程生成正式角色卡。