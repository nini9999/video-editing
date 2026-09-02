# CLAUDE.md — OpenChatCut 视频剪辑 AI 工作流

本仓库用于通过 **Claude Code + OpenChatCut**(开源 ChatCut 替代品)做「对话式 / 文字稿驱动」的视频剪辑。

OpenChatCut 是一个多轨时间线编辑器,所有 AI 改动都写入**真实可编辑的轨道**(而不是一次性生成),
支持文字稿逐词剪辑、字幕、转场、特效、运动图形与导出(MP4 / 音频 / 字幕 / FCPXML)。

## 连接方式(本机运行)

- Claude Code 通过项目根目录的 `.mcp.json` 连接 OpenChatCut 的本地 MCP 端点:
  `http://localhost:5199/api/external-mcp/mcp`(Streamable HTTP)。
- **该端点在 localhost 上也强制要 bearer token**。token 在 OpenChatCut app 的
  **Settings → MCP** 页面显示;`.mcp.json` 通过环境变量 `${OPENCHATCUT_MCP_TOKEN}`
  读取它,启动 Claude Code 前需 `export OPENCHATCUT_MCP_TOKEN='<token>'`。
  服务器重启会重新生成 token;想固定可在 OpenChatCut 的 `.env.local` 里设
  `OPENCHATCUT_MCP_TOKEN`。**切勿把 token 写进仓库。**
- 使用剪辑工具**前提**:OpenChatCut app 必须已在本机运行,并已打开目标项目。
- 已安装的 OpenChatCut Agent Skill 是一个 router,按需动态加载 26 个专用剪辑子技能,
  避免一次性把所有工具塞进上下文。

> 注意:云端 / 远程 Claude Code 会话访问不到你本机的 `localhost:5199`。
> 这套 MCP 连接需要在**本机**运行的 Claude Code 里使用。

## 标准编辑会话流程

对 OpenChatCut 做任何改动,都遵循「会话 → 编辑 → 送审」的模式:

1. `begin_edit_session` → 返回 `editSessionId`
2. 把该 `editSessionId` 传给所有 project 读取 / 编辑类工具
3. 编辑完成后调用 `review_edit_session` 提交送审
4. 用户在打开的 OpenChatCut 项目里批准 / 拒绝(手动模式),或自动应用(自动模式)

## 请求示例

> 「开一个 OpenChatCut 编辑会话。读取草稿,在第二条音轨的第 8 秒加一个 scratch 音效,
> 在相邻视频片段之间加一个 glitch 转场,然后提交送审。」

## 可选:对外暴露端点时的鉴权

仅当把端点暴露到公网时,在 OpenChatCut 的 `.env.local` 配置:

```bash
OPENCHATCUT_MCP_TOKEN=your-token
OPENCHATCUT_EDITOR_URL=https://your-editor.example.com
```

此时客户端需发送 `Authorization: Bearer <token>`,并相应地在 `.mcp.json` 的该 server 下
加一个 `headers` 字段(见 `README.md`)。单机单用户的本地场景无需鉴权。
