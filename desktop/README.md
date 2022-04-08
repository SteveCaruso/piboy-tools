# Setting up the PiBoy as a Desktop system:

The PiBoy is not only an awesome retro-gaming system that is incredibly hackable and customizable, but since it runs on a Raspberry Pi it can also be used as a decent portal personal computer system with just a few modifications.

Much like high-end cases like the PiTop, the PiBoy brings a display, a speaker, a set of buttons, a joystick, and a battery to the party – and these are things we can take advantage of.

This article assumes you're using the following:

- A Raspberry PI 4
- The official PiBoy disk image (v1.0.7)
- Firmware version 1.0.5
- That your username for your admin account is "pi" (otherwise, you'll need to change wherever you see "pi" to your username).

## Step 1: Make Sure You're Connected to the Internet

If you've already set up wifi, you can skip this step, but if you haven't:

Navigate to the RetroPie menu and select Wifi.

Select option 1 to Connect to Wifi Network and follow the prompts.

## Step 2: Install Pixel

This is stupidly easy.

In the RetroPie menu, go to RetroPie Setup.

Select `(C) Configutation / tools`

Select `(219) raspbian tools`

Select `(1) Update Raspbian packages`

When that's done, select `(2) Install Pixel Desktop`

Once that finished up, congrats! You now have Pixel installed.

## Step 3: Launch Pixel & Configure

Go to the `Ports` menu and select `Desktop` to launch Pixel.

You'll need a keyboard and mouse for this next part. A good portable keyboard/touchpad combo with a USB dongle works extremely well.

Once Pixel is up and running, go to the `Raspberry Menu > Preferences > Raspberry Pi Configuration`

In the `System` tab:

- Change your password immediately or you *will* be hacked.

In the `Localization` tab we need to tell it where we are, or a number of things won't work properly:

- Set your Locale
- Set your Timezone
- Set your Keyboard
- Set your WiFi Country

Say OK, and if it prompts you to reboot, don't do it *yet*.

## Step 4: Configure the Overlay

Now you're probably noticing that the overlays on the screen are getting in the way of using the menubar. 

You can – if you want – simply move the menubar to the bottom of the screen by going to the `Raspberry menu > Preferences > Appearance Settings > Menu Bar` tab and setting it there.

But if you're like me and have gotten used to the menubar being on top, we're going to want to customize the overlay so it doesn't interfere.

Open a new Terminal window and type in:

```
sudo pico /boot/osd.cfg
```

This file allows us to tinker with a bunch of bits of the PiBoy's hardware. I recommend changing the following:

Change the `redled` and `greenled` maximum brightness to 3 or 5 (this reduces bleed through the plastic).

Completely comment out the paramters for the `statistics`, `temperature`, `voltage`, `current`, `cpu`, `bluetooth`, and `wifi`. Most of this stuff we can get directly from Pixel's display menus as it is.

Set the `throttle` size to `small` so we can see if and when we're throttling.

It's the `battery` level is really the most important thing we need to see that Pixel cannot provide. Set the battery x coordinate to 625 and its size to small. This will place it to the right-hand side of the system clock in the upper-right-hand corner when using the onboard display, and tastefully in the center of the menubar when using the HDMI at 720p.

However, feel free to tinker around to see what works for you.

When you're done, hit `[control]-X`, type `Y` for yes, and then hit `[return]` to save.

You will also likely want to set the menubar color a bit darker (I use #CCCCCC) so that it lines up with the battery indicator and is dark enough so you can see it. But again, that's up to your taste.

## Step 5: Remove PulseAudio

And this is the final bit before we need to reboot for the first time. 

When you installed Pixel (or any other Pi desktop system for that matter) it also installs `PulseAudio`, an audio mixing package.

This system doesn't play nice with RetroPie when you make use of the HDMI connection. When you plug things in, it essentially freezes the volume where it's at, and will only play sound through the PiBoy's onboard speaker – essentially killing the experience with external displays.

No matter, though, as we can get rid of it. Open a Terminal window and type in the following command:

```
sudo apt-get remove --purge pulseaudio
```

And then reboot.

And now you should have a good baseline PiBoy desktop environment experience. :-)

But we're not done yet.


## Step 6: Lock Down Samba

Samba is a means of transferring and sharing files to and from the Raspberry Pi. It's very convenient, but it's enabled by default and PiBoy's default is to enable guest access.

This means that Samba is *wide open*. This is bad. We need to secure it.

First make sure that your password on Samba matches your changed password for your account:

```
sudo smbpasswd -a pi
```

Then change all of the Samba shares (roms, bios, configs, and splashscreens) so that the following are set:

```
guest ok = no
```

and

```
valid users = pi
```

Then press `[ctl]-x` to exit, `Y` to save changes, and `[enter]` to write them.

Then finally, reboot samba:

```
sudo systemctl restart smbd smbd
```

Whew... now that's secure.

## Step 7: Install QJoypad

It'd be really convenient to use the joystick as a mouse in the desktop environment. QJoypad will allow us to do this.

Let's get it working, and then set it up so that it auto starts when we log into Pixel.

First, we install it:

```
sudo apt-get install qjoypad
```

And then we open it (it should be in `Raspberry Menu > Games`) and configue it for the PiBoy. You can figure out which buttons are which by [using the handy chart that Experimental Pi has on their website](https://resources.experimentalpi.com/mapping-the-piboys-buttons-in-retropie/).

But for reference here are the button mappings:

0) A
1) B
2) C
3) X
4) Y
5) Z
6) Right Shoulder
7) Left Shoulder
8) Start
9) Select
10) Pressing the joystick in
11) D-pad Down
12) D-pad Up
13) D-Pad Left
14) D-Pad Right

