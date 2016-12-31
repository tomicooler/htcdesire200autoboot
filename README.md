==================================================================
HTC Desire 200 - auto boot when charger is plugged in
==================================================================

What's this?
------------

I have made a car alarm system for android - tATA Protector.
I have an android phone in my car hidden somewhere, I use it for GPS tracker,
location based car alarm system, "engine start" detection, etc.
The phone is connected to a charger, when the car's engine is running the phone is being charged.

If I don't use my car for a while the phone will discharge and power off. So I needed a solution
to automatically power on my phone when it's being charged.


Further info, apps:
 * https://play.google.com/store/apps/details?id=com.tomicooler.tata.protector
 * https://play.google.com/store/apps/details?id=com.tomicooler.tata.watchergcm
 * https://play.google.com/store/apps/details?id=com.tomicooler.tata.watchersms

Rooting
-------

Open your bootloader: http://www.htcdev.com/bootloader/

Root your phone: http://forum.xda-developers.com/showthread.php?t=1886460

	(3) New Standard-Root (thx Ariel Berkman)


Auto boot
---------

I got it working - auto boot when charger is plugged in - on my HTC Desire 200. There is no *playlpm/lpm/idot/chargemon* on these HTC devices. Fortunately there is something similar, **/system/bin/zchgd**.
This is a service that takes care for the charging animation. On this specific device, the charging animation is played even though the phone is powered on (on the lockscreen), it is also played when the phone is powered off and it's on a charger. (BTW: the images for the animation can be found at */system/media/zchrgd*.)

As I found out, the **zchgd** service is started differently when the phone is powered off or on. I just grep through the init scripts, in my case **/init.gtou.rc**:

    service zchgd_offmode /system/bin/zchgd -pseudooffmode
        user root
        group root graphics
        disabled
    
    service zchgd_onmode /system/bin/zchgd -onmode
        user root
        group root graphics
    
    on property:dev.zcharge=true
        start zchgd_onmode
    
    on property:dev.zcharge=false
        stop zchgd_onmode


So, I created a fake **zchgd** service, a simple *sh* script that replaces the original service.

Steps:

    adb shell
    su
    mount -o rw,remount,rw /system
    cd /system/bin
    ls -l zchgd*
    mv zchgd zchgd_original
    touch zchgd
    vi zchgd # <- insert content, see zchgd service
    chmod 0755 zchgd
    chown root.shell zchgd
    ls -l zchgd*


When the service is stared with *-pseudooffmode* argument, then it checks the battery status periodically
(every minute in my case). If the phone is being charged or the battery is full then it will automatically
restart the phone. I could not use **/system/bin/reboot**, I got permission errors, even though I had them, I'm pretty sure about that :). So I restart it with a hack, that two echo lines.
The *-onmode* is there only because, I don't want to see the charging animation even if my phone is powered on.

You can debug the script with **adb logcat -s zchgdwrapper**. I hope this was helpful!

IMPORTANT
---------

For this to work, you **MUST** enable **Settings/Power/Fast boot** option. It won't work if it is disabled, I think without fast boot another firmware is booted, but not sure about that.

Also I have this config in **SuperSU#**, I don't think it is necessary but, here it is:

 * Default Acces < Grant
 * Enable su during boot
 * Trust system user