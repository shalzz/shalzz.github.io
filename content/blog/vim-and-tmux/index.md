+++
title="My Vim and Tmux Setup"
date=2018-05-27
[extra]
image="nvim-and-tmux.jpg"
tags="vim, tmux, dotfiles, dotconfig, .vimrc, personal, vim config, plugins, cpp, rust, c/cpp, c++/c, c++ , c, .tmux.conf, nvim, neovim"
+++

For the past couple of weeks I have been slowly setting up my [Vim][1] and [tmux][2]
config files to just the way I prefer, creating the perfect workflow for working on
C/C++ and Rust projects that increases my productivity by not only *not* having to 
constantly switch between the keyboard and the mouse/touchpad for every little
task/navigation but also to just touch type my way to glory :).
Especially since with C and Rust not having a single canonical IDE and inherently
requiring to interact with the terminal for compiling and debugging, Vim coupled 
with a language server (I'm using cquery for C/C++, RLS for Rust) *is* the 
perfect IDE for these languages.

<!-- more -->

Some of the vim plugins that you definitely need for C++ but are hard to find are
[vim-cpp-enhanced-highlight] and [tagbar] with ofcourse a decent language server
like [cquery]. You can find my complete .vim config file [here][1].

Vim and tmux work perfectly together and with the tmuxline.vim plugin even more
seamlessly.
One thing that does trip you up while setting up both vim/nvim and tmux is 
enabling true color support, [this][3] blog helped me sort that out rather quickly.

![screenshot](nvim-and-tmux.jpg "screenshot of my vim setup")

That's what my development environment looks like now.

[vim-cpp-enhanced-highlight]: https://github.com/octol/vim-cpp-enhanced-highlight
[tagbar]: https://github.com/majutsushi/tagbar
[cquery]: https://github.com/cquery-project/cquery
[1]: https://github.com/shalzz/.vim
[2]: https://github.com/shalzz/dotfiles
[3]: https://www.cyfyifanchen.com/neovim-true-color/
