# Sublime Text 3 and Atom Collection

## Sublime Text 3

### Important packages

#### SublimeAStyleFormatter

A simple auto-format tool. After installation, use (CTRL-K + CTRL-F) or CTRL-ALT-F to format codes (similiar to Visual Studio). However, it is much less powerful than ClangFormat shown below.

#### Clang Format

C++ code formater based on ClangFormat, for beautify code with minimal effort in Sublime Text 3.

Ref: https://www.jianshu.com/p/5dea6bdbbabb

Default auto format shortcut is CTRL-ALT-A.

The package contains several different style to use. ClangFormat also supports custom format file. You can define the custom style file by setting a `.clang-format` file and put it into your project folder (so it will be project/directory-specific). Note that the file must be named as `.clang-format` exactly (i.e., it is a filename instead of a file format). You can find some simple example style files in the above link, or use the following one for C++ including more custom styles about braces:
```shell
# Example custom .clang-format file.
---
BasedOnStyle: google
IndentWidth: 4
TabWidth: 4
UseTab: Never
ColumnLimit: 120
---
Language: Cpp
Standard: Cpp11
NamespaceIndentation: None
AccessModifierOffset: -4
ContinuationIndentWidth: 4
SortIncludes: false
CompactNamespaces: false
Cpp11BracedListStyle: true
AlwaysBreakTemplateDeclarations: true

# Break after everything except namespaces.
BreakBeforeBraces: Custom
BraceWrapping:
    AfterClass: true
    AfterControlStatement: true
    AfterEnum: true
    AfterFunction: true
    AfterNamespace: false
    AfterStruct: true
    AfterUnion: true
    AfterExternBlock: true
    BeforeCatch: true
    BeforeElse: true
    IndentBraces: false

# Not sure if the following works.
AllowShortFunctionsOnASingleLine: Inline
AllowShortLoopsOnASingleLine: false
AllowShortIfStatementsOnASingleLine: false
AllowShortBlocksOnASingleLine: false
AllowShortCaseLabelsOnASingleLine: false
SpaceBeforeParens: ControlStatements
SpaceAfterTemplateKeyword: true
SpaceBeforeAssignmentOperators: true
SpaceInEmptyParentheses: false
SpacesInAngles: false
SpacesInCStyleCastParentheses: false
AlignAfterOpenBracket: DontAlign
```

To use the custom clang-format file, the simplest way is to save the above contents as a `.clang-format` file and put it just into your code folder. Then the tool will automatically use this format file by default. If you want to use a different file, you can set the path of the format file by *Clang Format: set path*, and then *Clang Format: select style * and choose *file*. Try to format some code and you will see it works then.

#### PackageResourceViewer

View or extract packages by extracting internal files from packages. Very useful if you want to copy and then change something inside an existing package.

#### C++ Starting Kit

C++ code theme. Better than default syntax theme.

#### BracketHighlighter

Bracket and tag highlighter in codes. Highlight all kinds of brackets.

#### CTags

Index code symbols for code naviation. It's very useful for variable and function navigation and will add command *Go/Jump/Navigation to definition* in right clicking menu. After installation, run *ctags: rebuild ctags* and it will build and create a ctags file `.ctags` in project folder.
The shortcut for *Navigation to definition* is (CTRL-T + CTRL-T).

#### MarkdownPreview

Preview markdown file in different styles including Github style.

### Highlight class/struct keywords in C++ codes

https://stackoverflow.com/questions/23145265/sublime-text-c-highlight

Default C/C++ syntax theme is bad that it can hardly detect many types such as custom classes.
We can improve it by installing new package and adding some new definition inside it.
- Firstly install package *C++ Starting Kit*. This new syntax is slightly better than default C++ syntax theme that almost all built-in keywords are detected (like namespace, string, etc). However, one important missing type is still the custom Class/Struct keyword. So the idea is to change some content of existing *C++ Starting Kit* package.
- Create a copy of *C++ Starting Kit*. Firstly install another new package *PackageResourceViewer* which is used to extract package files. After installation, run *PackageResourceViewer: extract packages* to extract package files of *C++ Starting Kit*. It will create a folder inside sublime text as `~/.config/sublime-text3/Packages/C++ Starting Kit`. You can rename the folder as a new one like `CNew` to distinguish between it and original package. Now you can find the new syntax name `CNew` in *View -> Syntax* in sublime text and its functionality now is exactly the same as *C++ Starting Kit*.
- Change the file `~/.config/sublime-text3/Packages/User/Package Control.sublime-settings` by adding the new package name `CNew` inside the category `installed_packages`. If you don't do this, the extracted files will be **deleted automatically after you close sublime text**.
- Now it's time to change syntax content. Open the file `~/.config/sublime-text3/Packages/C++ Starting Kit/C++.tmLanguage` and find the lines similar to the following:
```shell
<!--------------------------------------
    Storage Type - C++
---------------------------------------->
        <dict>
            <key>match</key>
            <string>\b(class|wchar_t|nullptr_t)\b</string>
            <key>name</key>
            <string>storage.type.c++</string>
        </dict>
```
We will make a **copy** of this part and put it underneath, and then change it a little bit as syntax of detecting class/struct keywords like this:
```shell
<!--------------------------------------
    Storage Type - C++ (added)
---------------------------------------->
        <dict>
            <key>match</key>
            <string>\b([A-Z][a-zA-Z0-9]+)\b</string>
            <key>name</key>
            <string>duncan.name.class</string>
        </dict>
```
Here the new variable/syntax tyle is `duncan.name.class`. Now the syntax change is done.
- Last step is to change the corresponding color theme. Use *PackageResourceViewer* again to open a color theme file by running Run *PackageResourceViewer: open packages -> Color theme* and choose one theme file you want to open and edit. Then the theme file will be put in `~/.config/sublime-text3/Packages/Color Scheme - Default` folder. Make a copy of the theme file like `Monokai-new.sublime-color-scheme`. Then open it and find lines about any arbitrary rule definition, such as rule for string like this:
```
{
    "name": "String",
    "scope": "string",
    "foreground": "var(yellow)"
},
```
Again, make a **copy** underneath and change it like this:
```
{
    "name": "Duncan",
    "scope": "duncan.name.class",
    "foreground": "var(blue)"
},
```
Here the `scope` name must be the same of the type as you defined in the syntax file. Use any color you like. Now the new color theme with supported syntax is finished. Apply this new color theme and then you can find the newly detected class/struct keywords.

