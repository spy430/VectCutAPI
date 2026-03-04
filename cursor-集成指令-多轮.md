# Cursor 多轮集成指令：把 VectCutAPI 内置到自己的项目并修复问题

按顺序在 Cursor 里**分轮**发送下面每一段（每轮只发一段，等 Cursor 做完再发下一段）。

---

## 第一轮：打开自己的项目并说明要内置 VectCutAPI

**复制下面整段发给 Cursor：**

```
请先确认当前工作区是我自己的项目（Electron + Vue 短剧制作工具），不是 VectCutAPI 的目录。

我要把开源项目 VectCutAPI 内置到当前项目里，用来把应用里带时间轨的图片、视频、音频导出成剪映草稿。

VectCutAPI 的 GitHub 地址：
https://github.com/sun-guannan/VectCutAPI

请帮我：
1. 克隆或拉取该仓库到当前项目下的一个子目录（例如 vectcut-api 或 lib/vectcut-api），不要覆盖现有代码。
2. 在项目里接入/调用方式说明：例如 Electron 主进程如何启动 VectCutAPI 的 HTTP 服务（python capcut_server.py）、前端如何调 create_draft / add_video / add_image / add_audio / save_draft 等 API，以及导出后的草稿文件夹要复制到剪映草稿目录才能打开。
3. 如需 config，请基于 VectCutAPI 里的 config.json.example 在当前项目里生成一份可用的 config.json（端口等可按我们项目约定调整）。

依赖说明：VectCutAPI 需要 Python 3.10+、FFmpeg；若我们项目是 Electron 打包，请说明如何把 Python 环境或可执行文件一起打包（或文档里写清用户需预装 Python 和 FFmpeg）。
```

---

## 第二轮：提出需求，让 Cursor 修复「导出为空」和「端口占用」

**等第一轮集成完成后，再发下面这段：**

```
VectCutAPI 已经内置到项目里了。现在有两个问题需要你在我当前项目里修复（在已内置的 VectCutAPI 代码上改，不要改回原 GitHub 的未修改版）：

**问题 1：导出的草稿在剪映里打开是空的**  
- 原因：保存草稿时，draft_info.json 里视频/图片/音频的 path 没有正确写成「相对草稿目录」的路径，剪映找不到媒体文件。
- 请修复：在 save_draft_impl.py 里，在调用 script.dump() 之前，为所有素材（script.materials.audios 和 script.materials.videos）设置 replace_path 为相对草稿目录的路径，例如：
  - 音频：assets/audio/<material_name>
  - 图片：assets/image/<material_name>
  - 视频：assets/video/<material_name>
  路径在 JSON 里请用正斜杠（/），兼容 Windows。这样 draft_info.json 里会有正确的 path，剪映打开草稿才能看到内容。用户需要把整个 dfd_xxx 文件夹（含 draft_info.json 和 assets）复制到剪映草稿目录。

**问题 2：导出时经常提示 9001 端口被占用**  
- 原因：多次启动 capcut_server 时，旧进程没退出，导致端口占用。
- 请修复：在 capcut_server.py 的 if __name__ == '__main__' 里，启动前检测当前配置端口是否可用；若被占用则自动尝试下一个端口（如 9002、9003），并打印实际监听的端口，方便前端或文档使用。同时建议在 README 或项目文档里写一句：内置时建议只启动一个 VectCutAPI 服务进程，多次导出复用同一进程，避免重复起进程导致端口占用。
```

---

## 第三轮（可选）：补充说明或检查

如果第二轮后还有问题，可以发：

```
请检查：1) 剪映草稿是否必须复制「整个 dfd_xxx 文件夹」到剪映草稿目录；2) 我们 Electron 里是否做到了「只起一个 VectCutAPI 服务、多次导出只调 API」；3) 若端口自动换了，前端请求的 baseURL 是否需要从某处（如启动日志或健康检查接口）读取当前端口。
```

---

## 使用提醒

- 每轮只发**一段**，等 Cursor 执行完再发下一段。
- 第一轮前请先在 Cursor 里**打开你自己的项目目录**作为工作区，不要打开 VectCutAPI 目录。
- 这样 Cursor 会基于**原版 GitHub 的 VectCutAPI** 做内置，再在你的项目里直接改 bug，不依赖我之前改过的那份本地 VectCutAPI。
