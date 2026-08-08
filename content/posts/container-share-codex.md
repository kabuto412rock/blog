---
title: "在 Dev Container 裡共用 Codex CLI"
date: 2026-08-08T11:26:30Z
# weight: 1
# aliases: ["/first"]
tags: ["codex", "docker", "dev container"]

draft: false
---

# 前言
身為一個喜歡在 Dev Container 裡開發的軟體工程師，時常會切換不同的容器，但依然需要 Codex CLI 輔助開發。每次重建容器都要重新安裝、登入 Codex CLI，
重複作業令人煩躁，所以有了這篇文章。

# 解法
## 整體思路
我的做法是將安裝、登入 Codex CLI 後的資料都存放在容器間共用的 volume，
當容器啟動時會先檢查 Codex CLI 是否存在，存在則略過，不存在則嘗試安裝。

後續只要登入過一次，其他容器就能直接使用😁

> 限制大概是這些容器的作業系統環境需要相同，而且必須使用 Dev Container 搭配 docker-compose.yml。

### 第一步：建立共享 volume
```bash
# 在本機終端機執行
docker volume create codex-home
```

### 第二步：修改 docker-compose.yml
```yml
volumes:
  codex-home:
    external: true

services:
  app:
    volumes:
      - codex-home:/home/vscode/.codex
```

### 第三步：修改 Dockerfile

```Dockerfile
ENV PATH="/home/vscode/.codex/bin:${PATH}"

RUN mkdir -p /home/vscode/.codex \
    && chown vscode:vscode /home/vscode/.codex
```

### 第四步：修改 devcontainer.json
調整 postCreateCommand 欄位的目的是確保以下情況，
若沒有安裝 Codex CLI 才會執行安裝：
1. 第一次建立容器
2. 使用 Rebuild Container 重新建立容器
```json
{
    "postCreateCommand": "原本的命令 && (test -x \"$HOME/.codex/bin/codex\" || curl -fsSL https://chatgpt.com/codex/install.sh | env CODEX_INSTALL_DIR=\"$HOME/.codex/bin\" CODEX_NON_INTERACTIVE=true sh)"
}
```

## 結論
願大家都能更順利使用 AI 實現自己的想法，
至少不會再因為設定環境而苦惱，哈哈！

[Codex CLI 官方文件](https://learn.chatgpt.com/docs/codex/cli)
