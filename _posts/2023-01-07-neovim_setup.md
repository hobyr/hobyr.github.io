# How I set my Neovim up for C/C++ & Python

1. TOC
{:toc}

---

Running almost always in Ubuntu, I wanted to get familiar with the
terminal and its tools. One of these tools are text editors, and I read
countless pieces about Vim being better than Emacs and vice-versa.

I've been running with Vim for almost 2 years now and I find it a superb text editor. After finding
out about Neovim, supposedly an improvement over Vim, I wanted to give it a try.

Although it took me several months and plugins to find a suitable Vim configuration, I feared moving
to Neovim would be a headache but not at all! The transition to Neovim was almost seamless.

One of the reasons I like working in the terminal, is that I have direct access to the shell by
hitting `Ctrl-Z` (and running `fg` from shell to return to the editor), instead of switching apps.

Even better, I can use it in `tmux` too, a great built-in terminal multiplexer which I'm currently
using. I'll have two `tmux` panes side-by-side, one with Bash, and the other with Neovim.

Since I do a lot of work in the terminal too, like compiling, listing or creating files/directories,
that combo is a no-brainer.

In this post, I'll show you a way to customize Neovim to make your coding life easier, and also make it look great so you can show off!

We'll be going from this:

![](/images/2023-01-07-neovim_setup_assets/before_look.png)

to this:

![](/images/2023-01-07-neovim_setup_assets/final_look.png)

## My configuration

Vim/Neovim is just a text editor, and in its raw form, it's not particularly suitable for programming, so there's a little bit of configuring to do. 

### How to configure Vim/Neovim ##

Configuring Vim is done by modifying the `.vimrc` file in the `$HOME` folder. Neovim uses a different file, `$HOME/.config/nvim/init.vim` but there's no need to copy and paste the contents from one file to another. 

A few simple commands will tell Neovim to look in the former for its configuration.

By adding the following code to the `init.vim` file:

```
set runtimepath^=~/.vim runtimepath+=~/.vim/after
let &packpath = &runtimepath
source ~/.vimrc
```

modifying the `.vimrc` will configure both Vim and Neovim. 

To start extending the functionality of Vim/Neovim, I can use built-in commands, but to really make this editor my own, using plugins is the way to go.

First, I'll go over the basic commands to start improving Vim/Neovim, then I'll list some plugins I use daily to make my life easier when writing code.

### Basic configuration commands ##

#### Line numbering ###

When using an IDE or a text editor aimed at software developers, one of the first features is _line numbering_.

When first starting Vim/Neovim and a blank `.vimrc`, this feature is _inactive_. To activate, add `set number` to `.vimrc`.

#### Current line highlighting ###

Some editors such as Visual Studio Code offer that feature, and though it's subtle, it's really helpful to see where your cursor is when you have many lines of code in front of you.

Vim/Neovim also have that feature _inactive_ at first. To activate, I added `set cursorline` to `.vimrc`.

#### Basic syntax highlighting ###

Syntax highlighting is so much helpful when writing code, it helps identify keywords, variables and functions names, etc. It's also a basic feature of any code editor and IDE.

A hidden feature like the previous two, activating syntax highlighting in Vim/Neovim is very simple: add `syntax on` to the `.vimrc`.

I find the native syntax highlighting feature quite lacking and I found a great plugin to enhance it. I'll write more about it later.

#### Indentation settings ###

When writing code, we very often, if not always, use indentation to improve readability. Indentation is defined by _tab stops_. Depending on standards, a tab stop can be equivalent to 2 or 4 spaces.

A common value is 4 spaces, and we can configure this behavior in Vim/Neovim. To set indentation settings to 4 in Vim/Neovim, I added the following to `.vimrc`:

```
set tabstop=4
set shiftwidth=4
set expandtab
```

So while writing code, let's say C/C++, opening a new block will automatically indent the code by 4 spaces relative to the parent block.

#### The configuration up to now ###

The `.vimrc` should look like this now:

```
set number
set cursorline
syntax on
set tabstop=4
set shiftwidth=4
set expandtab
```

Once I saved the `.vimrc` and restarted Vim/Neovim, I felt it was closer to a code editor rather than a simple text editor.

###  Plugins for Vim/Neovim

Plugins heavily extend the functionality of Vim/Neovim, and I could make it like an IDE if I wanted. I still want Vim/Neovim to be fast and have just the features I need, not more.

Almost every plugin I used with Vim is compatible with Neovim, except for the colorscheme.

I write mainly in C/C++ and some Python, so finding plugins suitable for these languages was necessary, and upon retrospect quite easy.

#### How to properly add plugins to Vim/Neovim ###

First of all, at the top of the `.vimrc`, I have to add 3 commands:

``` 
set nocompatible
filetype plugin on
set secure
```

