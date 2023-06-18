## vnc server on ubuntu

### tigervnc
```
sudo apt-get install tigervnc-standalone-server tigervnc-xorg-extension
vncpasswd
```

### ~/.vnc/xstartup
```
#!/bin/sh
# Start Gnome 3 Desktop 
[ -x /etc/vnc/xstartup ] && exec /etc/vnc/xstartup
[ -r $HOME/.Xresources ] && xrdb $HOME/.Xresources
vncconfig -iconic &
dbus-launch --exit-with-session gnome-session &
```

### start vnc server
```
vncserver
```

### list 
```
vncserver -list
```

### kill
```
vncserver -kill :${DISPLAY_INDEX}

ex) vncserver -kill :1
```
