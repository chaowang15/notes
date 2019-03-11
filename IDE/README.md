# Sublime Text 3 and Atom

## Sublime Text 3


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

You can define the custom style of clang-format by setting a `.clang-format` file and put it into your project folder (so it will be project/directory-specific). Note that the file must be named as `.clang-format` exactly (i.e., it is a filename instead of a file format). You can find some simple example style files in the above link, or use the following one for C++ including more custom styles about braces:
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
