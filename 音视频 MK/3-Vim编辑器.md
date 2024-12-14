核心理念：编写代码手尽量放在键盘上

Vim 详细命令：https://www.runoob.com/linux/linux-vim.html

Vim 处理模式：

- 命令模式

  拷贝、删除、粘贴等，通过i/a等键切换到编辑模式

- 编辑模式

  编辑字符，通过Esc键进行切换

Vim 常用命令

- 创建文件：vim filename，比如 vim 1.txt
- 保存文件：:w
- 退出文件：:q
- 拷贝：yy/yw
- 粘贴：p
- 删除：dd/dw
- 光标移动：左下上右：hjkl
- 光标移动：跳到文件头：gg
- 光标移动：跳到文件尾：G
- 光标移动：移动到行首：^
- 光标移动：移动到行尾：$
- 光标移动：按单词移动：向前w/ 2w/，向后b/ 2b
- 查找关键字：/关键字
- 查找与替换：:%s/关键字/替换字/gc
- 显示行号：:set number
- 指定范围查找与替换：:21,23s/关键字/替换字/gc
- 横向和纵向分窗口：:split / vsplit
- 窗口间跳转：control + ww / control + w[hjkl]

