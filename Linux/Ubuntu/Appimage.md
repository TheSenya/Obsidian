### Making the image executable
```bash
chmod +x your-app.AppImage
./your-app.AppImage
```

## Creating a desktop icon

create a desktop file
```bash
nano ~/.local/share/applications/your_app_name.desktop
```
add this to the desktop file
```bash
[Desktop Entry]
Name=Your Application
Name Exec=/path/to/your/appimage.AppImage
Icon=/path/to/your/icon.png
Type=Application
Categories=Utility;
Terminal=false
```
add the icon and appimage file into Applications folder (you created this)
```bash
~/home/Applications/
```
