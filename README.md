# dottmux
Manage tmux config and plugins like [dotvim][7].

配置的 tmux 效果如下：

![tmux demo aminated gif](img/tmux-demo.gif)

## 安装

```
$ git clone --recurse-submodules https://github.com/schnell18/dottmux ~/.tmux
$ ln -sf ~/.tmux/tmux.conf ~/.tmux.conf
```
首次启动 tmux 请运行`C a I`（也就按 Control a Shift i）安装插件。本项目预装的
插件有：

- [tmux-plugins/tmux-sensible][1]
- [tmux-plugins/tmux-prefix-highlight][2]
- [tmux-plugins/tmux-resurrect][3]
- [tmux-plugins/tmux-continuum][4]
- [schnell18/tmux-weather][5]
- [olimorris/tmux-pomodoro-plus][6]

## 脚本打开 tmux

对于复杂的多窗口项目，可以使用预先定义好的脚本来打开 tmux。以下是一个典型的例子：

    #!/bin/sh

    # Check to see if we're already running the session
    tmux has-session -t main &> /dev/null

    if [ $? != 0 ] ; then
        # Create overall tmux session
        tmux new-session -d -s main -n java > /dev/null

        tmux split-window -v
        tmux split-window -h
        tmux send-keys -t main:1.1 "cd ~/work" C-m
        tmux send-keys -t main:1.2 "cd ~/work" C-m
        tmux send-keys -t main:1.3 "cd ~/work && jshell" C-m

        # Create window for running nexus
        tmux new-window  -n nexus
        tmux send-keys -t main:2 "cd ~/apps/nexus-3.30.1-01/bin && ./nexus start" C-m

        # Create window for running docker-compose
        tmux new-window  -n docker
        tmux split-window -v
        tmux split-window -h
        tmux send-keys -t main:3.1 "cd ~/virtualenv && docker-compose logs -f" C-m
        tmux send-keys -t main:3.2 "cd ~/virtualenv && mysql -u mfg -pabc -P 3307 -h 127.0.0.1" C-m
        tmux send-keys -t main:3.3 "cd ~/virtualenv && redis-cli -h 127.0.0.1 -p 6379" C-m

    else
        echo "tmux session already running, attaching..."
        sleep 2
    fi

    tmux select-window -t 1
    tmux attach

## 配置项

### 修改指令前缀

可根据自己的喜好来设置，本项目使用 Ctrl + a 作为前缀。

```sh
#set -g prefix C-a #
#unbind C-b # C-b 即 Ctrl+b 键，unbind 意味着解除绑定
#bind C-a send-prefix # 绑定 Ctrl+a 为新的指令前缀

# 从 tmux v1.6 版起，支持设置第二个指令前缀
#set-option -g prefix2 ` # 设置一个不常用的`键作为指令前缀，按键更快些
```
### 添加加载配置文件快捷指令 r

```sh
bind r source-file ~/.tmux.conf \; display-message "Config reloaded.."
```

### 支持鼠标

* 选取文本
* 调整面板大小
* 选中并切换面板

```sh
# 老版本：
#setw -g mode-mouse on # 支持鼠标选取文本等
#setw -g mouse-resize-pane on # 支持鼠标拖动调整面板的大小（通过拖动面板间的分割线）
#setw -g mouse-select-pane on # 支持鼠标选中并切换面板
#setw -g mouse-select-window on # 支持鼠标选中并切换窗口（通过点击状态栏窗口名称）

# v2.1 及以上的版本
set-option -g mouse on
```
### 面板
#### 更改新增面板键
* - 垂直新增面板
* + 水平新增面板

```sh
unbind '"'
bind - splitw -v -c '#{pane_current_path}' # 垂直方向新增面板，默认进入当前目录
unbind %
bind =  splitw -h -c '#{pane_current_path}' # 水平方向新增面板，默认进入当前目录
```

#### 面板调整大小

绑定 Ctrl+hjkl 键为面板上下左右调整边缘的快捷指令

```sh
bind -r ^k resizep -U 10 # 绑定 Ctrl+k 为往↑调整面板边缘 10 个单元格
bind -r ^j resizep -D 10 # 绑定 Ctrl+j 为往↓调整面板边缘 10 个单元格
bind -r ^h resizep -L 10 # 绑定 Ctrl+h 为往←调整面板边缘 10 个单元格
bind -r ^l resizep -R 10 # 绑定 Ctrl+l 为往→调整面板边缘 10 个单元格
```

### 复制模式
#### 复制模式更改为 vi 风格

注意： 进入复制模式 快捷键：prefix + [

```sh
setw -g mode-keys vi # 开启 vi 风格后，支持 vi 的 C-d、C-u、hjkl 等快捷键
```

#### 复制模式向 vi 靠拢

* v 开始选择文本
* y 复制选中文本
* p 粘贴文本

```sh
bind -t vi-copy v begin-selection # 绑定 v 键为开始选择文本
bind -t vi-copy y copy-selection # 绑定 y 键为复制选中文本
bind p pasteb # 绑定 p 键为粘贴文本（p 键默认用于进入上一个窗口，不建议覆盖）
```

### 优化

#### 设置窗口面板起始序号

```sh
set -g base-index 1 # 设置窗口的起始下标为 1
set -g pane-base-index 1 # 设置面板的起始下标为 1
```
#### 自定义状态栏

``` sh
set -g status-utf8 on # 状态栏支持 utf8
set -g status-interval 1 # 状态栏刷新时间
set -g status-justify left # 状态栏列表左对齐
setw -g monitor-activity on # 非当前窗口有内容更新时在状态栏通知

set -g status-fg yellow # 设置状态栏前景黄色
set -g status-style "bg=black, fg=yellow" # 状态栏前景背景色

set -g status-left "#[bg=#FF661D] 🐶 #S " # 状态栏左侧内容
set -g status-right 'Continuum status: #{continuum_status}' # 状态栏右侧内容
set -g status-left-length 300 # 状态栏左边长度 300
set -g status-right-length 500 # 状态栏左边长度 500

set -wg window-status-format " #I #W " # 状态栏窗口名称格式
set -wg window-status-current-format " #I:#W#F " # 状态栏当前窗口名称格式 (#I：序号，#w：窗口名称，#F：间隔符）
set -wg window-status-separator "" # 状态栏窗口名称之间的间隔
set -wg window-status-current-style "bg=red" # 状态栏当前窗口名称的样式
set -wg window-status-last-style "fg=red" # 状态栏最后一个窗口名称的样式

set -g message-style "bg=#202529, fg=#91A8BA" # 指定消息通知的前景、后景色

```

[1]: https://github.com/tmux-plugins/tmux-sensible
[2]: https://github.com/tmux-plugins/tmux-prefix-highlight
[3]: https://github.com/tmux-plugins/tmux-resurrect
[4]: https://github.com/tmux-plugins/tmux-continuum
[5]: https://github.com/aaronpowell/tmux-weather
[6]: https://github.com/olimorris/tmux-pomodoro-plus
[7]: https://github.com/schnell18/dotvim.git
