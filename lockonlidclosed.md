

In the meen time I switched to Manjaro (much nicer), but this solution should work on both:

Creating a service called lock@.service in /etc/systemd/system with this content:

[Unit]
Description=i3lock on suspend
After=sleep.target

[Service]
User=%i
Type=forking
Environment=DISPLAY=:0
ExecStart=/usr/bin/locker

[Install]
WantedBy=sleep.target

Makeing it executable

chmod +x lock@.service

Then enableing it for your user

systemctl enable lock@<username>.service

should do the trick.

Note that the %i in User=%i will be replaced with the . The "/usr/bin/locker" could just be /usr/bin/i3lock but i have some fancy stuff added there to make it look good.

Hope this helps some body at some point
