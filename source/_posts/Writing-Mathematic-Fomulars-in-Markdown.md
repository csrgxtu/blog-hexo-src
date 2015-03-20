title: "Writing Mathematic Fomulars in Markdown"
date: 2015-03-20 14:22:11
tags: [Markdown, latex, mathematic]
categories: [Linux, Latex, Markdown]
---
In this post, I am gonna show you how to write Mathematic symbols in markdown. since I am writing blog post that hosted by [Github](https://github.com) with Editor [Atom](http://atom.io), and use plugin [markdown-preview-plus](https://atom.io/packages/markdown-preview-plus) and [mathjax-wrapper](https://atom.io/packages/mathjax-wrapper), and use [mathjax](https://www.mathjax.org/) Javascript display the math symbols on the web page.

I am not gonna to tell you how to make all these things work together, if you want to do what I am do, please take a little time and search around.

Most import, this post is showing you the basics about math symbols in [Latex](http://www.latex-project.org/).
<!--more-->

### Greek Letters
Symbol    | Script
----------|-------
$\alpha$  |\alpha
A         |A
$\beta$   |\beta
B         |B
$\gamma$  |\gammma
$\Gamma$  |\Gamma
$\pi$     |\pi
$\Pi$     |\Pi
$\phi$    |\phi
$\Phi$    |\Phi
$\varphi$ |\varphi
$\theta$  |\theta

### Operators
Symbol    | Script
----------|--------
$\cos$    |\cos
$\sin$    |\sin
$\lim$    |\lim
$\exp$    |\exp
$\to$     |\to
$\infty$  |\infty
$\equiv$  |\equiv
$\bmod$   |\bmod
$\times$  |\times

### Power and Indices
Symbol    | Script
----------|--------
$k_{n+1}$ |k_{n+1}
$n^2$     |n^2
$k_n^2$   |k_n^2

### Fractions and Binomials
Symbol               | Script
---------------------|--------
$\frac{n!}{k!(n-k)!}$|\frac{n!}{k!(n-k)!}
$\binom{n}{k}$       |\binom{n}{k}
$\frac{\frac{x}{1}}{x - y}$|\frac{\frac{x}{1}}{x - y}
$^3/_7$              |^3/_7

### Roots
Symbol     | Script
-----------|--------
$\sqrt{k}$ |\sqrt{k}
$\sqrt[n]{k}$|\sqrt[n]{k}

### Sums and Integrals
Symbol     | Script
-----------|--------
$\sum_{i=1}^{10} t_i$|\sum_{i=1}^{10} t_i
$\int_0^\infty \mathrm{e}^{-x}\,\mathrm{d}x$|\int_0^\infty \mathrm{e}^{-x}\,\mathrm{d}x
$\sum$     |\sum
$\prod$    |\prod
$\coprod$  |\coprod
$\bigoplus$|\bigoplus
$\bigotimes$|\bigotimes
$\bigodot$ |\bigodot
$\bigcup$  |\bigcup
$\bigcap$  |\bigcap
$\biguplus$|\biguplus
$\bigsqcup$|\bigsqcup
$\bigvee$  |\bigvee
$\bigwedge$|\bigwedge
$\int$     |\int
$\oint$    |\oint
$\iint$    |\iint
$\iiint$   |\iiint
$\idotsint$|\idotsint
$\sum_{\substack{0<i<m\\0<j<n}} P(i, j)$|\sum_{\substack{0<i<m\\0<j<n}} P(i, j)
$\int\limits_a^b$|\int\limits_a^b

### Brackets etc
Symbol     | Script
-----------|--------
$(a)$      |(a)
$[a]$      |[a]
$\{a\}$    |\{a\}
$\langle f \rangle$|\langle f \rangle
$\lfloor f \rfloor$|\lfloor f \rfloor
$\lceil f \rceil$  |\lceil f \rceil
$\ulcorner f \urcorner$|\ulcorner f \urcorner
$\left(\frac{x^2}{y^3}\right)$|\left(\frac{{x^2}{y^3}}\right)
$\left[x^2\right]$|\left[x^2\right]
$($        |(
$\big($    |\big(
$\Big($    |\Big(
$\bigg($   |\bigg(
$\Bigg($   |\Bigg(
$x \in [0, 1]$|x \in [0, 1]
$\begin{matrix}a&b&c\\d&e&f\\g&h&i\end{matrix}$|\begin{matrix}a&b&c\\\d&e&f\\\g&h&i\end{matrix}
$\begin{matrix}a&b&c\\d&e&f\\g&h&i\end{matrix}=\begin{matrix}a&b&c\\d&e&f\\g&h&i\end{matrix}$|\begin{matrix}a&b&c\\d&e&f\\g&h&i\end{matrix}=\begin{matrix}a&b&c\\d&e&f\\g&h&i\end{matrix}

Symbol    | Script
----------|--------
$a'$ $a^{\prime}$|a` a^{\prime}


### Reference
[Atom](http://atom.io) - Atom editor for hackers

[markdown-preview-plus](https://atom.io/packages/markdown-preview-plus) - preview your markdown in atom

[mathjax-wrapper](https://atom.io/packages/mathjax-wrapper) - display math symbols in atom

[mathjax](https://www.mathjax.org/) - Javascript lib for browsers

[Latex](http://www.latex-project.org/) - Latex Homepage

[Wiki Latex Mathematics](http://en.wikibooks.org/wiki/LaTeX/Mathematics) - introduction to math symbols in latex

[Github tables](https://help.github.com/articles/github-flavored-markdown/) - Github Flavored Markdown
