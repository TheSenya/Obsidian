
## Creating a Webapp
create a 'Gemini.desktop' file
```toml
[Desktop Entry]
Version=1.0
Type=Application
Name=Gemini
Comment=Access Google's Gemini AI
Exec=brave-browser --app=https://gemini.google.com/
Icon=brave-browser
Terminal=false
StartupWMClass=gemini.google.com
```

save this file in 
```
~/.local/share/applications/
```

allow desktop file to be executable
```bash
chmod +x ~/.local/share/applications/gemini.desktop
```

restart desktop shell
- **GNOME:** Press `Alt + F2`, then type `r` and press `Enter`.
    
- **KDE Plasma:** The command is `kwin_x11 --replace & disown`.
    
- **Cinnamon:** The command is `cinnamon --replace`.
    
- **LXDE:** The command is `lxpanelctl restart`.

update the desktop database
```
update-desktop-database
```

To make the icon show up everywhere and make everything look nice
Launch the desktop app
open terminal and type
```
xprop WM_CLASS
```
click on the desktop appliocation that is currently running 
copy that name and enter it under the field in the .desktop file
```
StartupWMClass=obsidian
```