## Atom

Background:
- Atom loads configuration files located in `~/.atom/` root path, such as **keymap.cson** (for keybinding), **config.cson** (global settings), **styles.less** (editor code styles), **packages** (package folder), etc.
- Only need to remember one shortcut: open the menu by `CTRL-SHIFT-p` (same as in Sublime Text). Type in anything you want to search, such as installing a package, open settings, etc.

### Change the color of comments

Add this in your `styles.less` file:
```shell
atom-text-editor.editor .syntax--comment {
    color: #ffffaa; // light yellow
}
```

### Highlight all instances of selected content

Install package `highlight-selected`.

### Go to last/next cursor position

https://atom.io/packages/last-cursor-position

Need to install package `last-cursor-position`. Default is `ALT--` to go last and `ALT-_` to go next. Change it if you want in your `keymap.cson` file like this
```shell
'atom-workspace':
  'alt-=': 'last-cursor-position:next'
```

### Use CTags/Symbol-gen for code navigation

Ref:
- https://github.com/atom/symbols-view/issues/9
- https://atom.io/packages/symbol-gen

Atom contains its own **Go to definition** tool to jump to definitions or not. However, it still needs to utilize the CTags, which is a tag/index generation tool for various programming languages. To use it, a simple way is to install the package `symbol-gen`, and use shortcut `CTRL-ALT-g` to generate the ctags file for your project. Then, enjoy 'Go to definition' smoothly. Of course, you can manually create it by running command `ctags -R .` inside your project directory.

### Auto-format

Use package `atom-beautify`. Besides this, you still need to install default beautifiers **uncrustify** or **clang-format** (simply by `sudo apt-get install`). Choose one in the settings of atom-beautify (while clang-format is recommended). The default shortcut to use atom-beautify is CTRL-ALT-B, but sometimes it is already occupied by some other function. To change it to some other binding (like CTRL-ALT-f), add this in your `keymap.cson` file in the atom root path:
```shell
'.editor':
  'ctrl-alt-f': 'atom-beautify:beautify-editor'
```

#### Use clang-format (recommended)

Ref: https://www.jianshu.com/p/5dea6bdbbabb

See descriptions in Clang Format usage in sublime text above.

#### Use uncrustify

Ref: https://github.com/Glavin001/atom-beautify/issues/2012

You may meet the problem that the default tab size is too large in uncrustify. If you want to change the tab width from 8(Unix default) to 4:
- Open the file `~/.atom/packages/atom-beautify/src/beautifiers/uncrustify/default.cfg` (You can find deafult.cfg by going to atom-beautify settings and then click view code);
- Change these lines:
```
output_tab_size = 4 # (edit about line 14)
indent_columns = 4 # (edit about line 42)
```
and comment out the line:
```shell
# align_number_left = false    # false/true (about line 720)
```
- Change path of the C and C++ setting of atom-beautify as the file `~/.atom/packages/atom-beautify/src/beautifiers/uncrustify/default.cfg` in the package settings of atom-beautify.


### Custom keybinding

Ref: https://stackoverflow.com/questions/33023349/atom-disable-single-key-binding

To change a default keybinding, firstly find the keybinding in settings, and copy and paste it to your keymap file `keymap.cson`. Note that if you simply want to disable some keybinding, set its command as 'unset':
```shell
'.editor':
  'ctrl-up': 'unset'
```

### Open a new file in a tab instead of new instance window

https://superuser.com/questions/1389397/how-to-open-a-new-file-in-a-another-tab-instead-of-a-new-window-in-atom-editor)


Command `atom <file>` will open a file in a new atom window each time instead of opening a new tab in existing atom window, while for the latter you have to use `atom -a <file>` . A little annoying since you cannot set it globally by default.


### Use panes

https://flight-manual.atom.io/using-atom/sections/panes/

Open a pane: CTRL + K, then UP/DOWN/LEFT/RIGHT to open a pane in different place;
Close current pane: CTRL + W
