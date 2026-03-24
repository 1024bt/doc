### openclaw的token使用量
1.QMD模式, 用 memory_search 向量检索+memory_get精确读取，不用全量加载记忆文件。
2.精简记忆,设置定时任务，每天自动沉淀日志，定期合成到主记忆文件。保持MEMORY.md精简

#### 安装qmd和sqlite

#### 第一步：安装QMD

在Windows命令提示符（cmd）或PowerShell中（**建议以管理员身份运行**），使用npm进行全局安装:

```bash
npm install -g @tobilu/qmd
```

安装完成后，验证QMD是否成功：

```bash
qmd --version
```

如果显示版本号，说明QMD安装成功 。

#### 如果提示找不到路径,说明qmd.cmd脚本是linux系统的,需要手动修复

将qmd.cmd中的内容改为:

```shell
@echo off
node "C:\Users\用户名\AppData\Roaming\npm\node_modules\@tobilu\qmd\dist\cli\qmd.js" %*
```



#### 第二步：安装支持向量扩展的SQLite（关键步骤！）

这是Windows原生配置和macOS/Linux最大的不同点。QMD需要一个**支持向量扩展的SQLite**，Windows系统自带的版本不满足要求，需要手动下载配置 。

1. **下载SQLite工具**：

   - 访问SQLite官方下载页面：https://www.sqlite.org/download.html
   - 在"Precompiled Binaries for Windows"区域，下载 **`sqlite-tools-win-x64-\*.zip`** 文件（`*`是版本号，下载最新的即可）。

2. **解压文件**：

   - 将下载的zip包解压到一个你喜欢的、**没有空格和中文的目录**，例如：`C:\sqlite` 或 `D:\Tools\sqlite`。
   - 解压后，你应该能看到 `sqlite3.exe` 这个文件。

3. **添加到系统PATH（环境变量）** ：

   - 右键点击“此电脑”或“我的电脑”，选择“属性”。
   - 点击“高级系统设置”。
   - 点击“环境变量”。
   - 在“系统变量”或“用户变量”中找到 `Path` 变量，选中并点击“编辑”。
   - 点击“新建”，然后输入你刚才解压SQLite的文件夹路径，例如 `C:\sqlite`。
   - 点击“确定”保存所有更改。

4. **验证SQLite安装** ：

   - **关闭并重新打开**你的命令提示符或PowerShell（这一步很重要，要让系统重新加载环境变量）。

   - 输入以下命令并回车：

     ```bash
     sqlite3 --version
     ```



#### macOS系统自带的SQLite版本较旧且功能受限，必须替换。

```shell
# 使用 Homebrew 安装 SQLite
brew install sqlite

# 安装后，按照提示将其链接到环境变量，确保系统优先使用这个新版本
# 通常需要将 '/opt/homebrew/opt/sqlite/bin' 添加到 PATH 的最前面
echo 'export PATH="/opt/homebrew/opt/sqlite/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# 验证当前使用的 SQLite 是否为 Homebrew 版本
which sqlite3
# 应该输出类似 /opt/homebrew/opt/sqlite/bin/sqlite3 的路径
```

#### 配置

```shell
先安装qmd和sqlite，并且配置了在openclaw.json中增加了配置
"memory": {
// 1. 将后端切换为 QMD，这是最关键的一步
"backend": "qmd",
// 2. 控制在回复中是否显示记忆来源
"citations": "auto",
// 3. QMD 专属配置，用于精确控制token使用量
"qmd": {
// 自动索引默认的 MEMORY.md 和 memory/*.md 文件
"includeDefaultMemory": true,
// 4. 检索限制：这是你节省 token 的关键
"limits": {
"maxResults": 5, // 每次搜索最多返回5个片段
"maxSnippetChars": 1500, // 每个片段最多约375 token
"maxInjectedChars": 4500, // 所有片段总计最多约1125 token
"timeoutMs": 8000 // 搜索超时8秒，防止卡死 [citation:1]
},
// 索引更新策略
"update": {
"interval": "10m", // 每10分钟检查并更新一次索引
"debounceMs": 15000, // 文件变更后等待15秒再触发更新
"onBoot": true, // OpenClaw启动时更新索引
"waitForBootSync": false // 后台异步更新，不阻塞启动
}
}
}

在agent.default下增加了
// 1. 自动记忆刷新：在上下文压缩前，提醒Agent写入重要信息
"compaction": {
"reserveTokensFloor": 20000,
"memoryFlush": {
"enabled": true,
"softThresholdTokens": 6000,
"systemPrompt": "会话即将进行上下文压缩。请将任何需要持久化的决策、用户偏好或关键信息，写入到 MEMORY.md 或今天的日志文件中。",
"prompt": "请将需要长久记住的信息写入记忆文件。如果没有什么需要特别记录的，请回复 NO_REPLY。"
}
},
// 2. 心跳机制：让Agent在后台定期“思考”和“整理”
"heartbeat": {
"every": "1h", // 每30分钟唤醒一次Agent
// 提醒Agent在心跳时检查并执行记忆整理任务
"prompt": "现在是后台维护时间。请检查是否需要执行以下任务：\n1. 回顾今天的日志，将重要的经验、教训或决策提炼并追加到 MEMORY.md 中。\n2. 检查 MEMORY.md 是否有冗余或过时的信息可以清理或合并。\n如果执行了任何写入操作，请确保内容精炼。没有任务需要执行，则回复 NO_REPLY。"
}
```



#### 启动验证

网关输出[gateway] qmd memory startup initialization armed for agent "main" 成功

网关输出[memory] qmd collection add failed for memory-alt-main: spawn EINVAL 失败
需要升级openclaw版本, 老版本无法执行手动修复的qmd.cmd文件

