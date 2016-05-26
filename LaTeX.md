### Latex
__________________________________________________________________________________________
##### System of Equations
[wikibooks](https://en.wikibooks.org/wiki/LaTeX/Advanced_Mathematics)
```
f(x) = \begin{dcases*}
          a           & if n = 1\\
          f(x/2) + b  & if n > 2
       \end(dcases*}
```
#####  Mathematical Symbols
[List of LaTeX Methematical Symbols](https://oeis.org/wiki/List_of_LaTeX_mathematical_symbols)  
Percent `\%`  

##### Commands with parameters  
*Source*:[sharelatex.com](https://www.sharelatex.com/learn/Commands)  
{\Command Name}[parameter number]{\Command{#parameter number}  
`\newcommand{\bb}[1]{\mathbb{#1}}`  

##### Import PDF into Latex
*Source*:[stackexchange.com](http://tex.stackexchange.com/questions/105589/insert-pdf-file-in-latex-document)  
NOTE: There are options to only add slected pages  
```
\usepackage[final]{pdfpages}
\begin{document}
          \includepdf[pages=-]{file.pdf}
\end{document}
```  

