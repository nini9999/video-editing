# video-editing — Claude Code × OpenChatCut 工作流

用 **Claude Code** 驱动 **[OpenChatCut](https://github.com/0xsline/OpenChatCut)**(开源 ChatCut 替代品)
做对话式 / 文字稿驱动的视频剪辑。本仓库已经配置好接入所需的一切;剩下的只是在本机装好并启动 OpenChatCut。

## 前置要求

- **Node.js 24.x**(OpenChatCut 用 `.nvmrc` 强制)
- **FFmpeg**(导出用)
- 可选:AI 模型 API key(OpenAI / Anthropic / Gemini 等)
- 本机安装的 **Claude Code**(远程/云端会话访问不到本机 `localhost:5199`)

## 一次性安装

### 1. 装并启动 OpenChatCut

```bash
git clone https://github.com/0xsline/OpenChatCut.git
cd OpenChatCut
npm install
cp .env.example .env.local
npm run dev          # 打开 http://localhost:5199
```

或直接从 [Releases](https://github.com/0xsline/OpenChatCut/releases) 下载桌面版(macOS / Windows / Linux)。

### 2. 把 OpenChatCut 接进 Claude Code

本仓库根目录的 [`.mcp.json`](./.mcp.json) 已包含连接配置,在本机用 Claude Code 打开本仓库即会加载。
**该端点在 localhost 上也必须带 bearer token**:token 在 OpenChatCut app 的 **Settings → MCP**
页面显示,`.mcp.json` 通过 `${OPENCHATCUT_MCP_TOKEN}` 读取,所以启动前先 export:

```bash
export OPENCHATCUT_MCP_TOKEN='<从 Settings → MCP 复制的 token>'
```

若想全局注册(任意目录可用),运行:

```bash
claude mcp add --transport http \
  -H "Authorization: Bearer $OPENCHATCUT_MCP_TOKEN" \
  openchatcut http://localhost:5199/api/external-mcp/mcp
```

### 3. 安装 OpenChatCut Agent Skill

```bash
npx skills add 0xsline/OpenChatCut
```

然后对 Claude Code 说 **「Set up OpenChatCut」/「设置 OpenChatCut」**。
这个 skill 是一个 router,按需动态加载 26 个专用剪辑子技能。

## 日常使用

1. 启动 OpenChatCut 并**打开目标项目**(时间线工具依赖当前打开的项目)。
2. 在本机 Claude Code 里,用自然语言描述剪辑任务,例如:

   > 「开一个 OpenChatCut 编辑会话。读取草稿,在第二条音轨的第 8 秒加一个 scratch 音效,
   > 在相邻视频片段之间加 glitch 转场,然后提交送审。」

3. Agent 按 `begin_edit_session → 编辑 → review_edit_session` 的流程操作,
   你在 OpenChatCut 里批准 / 拒绝(手动模式)或自动应用(自动模式)。详见 [`CLAUDE.md`](./CLAUDE.md)。

## 核心能力(OpenChatCut)

- 多轨时间线:关键帧、转场、特效、LUT
- **文字稿驱动剪辑**:逐词删改、自动字幕
- AI agents(内置 + 外部 MCP 客户端,如 Claude Code / Codex)
- 运动图形(可编辑模板 + WebGL shader)
- 媒体生成(图像 / 视频 / 语音 / 音乐 / 音效)
- 本地优先:项目留在本机(`~/.openchatcut`)
- 生产级导出:MP4 / 音频 / 字幕 / FCPXML / 工程数据

## 可选:对外暴露端点的鉴权

仅当把 MCP 端点暴露到公网时,在 OpenChatCut 的 `.env.local` 配置:

```bash
OPENCHATCUT_MCP_TOKEN=your-token
OPENCHATCUT_EDITOR_URL=https://your-editor.example.com
```

并把本仓库 `.mcp.json` 里的该 server 改成带鉴权头:

```json
{
  "mcpServers": {
    "openchatcut": {
      "type": "http",
      "url": "https://your-editor.example.com/api/external-mcp/mcp",
      "headers": { "Authorization": "Bearer ${OPENCHATCUT_MCP_TOKEN}" }
    }
  }
}
```

单机单用户本地场景无需鉴权。

## 参考

- OpenChatCut 仓库:https://github.com/0xsline/OpenChatCut
- MCP 端点:`http://localhost:5199/api/external-mcp/mcp`(Streamable HTTP)
- 许可:OpenChatCut 采用 AGPL-3.0-or-later
