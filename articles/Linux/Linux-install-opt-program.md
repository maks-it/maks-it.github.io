# How to install binary probram in opt folder

We take an example from Postman tar.gz package

```bash
wget https://dl.pstmn.io/download/latest/linux64 -O postman-linux-x64.tar.gz
```

Extract to /opt

```bash
sudo tar -xzf postman-linux-x64.tar.gz -C /opt
```
Symbolic link to the install in /opt from you executables directory

```bash
sudo ln -s /opt/Postman/Postman /usr/bin/postman
```

Lastly create the launcher icon

```bash
nano ~/.local/share/applications/Postman.desktop
```

Paste the following in the new file

[Desktop Entry]
Encoding=UTF-8
Name=Postman
Exec=/opt/Postman/app/Postman
Icon=/opt/Postman/app/resources/app/assets/icon.png
Terminal=false
Type=Application
Categories=Development;
