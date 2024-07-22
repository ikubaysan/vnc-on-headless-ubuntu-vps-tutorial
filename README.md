# vnc-on-headless-vps-tutorial


I opted for a Ubuntu 24.04 VPS. Should work for other versions of Ubuntu, and other debian-based distros.


This command combines two commands:

* `apt update` updates the list of available packages and their versions, but it does not install or upgrade any packages.

* `apt upgrade` installs newer versions of the packages you have. 

* The `-y` flag is used to automatically answer "yes" to all confirmation prompts.

```shell
sudo apt update && sudo apt upgrade -y
```


Install required packages:
```shell
 sudo apt install ubuntu-mate-desktop x11vnc xvfb net-tools dbus-x11
```

Set up a password for the VNC server:
```shell
x11vnc -storepasswd
```


Xvfb is a display server that can run on machines with no display hardware and no physical input devices. 
It emulates a framebuffer using virtual memory. We use it to host a virtual display for the VNC server.

We need to create a script that starts Xvfb, sets the DISPLAY environment variable, 
starts the desktop environment, and then starts the VNC server.

```shell
 nano ~/start_vnc.sh
```

You will now be in the nano text editor. Paste the following code into the file:

```bash
# Start the X Virtual Framebuffer (Xvfb) on display number :1 with screen 0
# The resolution is set to 1280x720 pixels with a color depth of 16 bits per pixel.
# Xvfb allows us to run graphical applications without a physical display.
Xvfb :1 -screen 0 1280x720x16 &

# Export the DISPLAY environment variable to :1
# This tells applications to use the Xvfb display we just started.
export DISPLAY=:1

# Start the Mate desktop environment session
# This will initialize the Mate desktop environment on the virtual display.
mate-session &

# Start x11vnc to provide remote access to the Xvfb display
# -display :1 specifies the X display to connect to (the one started by Xvfb).
# -forever keeps the server running even after the client disconnects.
# -loop makes x11vnc restart if it crashes.
# -repeat allows key and mouse events to be repeated if held down.
# -shared allows multiple clients to connect simultaneously.
x11vnc -display :1 -forever -loop -repeat -shared -rfbauth ~/.vnc/passwd
```

Save the file with `Ctrl+o`, and then `enter`. Then press `Ctrl+x` to exit nano.

Set the script as executable:

```shell
chmod +x ~/start_vnc.sh
```

Now test the script by running it:

```shell
~/start_vnc.sh
```

You can now use SSH tunneling to connect to the VNC server. 
Open a terminal/command prompt and enter the following, replacing the username and IP address with the VPS', 
and "5910" with whichever port you want to connect to on your local machine.
Normally this would be 5900, but here we're using 5910 to avoid conflicts with local VNC servers you may have running.

```shell
ssh -L 5910:localhost:5900 ubuntu@12.34.56.78
```

Connect to the VNC server using a VNC client and confirm it works.
`localhost:5910`


Now we'll create a service file to automatically start the VNC server on boot, and be able to start/stop it easily.
Open the service file for editing:

```shell
sudo nano /etc/systemd/system/vncserver.service
```


Paste the following code into the file:

```shell
[Unit]
Description=Start x11vnc at startup

[Service]
# Set the path to the bash script that starts the VNC server
ExecStart=/home/ubuntu/start_vnc.sh
# Set the user this script should run as
User=ubuntu
# Restart the service if it crashes
Restart=on-failure
```

Save the file with `Ctrl+o`, and then `enter`. Then press `Ctrl+x` to exit nano.

Reload the systemd daemon so the new service file is recognized:

```shell
sudo systemctl daemon-reload
```

Enable the service to start on boot:

```shell
sudo systemctl enable vncserver.service
```

Start the service:

```shell
sudo systemctl start vncserver.service
```

Check the status of the service to ensure it's running:

```shell
sudo systemctl status vncserver.service
```

You should see output indicating that the service is active and running.

You can now start/stop the VNC server using the following commands:

```shell
sudo systemctl start vncserver.service
sudo systemctl stop vncserver.service
```
