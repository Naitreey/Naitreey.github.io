---
layout: post
title: Vim-LaTeX Feature Round Up
tags: [vim,latex]
---

This is mostly an answer to a question on [Vi.SE](http://vi.stackexchange.com/questions/2047/what-are-the-differences-between-tex-plugins).

There are _many_ features present in Vim-LaTeX. I don't remember all of them. I'll just talk about features that I know and use constantly.

Note: These are _my limited user experience_, which may be _very misleading_. I'm not a seasoned Vim user. And I know of nothing about vimscript.

# IMAP() and `<C-j>` Jumpping #
`IMAP()` function and `<C-j>` jumpping functions are provided separately as a plugin `imaps.vim` in Vim-LaTeX bundle. They are powerful features and could be very useful even when you are not writing LaTeX.

- `IMAP()` function provides a more natural way to do insert mode mappings and templating in general than the built-in `imap` and `iabbrev`, IMO.

- `<C-j>` jumpping is utilized by many Vim-LaTeX completion features. A jumpping point is indicated by `<++>`.

- Built-in insert mode key mappings are implemented as `IMAP()` calls. For example, you can find a long list of useful `IMAP()` calls in `main.vim` file:

```vim
call IMAP ('__', '_{<++>}<++>', "tex")
call IMAP ('()', '(<++>)<++>', "tex")
call IMAP ('[]', '[<++>]<++>', "tex")
call IMAP ('{}', '{<++>}<++>', "tex")
...
call IMAP ('((', '\left( <++> \right)<++>', "tex")
call IMAP ('[[', '\left[ <++> \right]<++>', "tex")
call IMAP ('{{', '\left\{ <++> \right\}<++>', "tex")
...
```

Then when you type say `()`, the cursor will reside automatically between the parenthese, replacing the first `<++>`. After you finished typing inside, you kick `<C-j>` and bang, the cursor will move out of parenthese and you just keep typing forward. Once you are used to it, it begins to form a typing flow which is kinda addictive...

You see from above a `\left` `\right` pair can be typed easily with double stroke of its opening bracket. And `<C-j>` jumpping makes typing flow.

One major glitch of `IMAP()` and `<C-j>` thing is that *they messes up your last change history*. (One bug I wish to fix for a long time.) Therefore, you may encounter unexpected behavior when trying to redo your last change by `.` if your "supposed last change" contains these function calls.

- You can do all kinds of mappings using `IMAP()`, from simple key mappings to more complex templating. Here are some examples of my mappings (`ftplugin/tex.vim`):

```vim
call IMAP('*EEQ',"\\begin{equation*}\<CR><++>\<CR>\\end{equation*}<++>",'tex')
call IMAP('DEF',"\\begin{definition}[<++>]\<CR><++>\<CR>\\end{definition}<++>",'tex')
call IMAP('BIC','\binom{<++>}{<++>}<++>','tex')
call IMAP('PVERB','\PVerb{<++>}<++>','tex')
call IMAP('VERB','\verb|<++>|<++>','tex')
```

- An interesting fact about `imaps.vim` plugin is that it's a global plugin, which implies its potential usage beyond LaTeX. Indeed, I do use `<++>` and `<C-j>` jumppings (combining with other plugins) to build code snippet templates in C.

# `<F5> <F7>` Insertion of Commands and Environments #
One disadvantage of `IMAP()` is that the key combination can not be used in normal text anymore (unless you undo the mapping by `u`). In the cases that you just want to _trigger_ the mapping as you wish, the `<F5>` and `<F7>` come in handy. These two keys are used for _triggering_ environments and inline commands insertion, respectively. And they behave differently based on the mode and customizations from user.

- In Insert/Normal Mode, when the cursor is attaching a word or is in the word, pressing `<F5>` will by default insert a basic environment of the form

```latex
\begin{word}

\end{word}<++>
```

 based on the word; pressing `<F7>` will by default insert a basic inline command of the form `\word{}<++>` based on the word. 

- "By default", I mean you can customize the behavior of a specific word when triggered by `<F5>`/`<F7>`. Here are some of my settings (`.vimrc`):

```vim
let g:Tex_Com_newcommand = "\\newcommand{<++>}[<++>]{<++>}<++>"
let g:Tex_Com_latex = "{\\LaTeX}<++>"
let g:Tex_Com_D = "\\D{<++>}{<++>}<++>"
```

- In Insert/Normal Mode, when the cursor is _not_ attached to anything (a.k.a _alone_), pressing `<F5>`/`<F7>` will prompt to you a menu to select environment/command to insert. Or you can type the name of desired environment/command at the bottom. Personally, I rarely use `<F5>`/`<F7>` this way.

- Press `<F5>`/`<F7>` after visually selecting a piece of text will prompt a menu for _wrapping text_. Then the selected text will be wrapped in the environment/command you selected or typed.

- In Insert/Normal Mode, when the cursor is in the scope of an environment/command, press `<Shift>+<F5>/<F7>` will prompt a menu for _changing environment/command_.

# Misc Key Mappings #
- Greek Letters. `` `a`` to `` `z`` and corresponding capitals.
- Symbols like `` `8`` for `\infty`, `` `<`` for `\le`, `` `I`` for `\int_{<++>}^{<++>}<++>`, etc.
- Pressing `"` twice gets a pair of normal TeX quotes. So to type literal `"` character, you have to use <C-v>.
- Pressing <Alt-i> in several enumeration environments will insert appropriate `\item` tag.
- You can wrap visually selected part of math in `\left` `\right` pair by `` `(``, `` `[`` and `` `{``.
- Folding is customizable. Three global variable control what can be folded: `Tex_FoldedSections`, `Tex_FoldedMisc`, and `Tex_FoldedEnvironments`.

Sometimes the built-in mappings have just gone too far or are not quit what you want. You can override the built-in mappings by redefining them in `after/ftplugin/tex.vim`:

```vim
call IMAP('`|','\abs{<++>}<++>','tex')
call IMAP('ETE',"\\begin{table}\<CR>\\centering\<CR>\\caption{<+Caption text+>}\<CR>\\label{tab:<+label+>}\<CR>\\begin{tabular}{<+dimensions+>}\<CR><++>\<CR>\\end{tabular}\<CR>\\end{table}<++>",'tex')
call IMAP('==','==','tex')
call IMAP('`\','`\','tex')
```

# Set Multiple Compilation Engine #
I always need to switch between `pdflatex` and `xelatex` engine. Thus, I have the following lines in my `.vimrc`:

```vim
"switch to pdflatex
function SetpdfLaTeX()
	let g:Tex_CompileRule_pdf = 'pdflatex --interaction=nonstopmode -synctex=1 -src-specials $*'
endfunction
noremap <Leader>lp :<C-U>call SetpdfLaTeX()<CR>

"switch to xelatex
function SetXeLaTeX()
	let g:Tex_CompileRule_pdf = 'xelatex --interaction=nonstopmode -synctex=1 -src-specials $*'
endfunction
noremap <Leader>lx :<C-U>call SetXeLaTeX()<CR>
```

# View PDF, Forward and Backward Search between Vim and PDF viewer #
This is a messy and complicated topic. With certain PDF viewer and _a certain amount of luck_, it can be very easy. But it's mainly a matter of google search.

# Suggestions #
- You should find your balance between LaTeX way and Vim-LaTeX way.
- Vim-LaTeX is no light-weight at all. There are some features and/or key mappings you'll possibly never gonna use and you have to manually override them.
- You use Vim. You know what _patience_ mean. :-)

Overall, I think it will work well if you are willing to invest some time to tame the beast. That being said, had I time and adequate knowledge, I will surely thin off the overhead features and explore the potentials of integration with other plugins.

# References #
- [Official user manual](http://vim-latex.sourceforge.net/index.php?subject=manual&title=Manual#user-manual)
- [Cheat sheet](http://michaelgoerz.net/refcards/vimlatexqrc.pdf)

