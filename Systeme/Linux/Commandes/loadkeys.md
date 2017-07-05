Loadkeys
========

loadkeys fr

The default console keymap is US. To list available layouts, run ls /usr/share/kbd/keymaps/**/*.map.gz.

To modify the layout, append a file name to loadkeys(1), omitting path and file extension. For example, run loadkeys de-latin1 to set a German keyboard layout.

Console fonts are located in /usr/share/kbd/consolefonts/ and can likewise be set with setfont(8). 