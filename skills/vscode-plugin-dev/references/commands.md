# 命令与视图开发详解

## 命令开发

### 完整命令添加流程

#### 1. 在 package.json 定义命令

```json
{
  "contributes": {
    "commands": [
      {
        "command": "agentSkills.myNewCommand",
        "title": "My New Command",
        "icon": "$(add)",
        "category": "Agent Skills"
      }
    ]
  }
}
```

**常用图标**:
- `$(add)` - 添加
- `$(remove)` - 删除
- `$(refresh)` - 刷新
- `$(search)` - 搜索
- `$(folder)` - 文件夹
- `$(file)` - 文件
- `$(gear)` - 设置
- `$(eye)` - 查看

#### 2. 注册命令处理函数

```typescript
// src/extension.ts
export function activate(context: vscode.ExtensionContext) {
  const myCommand = vscode.commands.registerCommand(
    'agentSkills.myNewCommand',
    async (arg?: any) => {
      try {
        // 实现逻辑
        vscode.window.showInformationMessage('Command executed!');
      } catch (error) {
        vscode.window.showErrorMessage(`Error: ${error}`);
      }
    }
  );
  context.subscriptions.push(myCommand);
}
```

#### 3. 添加到菜单/工具栏（可选）

```json
{
  "contributes": {
    "menus": {
      "view/title": [
        {
          "command": "agentSkills.myNewCommand",
          "when": "view == agentSkills.marketplace",
          "group": "navigation"
        }
      ],
      "view/item/context": [
        {
          "command": "agentSkills.myNewCommand",
          "when": "viewItem == skill",
          "group": "inline"
        }
      ]
    }
  }
}
```

### 命令参数传递

```typescript
// 注册带参数的命令
vscode.commands.registerCommand('agentSkills.action', (skill: Skill) => {
  console.log(skill.name);
});

// 在 TreeItem 中设置命令参数
class SkillItem extends vscode.TreeItem {
  constructor(skill: Skill) {
    super(skill.name);
    this.command = {
      command: 'agentSkills.action',
      title: 'Action',
      arguments: [skill]
    };
  }
}
```

### 条件显示 (when 子句)

常用条件:
- `view == viewId` - 当前视图
- `viewItem == contextValue` - TreeItem 的 contextValue
- `config.setting == value` - 配置值
- `resourceScheme == file` - 资源类型

组合条件:
- `&&` - 与
- `||` - 或
- `!` - 非

```json
"when": "view == agentSkills.installed && viewItem == installedSkill"
```

## 视图开发

### TreeView 数据提供者

```typescript
import * as vscode from 'vscode';

export class MyTreeProvider implements vscode.TreeDataProvider<MyItem> {
  // 用于触发刷新
  private _onDidChangeTreeData = new vscode.EventEmitter<MyItem | undefined>();
  readonly onDidChangeTreeData = this._onDidChangeTreeData.event;

  // 获取树节点元素
  getTreeItem(element: MyItem): vscode.TreeItem {
    return element;
  }

  // 获取子节点
  async getChildren(element?: MyItem): Promise<MyItem[]> {
    if (!element) {
      // 返回根节点
      return this.getRootItems();
    }
    // 返回子节点
    return element.children || [];
  }

  // 刷新视图
  refresh(): void {
    this._onDidChangeTreeData.fire(undefined);
  }

  private async getRootItems(): Promise<MyItem[]> {
    // 获取数据逻辑
    return [];
  }
}
```

### TreeItem 定义

```typescript
export class MyItem extends vscode.TreeItem {
  constructor(
    public readonly label: string,
    public readonly collapsibleState: vscode.TreeItemCollapsibleState,
    public readonly data: any
  ) {
    super(label, collapsibleState);

    // 设置描述（显示在标签右侧）
    this.description = data.version;

    // 设置工具提示
    this.tooltip = data.fullDescription;

    // 设置图标
    this.iconPath = new vscode.ThemeIcon('package');

    // 设置上下文值（用于 when 子句）
    this.contextValue = 'myItemType';

    // 点击时执行的命令
    this.command = {
      command: 'agentSkills.viewDetails',
      title: 'View Details',
      arguments: [data]
    };
  }
}
```

### 注册视图

```typescript
// 方式1: 使用 registerTreeDataProvider
vscode.window.registerTreeDataProvider('myView', new MyTreeProvider());

// 方式2: 使用 createTreeView（更多控制选项）
const treeView = vscode.window.createTreeView('myView', {
  treeDataProvider: new MyTreeProvider(),
  showCollapseAll: true,
  canSelectMany: false
});
context.subscriptions.push(treeView);
```

