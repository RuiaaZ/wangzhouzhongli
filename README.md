# 往周种梨

往周种梨（原 LocalCursor）是一款本地优先的 AI 编程桌面助手，支持日常聊天和 Agent 开发任务。

## 下载与安装

前往 [Releases 下载最新版](https://github.com/RuiaaZ/wangzhouzhongli/releases/latest)。

目前提供 **macOS Apple Silicon（M 系列芯片）** 安装包，Windows 版暂未发布。

1. 下载并打开 `.dmg`，将“往周种梨”拖入 Applications。
2. 打开应用，在左下角设置中添加模型接口、API Key 和模型。
3. 选择工作文件夹，新建聊天或 Agent 任务。

升级前请完全退出旧版应用。现有模型配置和会话会保留。当前 macOS 安装包使用本地签名，尚未经过 Apple 公证。

## 主要功能

- 聊天与 Agent 会话管理，支持历史续聊和删除。
- 多模型配置、连接测试和模型列表获取。
- 项目文件读取与修改、Diff 审核、命令执行和任务中断。
- 技能添加与删除、MCP 工具、内置浏览器和终端。
- 独立任务工作区、并行 Agent，以及明暗主题。

输入 `@` 引用项目文件，输入 `/` 选择技能。Agent 可修改文件和执行命令，请按需要选择操作权限。

## 说明

本仓库用于分发安装包，不包含软件源码。使用远程模型时，任务及所选上下文会发送至对应模型服务。安装包校验值见 Release 附件中的 `SHA256SUMS.txt`。

遇到问题可在 [Issues](https://github.com/RuiaaZ/wangzhouzhongli/issues) 反馈。
