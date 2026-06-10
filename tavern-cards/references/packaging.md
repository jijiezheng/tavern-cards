# 打包输出与状态栏前端开发

## 打包流程

执行 `node scripts/tavern-cards-forge.mjs pack {project}` 生成 SillyTavern 格式文件。

### 输出格式

| 形式 | 有头像 | 输出格式 | 产物路径 |
|------|--------|---------|---------|
| 角色卡 | 是 | PNG | `cards/{Project}/{Project}.png` |
| 角色卡 | 否 | JSON | `cards/{Project}/{Project}.json` |
| 独立世界书 | - | JSON | `cards/{Project}/{Project}.json` |

### 打包前检查清单

- [ ] 所有条目已注册到 entryManifest
- [ ] 无遗留的 `# 待细化` 注释

如果使用 MVU，还需要检查：

- [ ] MVU 一致性检查通过
- [ ] `validate-mvu` 校验通过
- [ ] schema.ts 与 Zod.txt 同步（运行 `diff` 同步检查命令确认，见 `references/mvu/guide.md#同步检查命令`）
- [ ] MVU 脚本已注册：`node scripts/tavern-cards-forge.mjs query {project} '$.extensions.tavern_helper.scripts.MVU'` 返回非空
- [ ] MVU Zod 脚本已注册：同上，查询路径改为 `$.extensions.tavern_helper.scripts.Zod`
- [ ] MVU 相关正则已注册：查询 `$.regex_scripts.*~` 应包含 `对AI隐藏状态栏`、`状态栏界面`、`对AI隐藏变量更新`、`变量更新中美化`、`变量更新美化` 五条

## 状态栏前端开发

**仅使用 MVU 时需要**。打包后，`正则/状态栏界面.html` 仍为占位符，需完成状态栏前端开发。

### 开发方式

使用 [tavern_helper_template](https://github.com/StageDog/tavern_helper_template) 开发状态栏前端界面。

### 开发流程

#### 1. 准备开发环境

先询问用户是否已克隆过 tavern_helper_template 仓库。如已有，确认仓库位置后直接使用；如没有，执行以下步骤：

```bash
# 克隆模板仓库
git clone https://github.com/StageDog/tavern_helper_template.git
cd tavern_helper_template

# 安装依赖
pnpm install
```

#### 2. 复制变量结构

将项目的 `schema.ts` 复制到模板仓库的 `src/{ProjectName}/schema.ts`：

```bash
cp /path/to/project/schema.ts src/{ProjectName}/schema.ts
```

**注意**：两个仓库的 schema.ts 结构互通，可直接复用。

#### 3. 开发状态栏界面

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

#### 4. CSS 色彩变量命名规范

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

#### 5. 本地测试

```bash
# 启动开发服务器
pnpm watch
```

在 SillyTavern 中配置酒馆助手的调试地址，实时预览效果。

#### 6. 打包与部署

```bash
# 打包到 dist 文件夹
pnpm build
```

**部署方式**：

- **GitHub + jsdelivr**（推荐）：推送到 GitHub 仓库，通过 jsdelivr CDN 访问
- **自托管**：部署到任意可访问的 URL

#### 7. 更新占位符

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

#### 8. 重新打包

```bash
node scripts/tavern-cards-forge.mjs pack {project}
```

## 后续维护

### 内容修改

1. 编辑项目目录中的条目文件
2. 运行 `configure` 和 `pack`

### 结构修改

1. 修改 `创作规划.yaml`
2. 按 `references/resume.md` 流程继续
3. 如涉及 schema 变更，同步更新 tavern_helper_template 中的 schema.ts

### 状态栏界面更新

1. 在 tavern_helper_template 中修改代码
2. 推送到 GitHub（自动触发 CI 打包）
3. jsdelivr CDN 自动更新（可能需要等待缓存刷新）
