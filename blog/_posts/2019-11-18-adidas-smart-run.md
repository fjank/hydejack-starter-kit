---
layout: post
title: Adidas Smart Run - Information Gathering
description: >
  Adidas smart Run is a fantastic training watch, but Adidas elected to shut down their online
  service, and left the buyers of the watch with absolutely nothing. At the moment of service
  shutdown, all the watches got effectively bricked, not possible to update, not possible to 
  upload new workouts, not possible to extract existing workouts.
  This blog entry is the first in a series with my attempts to make it possible to use your own
  server for the watch to talk to.
---
# Adidas Smart Run
![Adidas smart run outside stylistic](/assets/img/blog/adidas_smart_run_outside.jpg)
## Introduction
  For a very good review of the watch and all it has to offer, check out [DC Rainmakers blog entry](https://www.dcrainmaker.com/2013/12/adidas-smart-review.html).

  There was quite a few complaints from customers that bought the watch, but only a few
  got their money back.  
  [Keep alive Adidas Smart Run - 400 $ not to be thrown away](https://www.change.org/p/adidas-keep-alive-adidas-smart-run-400-not-to-be-thrown-away)  
  [miCoach End of Service](https://help.runtastic.com/hc/en-us/articles/360000371649-miCoach-End-of-Service)  
  Now, I want to tackle the challenge of actually getting both data out and in from the
  watch, and make it usable for the unlucky owners of this little gem. This first blog entry will
  be about the information gathering phase.

## Strategy
  I'm going to try to get access very methodically, following the below path, and updating this
  post as I go further. Hopefully I will not need to reach the last step. 
  Main goal is to root the watch, to be able to figure out how the communication to the now 
  non-existing micoach server works, and hopefully be able to put in a mock server of my own.
  I will not attempt to replace or update the firmware, because the firmware is very good
  as it is.
  
  * Information gathering  
    Get as much information as possible from the watch, online services, documentation etc.
  * UI access to internals (recovery/development mode/USB debugging)  
    If development mode is possible to enter from the UI, we are golden.
  * Network access  
    Check out all ports and all networkactivity by the watch when connected to Wifi.
  * USB access  
    Try different techniques via the USB port
  * Remote vulnerabilities  
    Explore the wifi/bluetooth chip for vulnerabilities
  * Physical access  
    Explore the outside and inside connection ports for wired access
  * Chip-off  
    Get a second watch with intact firmware, remove the chip and connect directly to the chip to read off the data.

## Information Gathering
  For the information gathering to see what we have to work with I have based the information on
  * DC Rainmaker as mentioned above made a very thorough review of the watch. 
  * The initial user guide.
  * Poking around on the watch itself.
  * Opening the watch to inspect the inside.

  ![inside of Adidas smart run, backcover with heart sensor off](/assets/img/blog/adidas_smart_run_inside.jpg)
  Lots of interesting stuff that is possible to figure out, by just using public sources and
  examining the watch on both inside and outside.
  I have not done extensive searches for the chips and libraries found, 
  that will be done when I go further into the details.
  * Possible to connect to a micro-USB for charging and MP3 file transfer; only that.
  * Five contact points for the USB charger (micro-usb B)
  * Mio Alpha Heartrate Sensor
  * Wifi
  * GPS
  * Bluetooth 4.0
  * Support for MP3, AAC, Ogg, Vorbis music files and playlists
  * Touch screen with click and swipe
  * All configuration is done from server and then synced to watch, little configuration done directly on the watch.
  * Hardware
    * [Texas Instruments OMAP4430 application processor PWM IC TWL6030](http://www.ti.com/lit/ml/swpt034b/swpt034b.pdf)
    * WLAN / BT / GPS / FM combo chip Murata LBEL1CESEC (WL1281) - difficult to find datasheet
    * PoP RAM 512 MB
    * eMMC 4GB (Samsung KLM4G1YE4C-B001) - datasheet available
    * Chip with unknown functionality (RAM?) - SKhynix H9TKNNN4GOMA HRNGM 317a - difficult to find datasheet
    * Several extra contact points on the inside of the watch.
  * First firmware: Android JellyBean 4.1.1 (with custom UI obviously)
  * Current firmware version 5.0.1 (corresponds to android version?)

Huge list of open source licenses as they are required to list, and the files they are connected to.
This can obviously be used to do an educated guess on what libraries they have used for the different functionality, 
and also what potential software that exists on the watch, can be useful later down the road.

I'll finish off this post with a list of all the files that are mentioned. 
Next post will be to look for easter-eggs/backdoors/dev-doors that is exposed via the UI.

  * /fake_packages/ti-wpan-fw
  * /kernel
  * /root/init
  * /root/sbin/adbd
  * /system/app/MediaProvider.apk
  * /system/app/SettingsProvider.apk
  * /system/bin/adb
  * /system/bin/app_process
  * /system/bin/applypatch
  * /system/bin/atrace
  * /system/bin/bluetoothd
  * /system/bin/bu
  * /system/bin/dbus-daemon
  * /system/bin/debuggerd
  * /system/bin/dhcpcd
  * /system/bin/dnsmasq
  * /system/bin/drmserver
  * /system/bin/dumpsys
  * /system/bin/fsck_msdos
  * /system/bin/gzip
  * /system/bin/hciattach
  * /system/bin/ip
  * /system/bin/linker
  * /system/bin/logcat
  * /system/bin/logwrapper
  * /system/bin/make_ext4fs
  * /system/bin/mediaserver
  * /system/bin/mksh
  * /system/bin/mtpd
  * /system/bin/netcfg
  * /system/bin/pand
  * /system/bin/ping
  * /system/bin/pppd
  * /system/bin/racoon
  * /system/bin/requestsync
  * /system/bin/run-as
  * /system/bin/sdptool
  * /system/bin/service
  * /system/bin/system_server
  * /system/bin/tc
  * /system/bin/tinycap
  * /system/bin/tinymix
  * /system/bin/tinyplay
  * /system/bin/toolbox
  * /system/etc/dhcpcd/dhcdcp-hooks/20-dns.conf
  * /system/etc/dhcpcd/dhcpcd-hooks/95-configured
  * /system/etc/dhcdcd/dhcdcd-run-hooks
  * /system/etc/fallback_fonts-ja.xml
  * /system/etc/firmware/ti-connectivity/wl-1271-nvs_127x.bin
  * /system/etc/firmware/ti-connectivity/wl-127x-fw-4-mr.bin
  * /system/etc/firmware/ti-connectivity/wl-127x-fw-4-plt.bin
  * /system/etc/firmware/ti-connectivity/wl-127x-fw-4-sr.bin
  * /system/etc/firmware/ti-connectivity/wl-128x-fw-4-mr.bin
  * /system/etc/firmware/ti-connectivity/wl-128x-fw-4-rs.bin
  * /system/etc/mkshrc
  * /system/etc/security/cacerts/many many
  * /system/fonts/many many
  * /system/framework/am.jar
  * /system/framework/apache-xml.jar
  * /system/framework/bmgr.jar
  * /system/framework/bouncycastle.jar
  * /system/framework/bu.jar
  * /system/framework/content.jar
  * /system/framework/core-junit.jar
  * /system/framework/core.jar
  * /system/framework/ext.jar
  * /system/framework/framework-res.apk
  * /system/framework/framework.jar
  * /system/framework/jme.jar
  * /system/framework/input.jar
  * /system/framework/javax_obex.jar
  * /system/framework/monkey.jar
  * /system/framework/pm.jar
  * /system/framework/svc.jar
  * /system/lib/bluez-plugin/audio.so
  * /system/lib/bluez-plugin/bluetooth-health.so
  * /system/lib/bluez-plugin/input.so
  * /system/lib/bluez-plugin/mcapapp.so
  * /system/lib/bluez-plugin/network.so
  * /system/lib/drm/libfwdlockengine.so
  * /system/lib/hw/audio.a2dp.default.so
  * /system/lib/libESR_Portable.a
  * /system/lib/libFFTEm.so
  * /system/lib/libFLAC.a
  * /system/lib/libFraunhoferAAC.a
  * /system/lib/libLLVMAnalysis.a
  * /system/lib/libSR_AudioIn.so
  * /system/lib/libaah_rtp.so
  * /system/lib/libandroid_runtime.so
  * /system/lib/libandroidfw.so
  * /system/lib/libapplypatch.a
  * /system/lib/libaudioflinger.so
  * /system/lib/libbcc.so.sha1
  * /system/lib/libbcc.so
  * /system/lib/libbluetoothd.so
  * /system/lib/libbtio.so
  * /system/lib/libbuiltinplugin.a
  * /system/lib/libbz.a
  * /system/lib/libc.so
  * /system/lib/libc.a
  * /system/lib/libc_common.a
  * /system/lib/libc_nomalloc.a
  * /system/lib/libcamera_client.so
  * /system/lib/libcameraservice.so
  * /system/lib/libcommon_time_client.so
  * /system/lib/libcorkscrew.so
  * /system/lib/libcrypto.so
  * /system/lib/libctest.so
  * /system/lib/libcutils.so
  * /system/lib/libcutils.a
  * /system/lib/libdbus-tools-common.a
  * /system/lib/libdbus.so
  * /system/lib/libdl.so
  * /system/lib/libdrmframework.so
  * /system/lib/libdrmframeworkcommon.sa
  * /system/lib/libdrmutility.a
  * /system/lib/libexif.so
  * /system/lib/libexif_jni.so
  * /system/lib/libexpat.so
  * /system/lib/libext4_utils.so
  * /system/lib/libfdlibm.a
  * /system/lib/libft2.a
  * /system/lib/libgccdemangle.so
  * /system/lib/libgdbus_static.a
  * /system/lib/libgif.a
  * /system/lib/libgsm.a
  * /system/lib/libhardware.so
  * /system/lib/libhardware_legacy.so
  * /system/lib/libharfbuzz.so
  * /system/lib/libhwui.so
  * /system/lib/libhyphenation.a
  * /system/lib/libicuii18n.so
  * /system/lib/libicuuc.so
  * /system/lib/libiprouteutil.so
  * /system/lib/libipsec.a
  * /system/lib/libjavacore.so
  * /system/lib/libjpeg.so
  * /system/lib/liblog.a
  * /system/lib/liblog.so
  * /system/lib/liblzf.a
  * /system/lib/libm.so
  * /system/lib/libmedia.so
  * /system/lib/libmedia_helper.a
  * /system/lib/libmedia_jni.so
  * /system/lib/libmediaplayerservice.so
  * /system/lib/libmincrypt.a
  * /system/lib/libmtp.so
  * /system/lib/libmusicbundle.a
  * /system/lib/libnativehelper.so
  * /system/lib/libnbaio.a
  * /system/lib/libnetlink.so
  * /system/lib/libnetutils.so
  * /system/lib/libnfc_ndef.so
  * /system/lib/libpixelflinger.so
  * /system/lib/libpng.a
  * /system/lib/libpower.so
  * /system/lib/libreference-ril.so
  * /system/lib/libreverb.a
  * /system/lib/libscheduling_policy.a
  * /system/lib/libskia.so
  * /system/lib/libskiagpu.so
  * /system/lib/libsonivox.so
  * /system/lib/libsoundpool.so
  * /system/lib/libspeexresampler.so
  * /system/lib/liblibsqlite.so
  * /system/lib/libsqlite3_android.a
  * /system/lib/libsqlite_jni.so
  * /system/lib/libssl.so
  * /system/lib/libstagefright.so
  * /system/lib/libstagefright_aacenc.a
  * /system/lib/libstagefright_amrnb_common.so
  * /system/lib/libstagefright_amrnbdec.a
  * /system/lib/libstagefright_amrnbenc.a
  * /system/lib/libstagefright_avc_common.so
  * /system/lib/libstagefright_color_conversion
  * /system/lib/libstagefright_enc_common.so
  * /system/lib/libstagefright_foundation.so
  * /system/lib/libstagefright_httplive.a
  * /system/lib/libstagefright_matroska.a
  * /system/lib/libstagefright_mp3dec.a
  * /system/lib/libstagefright_mpeg2ts.a
  * /system/lib/libstagefright_nuplayer.a
  * /system/lib/libstagefright_omx.so
  * /system/lib/libstagefright_rtsp.a
  * /system/lib/libstagefright_soft_aacdec.so
  * /system/lib/libstagefright_soft_mp3dec.so
  * /system/lib/libstagefright_soft_rawdec.so
  * /system/lib/libstagefright_soft_vorbisdec.so
  * /system/lib/libstagefright_soft_vpxdec.so
  * /system/lib/libstagefright_timedtext.a
  * /system/lib/libstagefright_yuv.so
  * /system/lib/libstdc++.so
  * /system/lib/libstlport.so
  * /system/lib/libstorage.a
  * /system/lib/libthread_db.a
  * /system/lib/libthread_db.so
  * /system/lib/libtinyalsa.so
  * /system/lib/libui.so
  * /system/lib/libunz.a
  * /system/lib/libutils.a
  * /system/lib/libv8.a
  * /system/lib/libvideoeditor_3gpwriter.a
  * /system/lib/libvideoeditor_core.so
  * /system/lib/libvideoeditor_jni.so
  * /system/lib/libvideoeditor_mcs.a
  * /system/lib/libvideoeditor_stagefrightshe
  * /system/lib/libvideoeditor_videofilters.so
  * /system/lib/liblibvideoeditorplayer.so
  * /system/lib/libvorbisidec.so
  * /system/lib/libvpx.a
  * /system/lib/libwebcore.a
  * /system/lib/libwebcore.so
  * /system/lib/libwfd_mpeg2tsrtsp.so
  * /system/lib/libxml2.a
  * /system/lib/libxslt.a
  * /system/lib/libz.so
  * /system/lib/libzipfile.a
  * /system/lib/soundfx/libaudiopreprocessing.so
  * /system/lib/soundfx/libbundlewrapper.so
  * /system/lib/soundfx/libdownmix.so
  * /system/lib/soundfx/libreverbwrapper.so
  * /system/lib/soundfx/libvisualizer.so
  * /system/usr/icu/icudt48l.dat
  * /system/xbin/agent
  * /system/xbin/attest
  * /system/xbin/avinfo
  * /system/xbin/avtest
  * /system/xbin/bdaddr
  * /system/xbin/dbus-monitor
  * /system/xbin/dbus-send
  * /system/xbin/dexdump
  * /system/xbin/gaptest
  * /system/xbin/hciconfig
  * /system/xbin/hcitool
  * /system/xbin/hstest
  * /system/xbin/l2ping
  * /system/xbin/l2test
  * /system/xbin/lmptest
  * /system/xbin/rctest
  * /system/xbin/rfcomm
  * /system/xbin/scotest
  * /system/xbin/sdptest
  * /system/xbin/

