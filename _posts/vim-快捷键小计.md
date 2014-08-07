title: vim spf13 快捷键小记
date: 2014-08-02 21:31:16
tags:
- vim
---

[spf13 intro](http://vim.spf13.com/)

[简明vim攻略](http://coolshell.cn/articles/5426.html)

[vim分屏功能](http://coolshell.cn/articles/1679.html)

[无插件vim编程技巧](http://coolshell.cn/articles/11312.html)

Toggle comments using `<Leader>c<space>`

mechanism to load files from the file system `<c-p>`

Use `<C-E>` to toggle NERDTree

Use `<leader>e` or `<leader>nt` to load NERDTreeFind

`<leader>gs` :Gstatus

`<leader>gd` :Gdiff

`<leader>gc` :Gcommit

`<leader>gb` :Gblame

`<leader>gl` :Glog

`<leader>gp` :Git push

`<Leader>a=` :Tabularize /=

`<Leader>a:` :Tabularize /:

`<Leader>a:`: :Tabularize /:\zs

`<Leader>a,` :Tabularize /,

`<Leader>a<Bar>` :Tabularize /

EasyMotion is triggered using the normal movements, but prefixing them with `,,w`


`map <leader>tn :tabnew<cr>`

`map <leader>to :tabonly<cr>`

`map <leader>tc :tabclose<cr>`

`map <leader>tm :tabmove`

`map <leader>te :tabedit <c-r>=expand("%:p:h")<cr>`

`map <leader>cd :cd %:p:h<cr>:pwd<cr>`

`map <leader>g :vimgrep // **/*.<left><left><left><left><left><left><left>`

`map <leader><space> :vimgrep // <C-R>%<C-A><right><right><right><right><right><right><right><right><right>`

关闭当前窗口 `Ctrl+W c`

关闭当前窗口，如果只剩最后一个了，则退出Vim `Ctrl+W q`

让所有的屏都有一样的高度 `Ctrl+W =`

增加高度 `Ctrl+W +`

减少高度。 `Ctrl+W -`

宽度你可以使用 `[Ctrl+W <]`或是`[Ctrl+W >]`

buffernext `:bn` bufferprevious `:bp`
