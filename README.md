# pi-agenote — oh-my-pi (omp) agenote 集成扩展

> oh-my-pi (omp) 的 agenote 集成钩子扩展。在 omp 会话中注入 agenote 健康度摘要、
> 检测任务完成信号触发经验采集、提供 `/agenote-*` 斜杠命令快捷入口。

本仓库是 [agenote](https://github.com/ShineBreaker/agenote) 跨 Agent 经验平台在
omp（oh-my-pi）侧的集成扩展，作为 `Guix-configs` 的 Git 子模块嵌入。

## 扩展功能

| 钩子/命令 | 触发 | 作用 |
| --------- | ---- | ---- |
| `session_start` | 会话启动 | 注入 agenote 健康度摘要（卡片数、陈旧率、薄弱类别） |
| `agent_end` | agent 响应结束 | 检测"任务完成信号"，命中时注入 agenote-review 评估提示（含留痕） |
| `/agenote-summarize` | 斜杠命令 | 在当前会话触发经验总结 + 资料留痕 |
| `/agenote-curate` | 斜杠命令 | 执行 agenote 策展（健康+去重+归档+权重重分配） |
| `/agenote-health` | 斜杠命令 | 显示 agenote 健康度报告 |

信号清单、写入流程、卡片格式由 [agenote-skills](https://github.com/ShineBreaker/agenote-skills)
的 `agenote-{base,curator,review}` skill 提供，本插件只做"事件触发 + 命令快捷入口"，
避免与 skill 重复维护。

## OMP 扩展规范

omp 扩展是**单文件 TypeScript**，无需构建步骤：

- omp 运行时**自动扫描** `~/.config/omp/extensions/*/index.ts`
- 内置 TypeScript 运行时，直接加载 `.ts` 源码（类似 tsx/bun）
- `config.yml` 中**无需显式注册**扩展
- `ExtensionAPI` 类型由 omp 运行时作为**全局类型**注入，无需 import
- 所有 import 都是 Node.js 内置模块（`node:child_process` 等），无第三方 npm 依赖

入口签名：
```typescript
export default function init(pi: ExtensionAPI): void { ... }
```

## 部署

### 作为 Guix-configs 子模块（主流程）

本仓库登记为 `Guix-configs` 的 `dotfiles/mutable/agenote/.config/omp/extensions/agenote-hooks`
子模块，由 `blue stow` 统一纳管：

```bash
git submodule update --init dotfiles/mutable/agenote/.config/omp/extensions/agenote-hooks
blue stow agenote           # 部署软链
blue stow --restow agenote  # 重建
```

部署后：
```
~/.config/omp/extensions/agenote-hooks/index.ts → 本仓库源（逐文件软链）
```

### 独立部署（原生 stow）

```bash
git clone https://github.com/ShineBreaker/pi-agenote.git ~/pi-agenote
stow --dir=~/pi-agenote --target=$HOME
```

## 依赖

- **[agenote](https://github.com/ShineBreaker/agenote) CLI**：本插件通过调用 agenote 的
  轻量 shim（`~/.local/bin/agenote-cli`）执行 health/curate，必须先安装 agenote CLI。
- **omp（oh-my-pi）**：扩展宿主。

## 改源生效路径

omp 直接加载 `.ts` 源码，**改源即生效**（下次 omp 会话启动时重新扫描）。

## 许可证

MIT，见 [LICENSE](LICENSE)。