This will ensure that I don't have any compatibility issues with Vi (Vim "ancestor"), and that there will be filetype detection and proper plugin loading according to the filetype.

There are many plugin managers for Vim/Neovim and one of the most popular is [vim-plug](https://github.com/junegunn/vim-plug).

After installing it, I just have to add the plugins between the `call plug#begin()` and `call plug#end()` commands, like this:

```
call plug#begin()

" This is where
" you install
" all your plugins

call plug#end()
```

To install the plugins, I have to run `:PlugInstall`.

#### Colorscheme and syntax highlighting

I admit I spent a lot of time deciding between colorschemes, it's an endless quest for the seemingly perfect one. There's none, but I found mine.

I'm now running the [Tokyonight](https://github.com/folke/tokyonight.nvim) colorscheme and I'm very satisfied.

And as I wrote above, native syntax highlighting is not so great in my opinion. Thankfully, [vim-polyglot](https://github.com/sheerun/vim-polyglot) by sheerun enhances syntax highlighting by a lot. My code is much more colored but it's not overwhelming and it helps me distinguish syntactic elements more clearly.

Let's add them to the `.vimrc`:

```
call plug#begin()
Plug 'sheerun/vim-polyglot'
Plug 'folke/tokyonight.nvim', { 'branch': 'main' }
call plug#end()
```

To activate Tokyonight, I'll add `colorscheme tokyonight` to my `.vimrc`.

#### A little more on appearance ###

By browsing Reddit, I found really good-looking Vim/Neovim setups and I wanted to improve upon an already good setup.

Before I switched to Neovim, I was using [vim-airline](https://github.com/vim-airline/vim-airline) on Vim, a statusline to improve the look and have more info displayed. Unfortunately, it is incompatible with Neovim.

When I searched for a proper Neovim colorscheme, Tokyonight mentioned a similar plugin to vim-airline, [lualine](https://github.com/nvim-lualine/lualine.nvim). After a bit of configuring, I saw no change in behavior between this and vim-airline.

Let's add it to my `.vimrc` in between `call plug#begin()` and `call plug#end()`:

```
Plug 'nvim-lualine/lualine.nvim'
Plug 'kyazdani42/nvim-web-devicons'
```

At this point, I was quite happy that I customized Vim/Neovim a little more to my taste. I started writing some code in it, but it lacked some features compared to some IDEs.

#### LSP or Language Server Protocol

By default, Vim/Neovim have an autocompletion feature. Actually, Neovim is slightly better than Vim in that regard since it automatically shows a suggestion menu based on what's already in your content.

But when it comes to writing code, for example Python, the code editor doesn't know anything about Python and doesn't show any proper suggestions.

To help with that, Microsoft created LSP (Language Server Protocol).

When I was using Vim, I was using the [YouCompleteMe](https://github.com/ycm-core/YouCompleteMe) plugin. Since I now use Neovim, I chose to use [Conquer of Completion](https://github.com/neoclide/coc.nvim).

After installing it by adding `Plug 'neoclide/coc.nvim', { 'branch': 'release' }` to the `.vimrc`, it's important to configure it and install proper [extensions](https://github.com/neoclide/coc.nvim/wiki/Using-coc-extensions).

After everything is set, Neovim should show proper suggestions when writing code. 

#### Snippets engine ###

Snippets can make you slightly faster when writing code. 

For example, when using a for loop, instead of writing the entire code for the loop, the snippet engine will generate it for you, and you just need to identify the looping variable and the ending condition, then the code in the loop.

One plugin I found that works seamlessly with Conquer of Completion is `vim-snippets` from honza. It requires no configuration.

I simply added `Plug 'honza/vim-snippets` to the `.vimrc`, installed it by running `:PlugInstall` and I was done.

#### Syntax checkers

When I'm coding, I'm bound to make some syntactic mistakes. For example in C++, I might forget a semicolon at the end of a statement, or count the wrong number of closing curly braces with nested blocks.

_Code linters_ are there to notify when you're not using a variable you declared or when you've made a syntactic mistake.

I use the [Asynchronous Lint Engine](https://github.com/dense-analysis/ale) to analyse my code. It's quite fast and reliable.

I added `Plug 'dense-analysis/ale` to my `.vimrc` for that.

## Conclusion

In this post, we've learned how to:

- Add and modify the `.vimrc` to customize our Vim/Neovim editor.
- Enable the basic features that make up a code editor such as line numbering and syntax coloring.
- Add plugins to further enhance the abilities of Vim/NeoVim.
- Add tools to make coding easier with snippets and a LSP.

By now, you should have a pretty functional Vim/Neovim code editor that is light and fast. While I do have other plugins installed, the ones I showed you today make the foundation of my Vim/Neovim setup.

I've been running with that specific setup for a couple of months and it really suits my needs. I can still extend Vim/Neovim in the future if needed so that's why I think it's a great editor.
