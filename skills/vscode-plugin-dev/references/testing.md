# 测试指南

## 测试框架

项目使用 **Mocha** + **@vscode/test-cli** + **@vscode/test-electron** 进行测试。

## 测试文件位置

```
src/test/
└── extension.test.ts    # 测试文件
```

编译后输出到 `out/test/*.test.js`

## 运行测试

```bash
# 运行所有测试
npm test

# 仅编译测试
npm run compile-tests

# 完整预测试流程（编译 + lint + 测试编译）
npm run pretest
```

## 测试配置

测试配置文件: `.vscode-test.mjs`

```javascript
import { defineConfig } from '@vscode/test-cli';

export default defineConfig({
  files: 'out/test/**/*.test.js',
});
```

## 编写测试

### 基本测试结构

```typescript
import * as assert from 'assert';
import * as vscode from 'vscode';

suite('Extension Test Suite', () => {
  vscode.window.showInformationMessage('Start all tests.');

  test('Sample test', () => {
    assert.strictEqual(-1, [1, 2, 3].indexOf(5));
    assert.strictEqual(-1, [1, 2, 3].indexOf(0));
  });

  test('Extension should be present', () => {
    assert.ok(vscode.extensions.getExtension('publisher.extension-id'));
  });

  test('Command should be registered', async () => {
    const commands = await vscode.commands.getCommands();
    assert.ok(commands.includes('agentSkills.install'));
  });
});
```

### 异步测试

```typescript
test('Async operation', async () => {
  const result = await someAsyncFunction();
  assert.strictEqual(result, expectedValue);
});
```

### 测试 VSCode API

```typescript
test('Show information message', async () => {
  const message = await vscode.window.showInformationMessage(
    'Test message',
    'OK'
  );
  // 用户点击后返回 'OK'
});

test('Read configuration', () => {
  const config = vscode.workspace.getConfiguration('agentSkills');
  const value = config.get('installLocation');
  assert.ok(value);
});
```

### Mock 和 Stub

由于 VSCode 测试在真实环境中运行，通常不需要 mock VSCode API。但对于外部服务（如 GitHub API），建议：

```typescript
// 使用接口依赖注入
interface ISkillsClient {
  getSkills(): Promise<Skill[]>;
}

// 测试时可以注入 mock 实现
class MockSkillsClient implements ISkillsClient {
  async getSkills(): Promise<Skill[]> {
    return [{ name: 'test-skill', description: 'Test', path: '/', repository: {...} }];
  }
}
```

## 手动测试

### 使用 Extension Development Host

1. 在 VSCode 中打开项目
2. 按 **F5** 启动调试
3. 在新窗口中测试插件功能

### 调试配置 (.vscode/launch.json)

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "name": "Run Extension",
      "type": "extensionHost",
      "request": "launch",
      "args": [
        "--extensionDevelopmentPath=${workspaceFolder}"
      ],
      "outFiles": ["${workspaceFolder}/dist/**/*.js"]
    },
    {
      "name": "Extension Tests",
      "type": "extensionHost",
      "request": "launch",
      "args": [
        "--extensionDevelopmentPath=${workspaceFolder}",
        "--extensionTestsPath=${workspaceFolder}/out/test"
      ],
      "outFiles": ["${workspaceFolder}/out/**/*.js"]
    }
  ]
}
```

## 测试检查清单

### 功能测试
- [ ] 命令是否正确注册
- [ ] 命令执行是否正常
- [ ] UI 更新是否正确
- [ ] 错误处理是否友好

### 边界情况
- [ ] 空数据处理
- [ ] 网络错误处理
- [ ] 配置缺失处理
- [ ] 权限不足处理

### 集成测试
- [ ] 视图刷新是否正常
- [ ] 数据流是否正确
- [ ] 状态同步是否一致

## 调试技巧

### 控制台输出
```typescript
console.log('Debug info:', variable);
// 在 Extension Development Host 的 Debug Console 中查看
```

### 断点调试
1. 在源代码中设置断点
2. 按 F5 启动调试
3. 触发相关功能
4. 在断点处检查变量

### 查看输出
- **Debug Console**: 查看 console.log 输出
- **Output Panel**: 查看插件输出通道
- **Developer Tools**: Help > Toggle Developer Tools
