---
title: 服务器终端环境搭建总结（zsh + Oh My Zsh + tmux + zoxide）
---

本文记录一次在 **服务器** 环境下， **zsh + tmux** 的完整配置过程。
配置属于个人喜好，仅供参考。

---

## 一、整体方案概览

最终使用的技术栈：

- Shell：**zsh**
- 框架：**Oh My Zsh（OMZ）**
- 会话管理：**tmux**
- 目录跳转：**zoxide**

---

## 二、zsh 与 Oh My Zsh 初始化

### 1. 使用 zsh 作为默认 shell

```bash
chsh -s /bin/zsh
```

---

### 2. 正确初始化 Oh My Zsh


装一些插件：
```bash
git clone https://github.com/zsh-users/zsh-autosuggestions \
${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting \
${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting

curl -sSfL https://raw.githubusercontent.com/ajeetdsouza/zoxide/main/install.sh | sh
```

激活这些插件：
```bash
omz plugin enable tmux zsh-autosuggestions zsh-syntax-highlighting zoxide
```
---

## 三、PATH 的正确添加方式（zsh）

zoxid需要配置环境变量如下：
```zsh
path+=(
  $HOME/.local/bin
)
```

---


## 四、tmux

1. 无感tmux
```zsh
#(end of file)
if [[ -o interactive ]] && [[ -z "$TMUX" ]]; then
  tmux attach -t main || tmux new -s main
fi
```
2. tmux优化
```bash
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm
```

```bash
cat > ~/.tmux.conf << 'EOF'
# --- 基础设置 ---
set -g mouse on
set -g history-limit 50000
set -g base-index 1
setw -g pane-base-index 1
set -g renumber-windows on

# --- 快捷键优化 ---
bind | split-window -h -c "#{pane_current_path}"
bind - split-window -v -c "#{pane_current_path}"
bind r source-file ~/.tmux.conf \; display "配置文件已重载!"

# --- 插件列表 ---
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'
set -g @plugin 'dracula/tmux'

# --- 插件配置 ---
set -g @continuum-restore 'on'
set -g @dracula-plugins "cpu-usage ram-usage time"
set -g @dracula-show-powerline true
set -g @dracula-show-flags true
set -g @dracula-refresh-rate 5

# --- 初始化 (必须在最后) ---
run '~/.tmux/plugins/tpm/tpm'
EOF
```

进入tmux，Ctrl+b -> Shift+i
