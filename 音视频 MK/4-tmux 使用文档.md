[Tmux 使用教程](https://www.ruanyifeng.com/blog/2019/10/tmux.html)

Tmux 最简单操作流程：

1. 新建会话：tmux new -s my_linux
2. 在 Tmux 窗口运行所需的程序
3. 按下快捷键Ctrl+b d将会话分离
4. 下次使用时，重新连接到会话 tmux attach-session -t my_linux
5. 查看会话：tmux ls
