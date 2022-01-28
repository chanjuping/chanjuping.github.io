Adding a post online just to keep this blog alive. I can always comeback to make corrections later.

Install syncthing

> sudo apt install syncthing

Run the syncthing systemd service.

> sudo systemctl start syncthing@$USER

Stop the service.

> sudo systemctl stop syncthing@$USER

Open config at `~/.config/syncthing/config.xml` to get the device id, and to change the GUI web address from:

>	<gui enabled="true" tls="false" debugging="false">
>		<address>127.0.0.1:8384</address>

to

>       <gui enabled="true" tls="false" debugging="false">
>               <address>0.0.0.0:8384</address>

This change allows opens up the server to allow connections from all other devices on the same network.

Start syncthing again

> sudo systemctl start syncthing@$USER

Then connect to the server on your local network to complete the setup. Set up a username and password to prevent potential fiddling from others who are sharing the local network.
