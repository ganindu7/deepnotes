---
layout: default
title: setting up YCM
# nav_order: 3 
permalink: /topics/utils/code_completion
parent: Utilities
---

## Setting up [YCM][YCM-URL]


Prerequisites:  
* install pyenv and virtual environment as mentioned [here](./setting_up_pyenv#building-with---enable-shared).

YCM is a vim plugin , install [Vundle](https://github.com/VundleVim/Vundle.vim) first and and then install YCM,  

*	set up Vundle:
	`git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim`  

*	Update/create the `~/.vimrc` file with following lines. There are detailed instructions [here](https://github.com/VundleVim/Vundle.vim#quick-start)

below here is an example

```vim
set nocompatible
filetype off
 
set rtp+=~/.vim/bundle/Vundle.vim
call vundle#begin()
 
" let Vundle manage Vundle, required
Plugin 'VundleVim/Vundle.vim'
Plugin 'Valloric/YouCompleteMe'
 
"All of your Plugins must be added before the following line
call vundle#end()            " required

"Not really needed because of "shared" option when building python
"let g:ycm_path_to_python_interpreter = '$HOME/.pyenv/shims/python'
let g:ycm_clangd_binary_path="/usr/bin/clangd-12"

filetype plugin indent on    " required
"To ignore plugin indent changes, instead use:
"filetype plugin on
"
" Brief help
" :PluginList       - lists configured plugins
" :PluginInstall    - installs plugins; append `!` to update or just :PluginUpdate
" :PluginSearch foo - searches for foo; append `!` to refresh local cache
" :PluginClean      - confirms removal of unused plugins; append `!` to auto-approve removal
"
" see :h vundle for more details or wiki for FAQ
" Put your non-Plugin stuff after this line
 
" :W sudo saves the file when the file is open in readonly mode.
 
"command W w !sudo tee % > /dev/null
 
" ***********  Line **********
" show line numbets
"
set number
 
 
" ********** Indents *********
"replace tabs with spaces
"
set expandtab
" 1 tab =  2 spaces
"
set tabstop=2 shiftwidth=2

" when deleting whitespaces at the begining of a line, delete 1 tab worth of
" spaces
"
set smarttab
 
" when creating a new line, copy the indentation from the line above
"
set autoindent
 
" ********* Search **********
" ignorecase 

set ignorecase
set smartcase
 
" highlight search results 
"
set hlsearch
 
" highlight all pattern matches 
"
set incsearch
 
" ************ Code stuff  *****
"
set showmatch
"set cursorline 
```

*	Now install the plugin by launching `vim` and running `:PluginInstall`	

You may need to [update cmake](installing_cmake_locally) or [GCC](setup_cXX) or [LLVM][LLVM-LINK]

note that in the above listing I've used `let g:ycm_clangd_binary_path="/usr/bin/clangd-12"` to point to the [clangd][CLANGD-LINK] server.

## Configuring projects

The following is taken from the YCM [readme][YCM-README] and the [wiki][YCM-WIKI]

If you are using CMake add `-DCMAKE_EXPORT_COMPILE_COMMANDS=ON` when configuring or add 
`set(CMAKE_EXPORT_COMPILE_COMMANDS ON )` to `CMakeLists.txt` and copy or symlink the database to the root pf the project.

the second method is having a `ycm_extra_conf.py` in the project dir (in or above the source dir as it searches upward until it finds a command file or a extra config python file) check [this example][SAMPLE-CONF-PY] provided by YCM developers. 




```python
def Settings(**kwargs):
  return { 'flags': list_of_compiler_flags,
           'override_filename': path_to_actually_compiled_file }
```



Note: [Clangd for arm64 can be downloaded(/via apt) [here][CLANGD-LINK].


for clangd to find the `compile_commands.json` we need it to be in a top-level directory relative to the source directory. 

sadly fot this to work  

```
IF(EXISTS "${CMAKE_CURRENT_BINARY_DIR}/compile_commands.json")
  EXECUTE_PROCESS(COMMAND ${CMAKE_COMMAND} -E create_symlink "${CMAKE_CURRENT_BINARY_DIR}/compile_commands.json" "${PROJECT_SOURCE_DIR}/compile_commands.json")
message(STATUS "******* symlinks generated *******")
ENDIF()

```


[YCM-URL]: https://github.com/ycm-core/YouCompleteMe/blob/master/README.md
[CLANGD-LINK]: https://clangd.llvm.org/
[LLVM-LINK]: https://apt.llvm.org/
[YCM-README]: https://github.com/ycm-core/YouCompleteMe#c-family-semantic-completion
[YCM-WIKI]: https://github.com/ycm-core/YouCompleteMe/wiki/C-family-Semantic-Completion-through-libclang
[SAMPLE-CONF-PY]:https://raw.githubusercontent.com/ycm-core/ycmd/66030cd94299114ae316796f3cad181cac8a007c/.ycm_extra_conf.py

[CLANGD-LINK]: https://ubuntu.pkgs.org/20.04/ubuntu-universe-arm64/clangd-10_10.0.0-4ubuntu1_arm64.deb.html

