---
name: vscode-plugin-dev
description: VSCode 插件开发助手。当用户请求修改、新增、删除插件功能时使用此 skill。适用场景：(1) 修改现有功能代码 (2) 添加新功能/命令/视图 (3) 修复 Bug (4) 重构代码 (5) 添加测试。此 skill 指导 Agent 完成代码定位、修改、测试的完整开发流程。
---

# VSCode Plugin Development Skill

指导 Agent 完成 VSCode 插件的功能开发，包括代码定位、修改和测试。

## 项目概览

这是一个 Agent Skills Manager VSCode 插件，用于管理和安装 AI Agent Skills。

**技术栈**: TypeScript 5.9+, VSCode API ^1.104.0, ESBuild, Mocha

**核心目录结构**:
```
src/
├── extension.ts          # 插件入口（激活/注销）
├── types.ts              # 类型定义
├── github/
│   └── skillsClient.ts   # GitHub API 客户端
├── services/
│   └── installationService.ts  # 技能安装服务
├── views/
│   ├── marketplaceProvider.ts  # 市场视图
│   ├── installedProvider.ts    # 已安装视图
│   └── skillDetailPanel.ts     # 详情面板
└── test/
    └── extension.test.ts # 测试文件
```

## 开发工作流

### 第一步：理解需求

1. 明确用户需求的功能范围
2. 确认是修改现有功能还是新增功能
3. 如有歧义，向用户确认

### 第二步：定位代码

根据功能类型定位到对应文件：

| 功能类型 | 相关文件 |
|---------|---------|
| 命令注册/插件生命周期 | `src/extension.ts` |
| 类型/接口定义 | `src/types.ts` |
| GitHub API/技能获取 | `src/github/skillsClient.ts` |
| 安装/卸载逻辑 | `src/services/installationService.ts` |
| 市场视图 | `src/views/marketplaceProvider.ts` |
| 已安装视图 | `src/views/installedProvider.ts` |
| 详情面板 WebView | `src/views/skillDetailPanel.ts` |
| 配置项定义 | `package.json` (contributes.configuration) |
| 命令定义 | `package.json` (contributes.commands) |
| 视图定义 | `package.json` (contributes.views) |

**定位策略**:
1. 先根据功能类型选择可能的文件
2. 使用 Grep 搜索关键词确认位置
3. 阅读相关代码理解上下文

### 第三步：实现修改

#### 修改现有功能

1. 阅读现有代码，理解实现逻辑
2. 确定修改点和影响范围
3. 使用 Edit 工具进行修改
4. 保持代码风格一致

#### 新增功能

新增命令流程：
1. 在 `package.json` contributes.commands 添加命令定义
2. 在 `src/extension.ts` 注册命令处理函数
3. 实现具体业务逻辑

新增视图流程：
1. 在 `package.json` contributes.views 添加视图定义
2. 创建 TreeDataProvider 类（参考现有 Provider）
3. 在 `src/extension.ts` 注册视图

新增配置项流程：
1. 在 `package.json` contributes.configuration 添加配置
2. 在代码中使用 `vscode.workspace.getConfiguration('agentSkills')` 读取

#### 代码规范

- 使用 TypeScript 类型注解
- 遵循现有命名规范（驼峰命名）
- 错误处理使用 try-catch 并显示友好提示
- 异步操作使用 async/await

### 第四步：类型检查与 Lint

```bash
npm run check-types  # TypeScript 类型检查
npm run lint         # ESLint 检查
```

修复所有类型错误和 lint 警告。

### 第五步：编译与测试

```bash
npm run compile      # 编译项目
npm test             # 运行测试
```

#### 手动测试

1. 按 F5 启动 Extension Development Host
2. 验证功能是否正常工作
3. 检查是否有控制台错误

### 第六步：验证完成

- [ ] 类型检查通过
- [ ] Lint 检查通过
- [ ] 编译成功
- [ ] 功能测试通过

## 参考文档

- **项目架构详情**: 见 [references/architecture.md](references/architecture.md)
- **测试指南**: 见 [references/testing.md](references/testing.md)
- **命令/视图开发详解**: 见 [references/commands.md](references/commands.md)

## 常见任务示例

### 添加新命令

```typescript
// 1. package.json
"contributes": {
  "commands": [{
    "command": "agentSkills.newCommand",
    "title": "New Command",
    "icon": "$(icon-name)"
  }]
}

// 2. extension.ts
const newCmd = vscode.commands.registerCommand(
  'agentSkills.newCommand',
  async () => {
    // 实现逻辑
  }
);
context.subscriptions.push(newCmd);
```

### 添加配置项

```json
// package.json
"contributes": {
  "configuration": {
    "properties": {
      "agentSkills.newSetting": {
        "type": "string",
        "default": "defaultValue",
        "description": "Setting description"
      }
    }
  }
}
```

```typescript
// 读取配置
const config = vscode.workspace.getConfiguration('agentSkills');
const value = config.get<string>('newSetting');
```

### 更新 TreeView 数据

```typescript
// 在 Provider 中调用
this._onDidChangeTreeData.fire();
```
