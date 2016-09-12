## Vim
_________________________________________________________________________________________________________  

## Change Text Between Punctuation  
*Source* [Replace text between quotes in vi](http://stackoverflow.com/questions/11630440/how-to-replace-text-between-quotes-in-vi)  

* `ci'` - change inside the single quotes
* `ciw` - change inside a word
* `ci(` - change inside parentheses
* `dit` - delete inside an HTML tag, etc.

## Set Autocommands  
*Source*: [Autocommands](http://learnvimscriptthehardway.stevelosh.com/chapters/12.html)  

In vim its convenient to set autocommands to run a desired action in the background as well as before or after an action happens.
Autocommands are set with in the `vimrc` by adding the fallowing line:
```
:autocmd BufNewFile * :write
         ^          ^ ^
         |          | |
         |          | The command to run.
         |          |
         |          A "pattern" to filter the event.
         |
         The "event" to watch for.
```
A list of events that can be used can be found vim documentation page [here](http://vimdoc.sourceforge.net/htmldoc/autocmd.html).   
Multiple events can be specified using the fallowing syntax: `event1, event2`.  
