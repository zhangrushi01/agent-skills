# 项目架构详解

## 目录结构

```
agent-skills-manager/
├── src/                    # 源代码
│   ├── extension.ts        # 插件入口
│   ├── types.ts            # 类型定义
│   ├── github/             # GitHub API 层
│   ├── services/           # 业务服务层
│   ├── views/              # UI 视图层
│   └── test/               # 测试文件
├── resources/              # 静态资源（图标）
├── dist/                   # 编译输出
├── package.json            # 插件清单
├── tsconfig.json           # TS 配置
└── esbuild.js              # 打包配置
```

## 核心模块

### extension.ts - 插件入口

**职责**: 插件生命周期管理、命令注册、视图初始化

**关键函数**:
- `activate(context)`: 插件激活，初始化所有服务和视图
- `deactivate()`: 插件停用

**初始化流程**:
1. 创建 SkillsClient 实例
2. 创建 InstallationService 实例
3. 创建视图 Provider（Marketplace、Installed）
4. 注册所有命令
5. 设置文件监听器（监听 skills 目录变化）

### types.ts - 类型定义

**核心接口**:
```typescript
interface Skill {
  name: string;
  description: string;
  path: string;
  repository: SkillRepository;
}

interface SkillRepository {
  owner: string;
  repo: string;
  branch: string;
  path: string;
}

interface InstalledSkill {
  name: string;
  path: string;
  description?: string;
}
```

### github/skillsClient.ts - GitHub API 客户端

**职责**: 与 GitHub API 交互，获取技能列表和文件内容

**关键方法**:
- `getSkills()`: 获取所有技能列表
- `getSkillContent(skill)`: 获取技能内容
- `searchSkills(query)`: 搜索技能

**缓存机制**: 使用内存缓存减少 API 调用

### services/installationService.ts - 安装服务

**职责**: 技能的安装和卸载

**关键方法**:
- `installSkill(skill)`: 安装技能到本地
- `uninstallSkill(skill)`: 卸载本地技能
- `getInstalledSkills()`: 获取已安装技能列表
- `openSkillFolder(skill)`: 在文件管理器中打开

**安装位置**: 由配置项 `agentSkills.installLocation` 决定
- `.github/skills/` 或 `.claude/skills/`

### views/ - 视图层

#### marketplaceProvider.ts
- 实现 `TreeDataProvider<SkillItem>`
- 显示市场中可用的技能
- 支持搜索过滤

#### installedProvider.ts
- 实现 `TreeDataProvider<InstalledSkillItem>`
- 显示已安装的技能
- 监听文件系统变化自动刷新

#### skillDetailPanel.ts
- 使用 WebviewPanel 显示技能详情
- 渲染 Markdown 内容
- 提供安装/卸载按钮

## 数据流

```
用户操作
    ↓
extension.ts (命令处理)
    ↓
services/ (业务逻辑)
    ↓
github/ (API 调用)
    ↓
views/ (UI 更新)
```

## VSCode API 使用

### 常用 API

```typescript
// 命令注册
vscode.commands.registerCommand(id, callback)

// 配置读取
vscode.workspace.getConfiguration('agentSkills')

// 消息提示
vscode.window.showInformationMessage(msg)
vscode.window.showErrorMessage(msg)
vscode.window.showWarningMessage(msg)

// 输入框
vscode.window.showInputBox(options)

// 快速选择
vscode.window.showQuickPick(items, options)

// 进度条
vscode.window.withProgress(options, task)

// 文件系统
vscode.workspace.fs.readFile(uri)
vscode.workspace.fs.writeFile(uri, content)
vscode.workspace.fs.delete(uri)

// TreeView
vscode.window.createTreeView(id, options)
vscode.window.registerTreeDataProvider(id, provider)
```

### 视图容器配置 (package.json)

```json
{
  "contributes": {
    "viewsContainers": {
      "activitybar": [{
        "id": "baidu-search-agent-skills",
        "title": "Agent Skills",
        "icon": "resources/skills-icon.svg"
      }]
    },
    "views": {
      "baidu-search-agent-skills": [
        { "id": "agentSkills.marketplace", "name": "Marketplace" },
        { "id": "agentSkills.installed", "name": "Installed" }
      ]
    }
  }
}
```

## 配置项

| 配置项 | 类型 | 说明 |
|-------|------|------|
| `agentSkills.skillRepositories` | array | 技能仓库源列表 |
| `agentSkills.installLocation` | string | 安装位置 |
| `agentSkills.githubToken` | string | GitHub Token |
| `agentSkills.cacheTimeout` | number | 缓存超时（秒）|

## 命令列表

| 命令 ID | 功能 |
|--------|------|
| `agentSkills.install` | 安装技能 |
| `agentSkills.uninstall` | 卸载技能 |
| `agentSkills.refresh` | 刷新视图 |
| `agentSkills.viewDetails` | 查看详情 |
| `agentSkills.search` | 搜索技能 |
| `agentSkills.clearSearch` | 清除搜索 |
| `agentSkills.openSkillFolder` | 打开文件夹 |
| `agentSkills.focusMarketplace` | 聚焦市场视图 |