### package.json 视图配置

```json
{
  "contributes": {
    "viewsContainers": {
      "activitybar": [
        {
          "id": "myContainer",
          "title": "My Extension",
          "icon": "resources/icon.svg"
        }
      ]
    },
    "views": {
      "myContainer": [
        {
          "id": "myView",
          "name": "My View",
          "icon": "resources/view-icon.svg",
          "contextualTitle": "My Extension"
        }
      ]
    },
    "viewsWelcome": [
      {
        "view": "myView",
        "contents": "No items found.\n[Add Item](command:myExtension.addItem)"
      }
    ]
  }
}
```

## WebView 面板

### 创建 WebView

```typescript
export class DetailPanel {
  public static currentPanel: DetailPanel | undefined;
  private readonly _panel: vscode.WebviewPanel;
  private _disposables: vscode.Disposable[] = [];

  public static createOrShow(extensionUri: vscode.Uri, data: any) {
    const column = vscode.window.activeTextEditor?.viewColumn;

    if (DetailPanel.currentPanel) {
      DetailPanel.currentPanel._panel.reveal(column);
      DetailPanel.currentPanel._update(data);
      return;
    }

    const panel = vscode.window.createWebviewPanel(
      'detailPanel',
      'Detail',
      column || vscode.ViewColumn.One,
      {
        enableScripts: true,
        localResourceRoots: [vscode.Uri.joinPath(extensionUri, 'resources')]
      }
    );

    DetailPanel.currentPanel = new DetailPanel(panel, extensionUri, data);
  }

  private constructor(
    panel: vscode.WebviewPanel,
    extensionUri: vscode.Uri,
    data: any
  ) {
    this._panel = panel;
    this._update(data);

    // 监听面板关闭
    this._panel.onDidDispose(() => this.dispose(), null, this._disposables);

    // 监听来自 WebView 的消息
    this._panel.webview.onDidReceiveMessage(
      message => {
        switch (message.command) {
          case 'action':
            vscode.window.showInformationMessage(message.text);
            return;
        }
      },
      null,
      this._disposables
    );
  }

  private _update(data: any) {
    this._panel.webview.html = this._getHtmlContent(data);
  }

  private _getHtmlContent(data: any): string {
    return `<!DOCTYPE html>
      <html>
      <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>Detail</title>
        <style>
          body { padding: 20px; font-family: var(--vscode-font-family); }
        </style>
      </head>
      <body>
        <h1>${data.title}</h1>
        <p>${data.description}</p>
        <button onclick="action()">Action</button>
        <script>
          const vscode = acquireVsCodeApi();
          function action() {
            vscode.postMessage({ command: 'action', text: 'Button clicked!' });
          }
        </script>
      </body>
      </html>`;
  }

  public dispose() {
    DetailPanel.currentPanel = undefined;
    this._panel.dispose();
    while (this._disposables.length) {
      const x = this._disposables.pop();
      if (x) x.dispose();
    }
  }
}
```

## 配置项开发

### 定义配置

```json
{
  "contributes": {
    "configuration": {
      "title": "Agent Skills",
      "properties": {
        "agentSkills.setting1": {
          "type": "string",
          "default": "default",
          "description": "Description",
          "enum": ["option1", "option2"],
          "enumDescriptions": ["Option 1 desc", "Option 2 desc"]
        },
        "agentSkills.setting2": {
          "type": "boolean",
          "default": true,
          "description": "Enable feature"
        },
        "agentSkills.setting3": {
          "type": "array",
          "items": { "type": "string" },
          "default": [],
          "description": "List of items"
        }
      }
    }
  }
}
```

### 读取配置

```typescript
const config = vscode.workspace.getConfiguration('agentSkills');

// 读取值
const value = config.get<string>('setting1');
const enabled = config.get<boolean>('setting2', true); // 带默认值

// 更新值
await config.update('setting1', 'newValue', vscode.ConfigurationTarget.Global);

// 监听配置变化
vscode.workspace.onDidChangeConfiguration(e => {
  if (e.affectsConfiguration('agentSkills.setting1')) {
    // 配置已更改，重新加载
  }
});
```

## 文件系统监听

```typescript
// 监听文件变化
const watcher = vscode.workspace.createFileSystemWatcher(
  '**/skills/**/*.md',
  false, // 创建
  false, // 修改
  false  // 删除
);

watcher.onDidCreate(uri => console.log('Created:', uri.fsPath));
watcher.onDidChange(uri => console.log('Changed:', uri.fsPath));
watcher.onDidDelete(uri => console.log('Deleted:', uri.fsPath));

context.subscriptions.push(watcher);
```
