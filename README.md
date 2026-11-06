# 往周种梨 1.16.1

本地保存会话的桌面编程助手：聊天、Agent、技能、MCP、文件编辑与网页/Markdown/PDF预览。

## 安装使用
1. 在 [GitHub Releases](https://github.com/RuiaaZ/wangzhouzhongli/releases) 下载实际发布的 macOS 安装包，拖入 Applications。
2. 打开软件，在设置添加在线模型服务商、API Key 和模型，执行连接测试。
3. 选择工作文件夹，新建对话/任务，输入目标；Agent会显示真实工具调用，完成后展示可打开的交付文件。

本版目标为macOS Apple Silicon。仅本地签名，未Apple公证。包含Windows构建脚本，本次未在Windows实际构建验证。发布结果以实际Release为准。

## 本机源码验收
公开仓库按用户要求仅分发安装包及README；以下命令需在持有源码的工作区运行，不能从分发仓库克隆得到源码。用户数据、密钥、源码和JSON报告均不随公开安装包仓库上传。

```bash
uv sync --extra test --extra build
npm ci
.venv/bin/python -m agent.selftest --full
.venv/bin/python -m agent.inspect skills --out skill_report.json
.venv/bin/python -m agent.inspect custom --out build/custom-mock.json
.venv/bin/python -m agent.inspect mcp --out build/mcp-live.json
```

custom默认使用明确标记的模拟模型与真实工具，不算真实模型自主恢复通过，故总ok=false并返回非零。真实验收：设置OPENAI_API_KEY、OPENAI_MODEL、OPENAI_BASE_URL后加 `--live`；或设置WANGZHOU_INSPECT_PROFILE为本机模型配置ID。不在日志或命令行写出密钥。真实请求可能消耗模型额度，只发送独立临时项目的验收数据。没有配置时明确未验证，不造usage。

```bash
.venv/bin/python -m agent.inspect custom --live --out custom_report.json
ENABLE_SKILLS=0 .venv/bin/python -m agent '你好' --model MODEL
ENABLE_HOOKS=0 .venv/bin/python -m agent.inspect custom --out build/hooks-off.json
LOCAL_CURSOR_TEST_SOURCE=1 .venv/bin/python scripts/test-bundle.py
LOCAL_CURSOR_RUNTIME='/Applications/往周种梨.app/Contents/Resources/runtime/localcursor-runtime' .venv/bin/python scripts/test-workspace-flow.py
```

注意：custom本身会为前置检查/开关对比显式设置开关并恢复环境；检验全局关闭请运行agent任务或test_customization中的同代码对照。不要在真实项目关闭安全hook做危险实验。

## 行为、技能和测试配置
全局行为文件默认 `~/AGENTS.md`，可用WANGZHOU_GLOBAL_AGENTS指定；项目文件按祖先到工作目录顺序追加，含来源和层级，总注入不超过12000 UTF8字节。不会重写行为文件。目录只提供id/name/description；正文需要load_skill，脚本和资源按需读取。斜杠选择也先记录选择，再真实调用加载工具。

项目 `.wangzhou/hooks.json` 示例：
```json
{"test_command":"python3 -B -m unittest discover -s tests -q","timeout":30}
```
没有配置且有tests目录时使用运行时内置unittest发现；否则标记skipped。命令超时最多120秒，stdout/stderr最多32000字符，原始受限输出保存进会话审计，模型只得到三行失败摘要。写/edit成功后才触发，失败或拒绝不会触发。测试自身通过直接子进程执行，不递归调用hook。模型调用delegate_reviewer才会委派独立只读上下文，read和严格git diff白名单以外的工具均拒绝。

密钥写入拦截包含.env及.env.*、*.pem、*.key；明确允许.env.example/.env.sample/.env.template。命令检查包含常见POSIX递归删除/强制push/硬reset及Windows递归删除形式；它不是系统沙箱，任意解释器、编码、嵌套命令或第三方MCP仍可能绕过模式检查。完全访问权限不跳过已启用的hook。ENABLE_SKILLS、ENABLE_HOOKS、ENABLE_REVIEWER支持0关闭；关闭hook会取消课程硬约束，仅供独立验收项目比较。

## 运行模式
日常daily默认不设固定步数或整体时限，模型收尾、用户停止、上下文上限、请求/工具超时、明显重复操作会终止。验收acceptance默认60轮、300秒、累计100000 token；可配置max_steps/task_timeout/token_budget，实际run_config进入记录。累计预算优先服务商usage，缺失时使用cl100k_base参考计数并标明来源，不称为计费token。

```bash
.venv/bin/python -m agent '运行项目测试并报告' --workspace . --allow-write --run-profile acceptance --max-steps 40 --token-budget 50000 --timeout 120
WANGZHOU_RUN_PROFILE=acceptance .venv/bin/python app.py
```

桌面从环境变量WANGZHOU_RUN_PROFILE选择验收模式；WANGZHOU_TASK_TIMEOUT、WANGZHOU_TOKEN_BUDGET、LOCAL_CURSOR_MAX_STEPS分别配置验收时限、预算及轮数。日常忽略旧配置中的固定max_steps，只有显式enforce_steps或验收模式才执行步数上限。模型连接、命令和MCP仍各自限时。完整清单见本机docs/acceptance-checklist.md，统计口径与未完成材料见report.md。
