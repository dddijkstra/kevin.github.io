# 服务器终端环境搭建总结（zsh + Oh My Zsh + tmux + zoxide）

本文记录一次在 **服务器 / 容器 / VSCode 集成终端** 环境下，
从 bash 迁移到 **zsh + tmux** 的完整配置过程，以及中途踩过的一些坑和最终的稳定方案。

目标很明确：**稳定、顺手、不过度折腾**。

---

## 一、整体方案概览

最终使用的技术栈：

- Shell：**zsh**
- 框架：**Oh My Zsh（OMZ）**
- 会话管理：**tmux**
- 目录跳转：**zoxide**
- 使用场景：服务器 / Docker / VSCode Terminal

核心原则：

> 服务器环境优先 **稳定和可控**，不追求花哨主题和重插件。

---

## 二、zsh 与 Oh My Zsh 初始化

### 1. 使用 zsh 作为默认 shell

```bash
chsh -s /bin/zsh
```

tmux 中也强制使用 zsh：

```tmux
set -g default-shell /bin/zsh
set -g default-command /bin/zsh
```

---

### 2. 正确初始化 Oh My Zsh

`.zshrc` 的关键点：

- `source $ZSH/oh-my-zsh.sh` **只能出现一次**
- 插件要少
- 服务器/容器不要用复杂主题

一个稳定模板示例：

```zsh
export ZSH="$HOME/.oh-my-zsh"
DISABLE_AUTO_UPDATE="true"

ZSH_THEME="robbyrussell"

plugins=(
  git
  zsh-autosuggestions
  zsh-syntax-highlighting
)

source $ZSH/oh-my-zsh.sh

# syntax-highlighting 必须放最后
source ${ZSH_CUSTOM:-$ZSH/custom}/plugins/zsh-syntax-highlighting/zsh-syntax-highlighting.zsh
```

---

## 三、PATH 的正确添加方式（zsh）

在 zsh 中，推荐直接操作 `path` 数组，而不是反复 `export PATH`。

```zsh
path+=(
  $HOME/bin
  $HOME/.local/bin
)
```

建议 **放在 source oh-my-zsh 之前**。

---

## 四、模糊跳目录（zoxide）

### 1. 安装与初始化

```bash
sudo apt install zoxide
```

```zsh
eval "$(zoxide init zsh)"
```

---

### 2. 将 `cd` 升级为“智能 cd”

```zsh
cd() {
  if [[ $# -eq 0 ]]; then
    builtin cd ~
  elif [[ -d "$1" || "$1" == /* || "$1" == .* ]]; then
    builtin cd "$@"
  else
    z "$@"
  fi
}
```

---

## 五、tmux 自动进入（不使用 OMZ tmux 插件）

```zsh
if [[ -o interactive ]] && [[ -z "$TMUX" ]]; then
  tmux attach -t main || tmux new -s main
fi
```

---

## 六、tmux 基础优化

```tmux
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"
set -s escape-time 0
set -g detach-on-destroy off
```

---

## 七、VSCode 集成终端修正

```json
"terminal.integrated.defaultProfile.linux": "zsh",
"terminal.integrated.shellIntegration.enabled": false
```

---

## 八、踩坑总结

- 多行 prompt 在容器下挡住输入
- `alias cd='z'` 行为反直觉
- OMZ tmux 插件在 VSCode 下不稳定
- bash → zsh → tmux 多层嵌套

---
