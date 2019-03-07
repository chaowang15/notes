# Unix Shell 
## Basic

### Ref:
- [The Unix Shell: Summary of Basic Commands](https://swcarpentry.github.io/shell-novice/reference/)

## Directory
### `du -sh`
Get total size (`-s`) of all files in the directory in human readable format (`-h`), e.g., 1K, 234M, etc.

### `ls -al`
Show all files in long listing format one per line (`-l`) including hidden files (`-a`).

NOTE:
- Use `-1` to list only filenames with one per line
- Use `-t` to list files by modification time, newest first

### `ls -l | wc -l`
Count the number of files in the directory.
NOTE:
- `wc` is to count words or lines or sth,  `-l` is to count number of lines. So `ls -1` also works fine.

#### Ref
- [6 WC Command Examples to Count Number of Lines, Words, Characters in Linux](https://www.tecmint.com/wc-command-examples/)


## Search 
### **find**: search file by filename
#### `find [path] -name '[filename]'`
Find a filename in a path by name.

NOTE:
- Use `-iname` to ignore the case of the file;
- It's always better to quote the string you want to search like `'word'`, since unquoted pattern may contain symbols like `\, |, *` which will influence the command you run. For instance, `find . -name '*.swl'` is to find all swl files in current directory.

### **locate**: search file by filename fast
#### `locate '[filename]'`
Find all paths of the file by name.

NOTE:
- `locate` searches files in a database. Run **`sudo updatedb`** to update the database to the latest at first before using `locate`.

### **grep**: search text in some file(s)
#### `grep <-option> '[word]' '[filename]'`
Search a word in files.

Options:
- `-c` to count the number of words found in the file.
- `-i` to ignore the case (insensitive to case);
- '-E' to interpret pattern as extended regular expression involved with **meta-characters** like `|, (, ), ?`, etc (See the reference below for details), which is very useful in searching things by patterns.
- `-F` to treat pattern as fixed string (all special meanings of characters are ignored)

Here is an example of showing all files in current directory in swl or swp format. All the following commands work exactly the same. 
- `ls -l | grep '\.swl\|\.swp'`: Here `|` is a meta-character which is regarded as a normal character without its special meaning in the **basic regular expression**, and backslashed version will get the special meaning back.
- `ls -l | grep -E '\.swl|\.swp'`: In extended regular expression, `|` is a meta-character for alternative.
- `ls -l | grep -E '\.sw(l|p)'`: use `(,),|` to include the alternative expression easier.

NOTE:
- The backslash `\` before the dot is to **escape** the dot symbol which is used as **matching a single character** by default. If you don't use the backslash, like `.swl`, it will be treated as any single character followed by "swl" (the dot will be ignored).  Of course you can also use `-F` to ignore all special meanings of characters, but so will be the `|`.
- `grep -E` is equal to `egrep`.
- `grep` involves regular expressions a lot. Refer to relevant materials in the **Regular Expression** section.

#### Ref
- [Grep and regular expressions](https://ryanstutorials.net/linuxtutorial/grep.php)
- [How To Use Find and Locate to Search for Files on a Linux VPS](https://www.digitalocean.com/community/tutorials/how-to-use-find-and-locate-to-search-for-files-on-a-linux-vps)
- [Search Multiple Words / String Pattern Using grep Command on Bash shell](https://www.cyberciti.biz/faq/searching-multiple-words-string-using-grep/)
- [Special Meta-Characters of grep](https://www.tecmint.com/difference-between-grep-egrep-and-fgrep-in-linux/)
- [(Official GNU3) Meta character in Regular expression and Extended regular expression](http://www.gnu.org/software/grep/manual/html_node/Basic-vs-Extended.html)

## Regular Expression
### Special characters
- . (dot) - a single character.
- ? - the preceding character matches 0 or 1 times only.
- * - the preceding character matches 0 or more times.
- + - the preceding character matches 1 or more times.
- {n} - the preceding character matches exactly n times.
- {n,m} - the preceding character matches at least n times and not more than m times.
- [agd] - the character is one of those included within the square brackets.
- [^agd] - the character is not one of those included within the square brackets.
- [c-f] - the dash within the square brackets operates as a range. In this case it means either the letters c, d, e or f.
- () - allows us to group several characters to behave as one.
- | (pipe symbol) - the logical OR operation.
- ^ - matches the beginning of the line.
- $ - matches the end of the line.


#### Ref
- [Grep and regular expressions](https://ryanstutorials.net/linuxtutorial/grep.php)
- [Regex Tutorial For Linux (Sed & AWK) Examples](https://likegeeks.com/regex-tutorial-linux/)
- [(Official GNU3) Regular Expression](http://www.gnu.org/software/grep/manual/html_node/Regular-Expressions.html#Regular-Expressions)
- [Special Meta-Characters of grep](https://www.tecmint.com/difference-between-grep-egrep-and-fgrep-in-linux/)

## Compression
```
zip -r [output.zip] [folder]
zip [output.zip] [filename]
```


## Decompression
```
tar xvjf *.tar.bz2 <-C some_directory>
tar xvzf *.tar.gz/tgz
tar xvf *.tar/.tar.xz
gunzip *.gz
unzip *.zip <-d some_directory>
bzip2 <-dk> *.bz2
```
Note:
- In `bzip2`, the option `-d` is decompression, and `-k` is to keep original bz2 file (otherwise it will be deleted by default) 

#### Ref
- [How to Compress and Decompress a .bz2 File in Linux](https://www.tecmint.com/linux-compress-decompress-bz2-files-using-bzip2/)

## Graphics Card

### `sudo lshw -C video/display`
Show existing graphics cards in the computer. Here `video` and `display` are equivalent. 

### `nvidia-smi`
Check the memory usage of Nvidia graphics card. Note it only works after you install nvidia driver successfully.

## Time

### `time <command>`
Show time statistics of running a command.

### Ref:
- [What do 'real', 'user' and 'sys' mean in the output of time(1)?](https://stackoverflow.com/questions/556405/what-do-real-user-and-sys-mean-in-the-output-of-time1/556411#556411)
