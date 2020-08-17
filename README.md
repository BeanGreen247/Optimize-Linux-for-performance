# Optimize Linux for performance

The Instructions here work on Debian based systems (like Ubuntu) and Arch Linux, but should also work for any other system.

#### The Basics (aka things you should have done already without me telling you because they're obvious duh)

install some proper graphics drivers it gives you an huge performance boost

for 64bit users please note that depending on your Distro you might have also to install the multilib GPU drivers (Ubuntu users should be fine I guess never did looked it up though))

use an fast window manager like openbox or i3-wm

#### Safe stuff to do.

every time you want to run an app you can now disable pulseaudio (and re enable it afterwards) (run as normal user)

```
#disable pulseaudio
pulseaudio -k
#enable pulseaudio
pulseaudio -D
```

(I would recommend using a script for that)

the next thing to do is to give alsa a higher priority

edit your "/etc/security/limits.conf" so it looks like that (for memlock use your amount of ram or "unlimited)

```
@audio - rtprio 99
@audio - memlock 512000
@audio - nice -19
```

->according to [this](https://gamesplusone.com/alsa_to_jack.html) (which I found through the Sound FAQ of the official WineHQ page) alsa can adjust to jackd2's buffer size by using a bridge, that means it is possible to get a audio latency of a few milliseconds.

=>Doing it the hard way would be to follow the guide mentioned above.

=>Or you could just use some tools that were created by the same guy [installation instructions](https://kx.studio/Repositories)

==>You only need the Cadence package the rest is only interesting for musicians.

==>Arch users find links to the AUR in the Software>Applications tab on the site

==>you will have to modprobe snd-aloop to make it work but this will be covered later in the tut

==>the actual minimal latency depends on your audio card (for me it's 128 -> 2.7ms (but you should start with 1024 or 512))

->many of you have some fancy graphic tablets which are some pretty good and fast input devices, only problem is that your system doesn't use them with the maximum refresh rate so lets fix that (that also counts for your mouse)

=>edit your "/etc/modprobe.d/modprobe.conf" 2=500HZ=2ms latency (you can also use 1 but normally 2 should be more than enough)

```
options usbhid mousepoll=2
```

=>if you use an USB Audio Card like I do also add:

```
options snd-usb-audio nrpacks=1
```

=>to enable this option (should be enabled by default, but just to make sure) edit "/etc/modules-load.d/lowerinput.conf" (you only need the second line if you enabled the snd-usb-audio option in the previous step and the snd-aloop if you want to use the alsa-jack bridge)

```
usbhid
snd-usb-audio
snd-aloop
```

this is it for the save stuff.

#### Advanced Modifications 

(these options are advanced changes that can potentially break your system (only if you are doing something wrong) or make it hang up together with the app (hasn't happened to me yet), it is advised that you know what you are doing or can at least pretend to know ;) )

These options are also only tested on Manjaro Linux (means that all commands and configuration locations used in here are the Arch standard ones) but should mostly apply to all the other systems.

->for the following steps you might want to optimize your compiler flags first

->everyone is always talking about how great lowlatency and rt kernel's are (well yes they are great) but they are nothing compared to the -ck patched kernel it really gave my in and output a big boost

you can get it if you are using fbdecondecor for your splashscreen

you also should read the Arch Wiki entry, I recommend enabling the BFQ and there are a few other settings you might want to change.

->increase the priority level so that you can actually use all the latency improvements to the fullest and minimize random framedrops:
=>edit "/etc/security/limits.d/91-nice.conf"

```
usernamehere           -   nice            -4
```

=>add the following in front of your start command (if you enabled BFQ in the last step you only use the nice command without the ionice)

```
ionice -c2 nice -n -4
```

#### Drive optimizations

Set swappiness

```
sudo sysctl vm.swappiness=10
```

```
sudo nano /etc/sysctl.conf
```

type in this

```
vm.swappiness=10
```

save and exit

Disable fsck after startup
To disable fsck after startup do the following

```
sudo nano /etc/fstab
```

And change the zero at the end of the root ("/") partition and home partition

```
# <file system>             <mount point>  <type>  <options>  <dump>  <pass>
UUID=1a8a0568-d1fd-403e-891b-cbc5f5c41e3d /              ext4    defaults,noatime 0 0
UUID=023fead2-f3f0-4431-93c3-0390b90e1aed /home          ext4    defaults,noatime 0 0
```

#### Is this it?

Well yes at least for now I might give you an update with more stuff in a few weeks.
I'm currently experimenting with wine-asio, driconf, general GPU driver configuration and some other small stuff like better CPU on demand scaling.

For all the people who are wondering if there is actually any practical room for more improvements let me tell you this:
I can get a Audio latency of 0.7ms (with xruns from time to time)/1.3ms(without xruns) by using mplayer+.opus file(opus itself has an minimal latency of 5ms so I'm talking about the actual server latency that is possible with the setup) hooked up directly to jack, and that's just for audio

Think I forgot something? Then please let me know!
I'm always happy about feedback.

Sources 

**give some credit to the people that deserve it (also so you can read up on all the stuff I posted above)**

[linuxaudio-latency guide](https://wiki.linuxaudio.org/wiki/system_configuration)

[Mouse Polling Rate](https://wiki.archlinux.org/index.php/Mouse_polling_rate)

[linux-ck](http://ck-hack.blogspot.com/)

[reduce-buffer-size](https://bbs.archlinux.org/viewtopic.php?pid=1002264#p1002264)

[Wine Developer's Guide/Kernel modules](https://wiki.winehq.org/Wine_Developer%27s_Guide/Kernel_modules)

the linux manpages

## Videos for reference

[Optimize Linux for performance](https://www.youtube.com/watch?v=nqTftr0nWEU) (main vid)

[Improve your HDD performance in Linux!](https://www.youtube.com/watch?v=rDyXzD8e310) (part 2)