And for the joystick:

- Axis -0 : Joystick Left
- Axis +0 : Joystick Right
- Axis -1 : Joystick Up
- Axis +1 : Joystick Down

Click `Add` to create a new mapping, and call it `PiBoy`.

Set up the joystick as the mouse by setting `Axis 1` and `Axis 2` from `Keyboard` to the appropriate `Mouse` direction. Check the `Gradient` box, and reduce the `Mouse Speed` down to 20, or it's too zippy.

Next set buttons for left and right click (A and B are good for those) as well as buttons for return, backspace, and escape (X, Y and Z work well – they'll come in handy).

I also recommend that you set up the D-pad for the arrow keys, which makes scrolling much easier.

Once you're done, be sure to hit the `Update` button, or your changes won't save.

So now once all that's set, we're good to go... provided that QJoypad is launched. If it's not running, none of this will work.

Open a terminal window and type in the following:

```
sudo pico /home/pi/.config/autostart/QJoyPad.desktop
```

Inside the text editor, type in the following:

```
[Desktop Entry]
Name=QJoyPad
Exec=qjoypad "PiBoy"
Type=Application
```

Then press `[ctl]-x` to exit, `Y` to save changes, and `[enter]` to write them.

What you've just done was to initialize the `autostart` directory in your `.config` folder and put a Desktop icon linked to qjoypad into it. Any applications or Desktop icons you put in the autostart folder are – as the name suggests – automatically started when you log in.

Now when you log into Pixel, qjoypad should be ready to use and running in the background. 

**NOTE:** Do remember, though, that if you open the QJoyPad configuration panel from this point forward, you'll need to plug in a keyboard and mouse. When the panel is open, until it's closed the PiBoy's controls are frozen out.

## Step 8: Install Matchbox Keyboard

And now with the joypad installed, we can also install an on-screen keyboard, so you can – if you absolutely have to – run the PiBoy as a desktop system with just the PiBoy controls themselves.

This portion is adapted from https://pimylifeup.com/raspberry-pi-on-screen-keyboard/

First let's install Matchbox Keyboard:

```
sudo apt install matchbox-keyboard
```

Next, now that it's installed, let's build a quick script that will allow us to toggle the keyboard on and off.

```
sudo nano /home/pi/.config/my-scripts/toggle-keyboard.sh
``` 

This creates a new shell script in your .config folder. Let's put the following into it:

```
#!/bin/bash
PID="$(pidof matchbox-keyboard)"
if [  "$PID" != ""  ]; then
  kill $PID
else
 matchbox-keyboard &
fi
```

What this script does is that it grabs the process ID of matchbox-keyboard. If the process ID *exists* it means that matchbox-keyboard is running, so we kill that process ID to shut it down. If it's *blank* it means matchbox-keyboard isn't running, so we start it up and keep it going in the background (that's what the & means).

So it's a simple toggle. If it's off, running this turns it on. If it's on it switches it off.

Next, we are going to put a button in the menubar that will fire this script off when clicked. Similarly to qjoypad, we need to create a .desktop file and put it in the proper place.

So type the following:

```
sudo nano /usr/share/raspi-ui-overrides/applications/toggle-keyboard.desktop
```

And type in the following

```
[Desktop Entry]
Name=Toggle Virtual Keyboard
Comment=Toggle Virtual Keyboard
Exec=/home/pi/.config/my-scripts/toggle-keyboard.sh
Type=Application
Icon=matchbox-keyboard.png
Categories=Panel;Utility;MB
X-MB-INPUT-MECHANISM=True
```

Then press `[ctl]-x` to exit, `Y` to save changes, and `[enter]` to write them.

Next, in order to get our hands on the menubar, itself, we need to make a local copy in a specific place that we can override. The following command copies the defaults into your `.config` folder so we can do just that.

```
cp /etc/xdg/lxpanel/LXDE-pi/panels/panel /home/pi/.config/lxpanel/LXDE-pi/panels/panel
```

And now with that local copy made, we need to install the .desktop file into it. So first, edit the panel file with this:

```
nano /home/pi/.config/lxpanel/LXDE-pi/panels/panel
```

Scroll to the *very bottom* of it and add the following:

```
Plugin {
  type=launchbar
  Config {
    Button {
      id=toggle-keyboard.desktop
    }
  }
}
```
Press `[ctl]-x` to exit, `Y` to save changes, and `[enter]` to write them.

Then reboot.

You're all set.
