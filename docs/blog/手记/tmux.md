---
title: tmux | 终端复用与持久会话
tags: 
  - tools
  - tmux
excerpt: tmux 是一个终端复用器，可以让你在一个终端中管理多个会话，实现持久化会话和终端复用。
createTime: 2026/09/22 19:22:18
permalink: /article/9y66j8zh/
---

之前在做远程调试时，经常需要 SSH 到设备上，同时开很多终端：有的启动程序，有的看日志，有的临时执行命令。慢慢就会发现，本地堆着一排 SSH 窗口并不是一件很舒服的事情。更麻烦的是，远程设备的网络未必一直稳定，SSH 一断，原本正在使用的终端也就跟着断掉了。

于是和 AI 讨论了我的烦恼后，它给我推荐了 tmux。

## tmux 是什么

tmux 是 **terminal multiplexer**，通常翻译成“终端复用器”。

官方对它的描述其实很直白：tmux 本身运行在一个终端中，同时允许多个终端程序运行在它里面；每个程序都有一个由 tmux 管理的独立终端，而我们可以通过外面的一个终端访问它们。

对我来说，它最直观的作用有两个：

- **终端复用**：不需要打开一堆 SSH 窗口，可以在一个 tmux 中管理多个终端；

- **会话保持**：退出 tmux 或 SSH 断开后，其中的终端和程序仍然可以继续存在，之后可以重新连接。

### Session、Window 和 Pane

tmux 使用三个层级来组织这些终端：

```text
Session
└── Window
    └── Pane
```

一个 **Pane** 是最直接看到的终端区域。

一个 **Window** 由一个或多个 Pane 构成，可以近似理解为普通终端软件中的一个 Tab。

多个 Window 又组成一个 **Session**。

例如：

```text
Session: dev
│
├── Window 0
│   ├── Pane 0
│   └── Pane 1
│
└── Window 1
    └── Pane 0
```

不过官方的模型其实比这个树形关系稍微灵活一点：Window 是被“链接”到 Session 中的，同一个 Window 甚至可以同时属于多个 Session；而一个 Session 可以同时 attach 到多个 client。只是日常使用时，把它理解成 `Session → Window → Pane` 基本就够了。

## tmux 是如何工作的

第一次执行：

```bash
tmux
```

时，如果当前还没有 tmux server，tmux 会启动一个后台的 **server process**。

官方文档中明确提到：tmux 的主要状态都保存在这个 server 中，它负责管理 tmux 内部运行的程序，并跟踪它们的输出。

与此同时，我们当前看到的界面属于一个 **tmux client**。

client 和 server 并不是同一个进程。

**server 才是真正保存 Session、Window、Pane 状态的一方；client 更像是当前终端进入这个工作空间的一个入口。**

tmux client 与 server 之间通过 Unix domain socket 通信。默认情况下，tmux 会在 `/tmp` 下建立 socket，由 client 通过它连接 server。

```text
Local Terminal
      │
     SSH
      │
 tmux client
      │
 Unix socket
      │
 tmux server
```

这时候 detach 的含义就变得很直观了。

按下：

```text
Ctrl-b d
```

并不是把 tmux 整体关掉，而只是**让当前 client 不再 attach 到这个 Session。**server 仍然运行。

下一次执行：

```bash
tmux attach
```

做的事情，本质上就是重新建立一个 client，再 attach 到已有 Session。

## tmux 的基本使用

### Session

直接运行：

```bash
tmux
```

就会创建一个 Session。

也可以给 Session 起个名字：

```bash
tmux new -s dev
```

方便重新连接时指定。

查看所有 Session：

```bash
tmux ls
```

例如：

```text
dev: 2 windows
test: 1 windows
```

重新连接：

```bash
tmux attach -t dev
```

也可以简写：

```bash
tmux a -t dev
```

以及，detach 当前 Session：

```text
C-b d
```


### Prefix Key

tmux 里大多数快捷操作都有一个共同的前缀：

```text
Ctrl-b
```

通常记作：

```text
C-b
```

它叫 **Prefix Key**。

比如：

```text
C-b d
```

并不是同时按下三个键，而是：

```text
按 Ctrl-b
松开
再按 d
```

### Window

创建新的 Window：

```text
C-b c
```

切换到 next Window：

```text
C-b n
```

切换到 previous Window：

```text
C-b p
```

或者直接：

```text
C-b 0
C-b 1
C-b 2
```

切换到对应编号。

也可以使用：

```text
C-b w
```

进入 Window 列表，选择要切换的 Window。

### Pane

左右分割：

```text
C-b %
```

上下分割：

```text
C-b "
```

然后可以使用：

```text
C-b ↑
C-b ↓
C-b ←
C-b →
```

在 Pane 之间移动。

```text
C-b z
```

可以临时把当前 Pane 放大到整个 Window。

再按一次恢复。

### Copy Mode

另一个有用的快捷键是：

```text
C-b [
```

进入 **Copy Mode**。

tmux 自己维护 Pane 的历史内容，但是 Pane 默认不支持鼠标滚轮翻阅历史记录，所以想向上翻看之前的输出时，Copy Mode 会很有用。

退出：

```text
q
```

如果开启 mouse support，很多滚动操作也可以直接通过鼠标完成。