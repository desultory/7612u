## Bridged Wireless Access Point - RasPiOS

A bridged wireless access point (aka Dumb AP) works within an existing
ethernet network to add WiFi capability where it does not exist or to
extend the network to WiFi capable computers and devices in areas where
the WiFi signal is weak or otherwise does not meet expectations.

#### Single Band

This document outlines a single band 5 GHz setup using the Raspberry Pi 4B
with a USB 3 WiFi adapter.

#### Information

This setup supports WPA3-SAE. It is disabled by default.

WPA3-SAE will not work with Realtek 88xx chipset based USB WiFi adapters.

WPA3-SAE will work with Mediatek 761x chipset based USB WiFI adapters.

-----

2021-05-21

#### Tested Setup

Raspberry Pi 4B (4gb)

Raspberry Pi OS (2021-03-04) (32 bit) (kernel 5.10.17-v7l+)

Ethernet connection providing internet

USB WiFi Adapter

[Case](https://www.amazon.com/dp/B07X8RL8SL)

[Right Angle USB Extender](https://www.amazon.com/dp/B07S6B5X76)

[Power Supply](https://www.amazon.com/dp/B08C9VYLLK)

Note: I use the case upside down. There are several little things that
work better with the case upside down and no negatives that I can find.

Note: Very few Powered USB 3 Hubs will work well with Raspberry Pi
hardware. The primary problem has to do with the backfeeding of
current into the Raspberry Pi. I have avoided using a powered hub
in this setup to enable a very high degree of stability.

Note: The rtl88XXxu chipset based USB3 WiFi adapters require from 504 mA of
power up to well over 800 mA of power depending on the adapter. The Raspberry
Pi 3B, 3B+ and 4B USB subsystems are only able to supple a total of 1200
mA of power to all attached devices.

Note: The Alfa AWUS036ACM adapter, mt7612u based adapter, requests a maximum
of 400 mA from the USB subsystem during initialization. Testing with a meter
shows actual usage of 360 mA during heavy load and usage of 180 mA during
light loads. This is much lower power usage than most AC1200 class adapters
which makes this adapter a good choice for a Raspberry Pi based access point.


#### Setup Steps

-----

USB WiFi adapter driver installation, if required, should be performed and tested
prior to continuing.

Note: For USB3 adapters based on Realtek rtl88xx chipsets, the following module
parameters may be needed for best performance:
```
rtw_vht_enable=2 rtw_switch_usb_mode=1
```
Note: For USB3 adapters based on Mediatek mt7612ux chipsets, the following module
parameters may be needed for best performance:
```
disable_usb_sg=1
```
-----

Determine names and state of the network interfaces.

Code:
```
ip a
```
Note: If the interface names are not `eth0` and `wlan0`,
then the interface names used in your system will have to replace
`eth0` and `wlan0` for the remainder of this document.

-----

Update system.

Code:
```
sudo apt update
```

-----

Upgrade system.

Code:
```
sudo apt full-upgrade
```
Note: Upgrading system is not mandatory for this installation but since some
users forget to upgrade their system on a regular basis, maybe it is a good idea.

-----

Reduce overall power consumption and overclock the CPU a modest amount.

Note: All items in this step are optional and some items are specific to
the Raspberry Pi 4B. If installing to a Raspberry Pi 3b or 3b+ you will
need to use the appropriate settings for that hardward.

Code:
```
sudo nano /boot/config.txt
```
Change
```
# turn off onboard audio
dtparam=audio=off

# disable DRM VC4 V3D driver on top of the dispmanx display stack
#dtoverlay=vc4-fkms-v3d
#max_framebuffers=2
```
Add
```
# overclock CPU
over_voltage=1
arm_freq=1600

# turn off Mainboard LEDs
dtoverlay=act-led

# disable Activity LED
dtparam=act_led_trigger=none
dtparam=act_led_activelow=off

# disable Power LED
dtparam=pwr_led_trigger=none
dtparam=pwr_led_activelow=off

# turn off Ethernet port LEDs
dtparam=eth_led0=4
dtparam=eth_led1=4

# turn off Bluetooth
dtoverlay=disable-bt

# turn off onboard WiFi
dtoverlay=disable-wifi
```
-----

Install needed package. Website - [hostapd](https://w1.fi/hostapd/)

Code:
```
sudo apt install hostapd
```
-----

Enable the wireless access point service and set it to start when your
Raspberry Pi boots.

Code:
```
sudo systemctl unmask hostapd

sudo systemctl enable hostapd
```
-----

Create hostapd configuration file.

Code:
```
sudo nano /etc/hostapd/hostapd.conf
```
File contents
```
# /etc/hostapd/hostapd.conf
# Documentation: https://w1.fi/cgit/hostap/plain/hostapd/hostapd.conf
# 2021-05-20

# Defaults:
# SSID: myAP
# PASSPHRASE: myPW1234
# Band: 5g
# Channel: 36
# Country: US

# needs to match wireless interface in your system
interface=wlan0

# needs to match bridge interface name in your system
bridge=br0

driver=nl80211
ctrl_interface=/var/run/hostapd
ctrl_interface_group=0

# change as desired
ssid=myAP

# change as required
country_code=US

# enable DFS channels
ieee80211d=1
ieee80211h=1

# a = 5g (a/n/ac)
# g = 2g (b/g/n)
hw_mode=a
channel=36
#channel=149

beacon_int=100
dtim_period=2
max_num_sta=32
macaddr_acl=0
rts_threshold=2347
fragm_threshold=2346
#send_probe_response=1

# security
# change wpa_passphrase as desired
wpa_passphrase=myPW1234
auth_algs=1
ignore_broadcast_ssid=0
# wpa=2 is required for WPA2 and WPA3 (read the docs)
wpa=2
rsn_pairwise=CCMP
# only one wpa_key_mgmt= line should be active.
# wpa_key_mgmt=WPA-PSK is required for WPA2-AES
wpa_key_mgmt=WPA-PSK
# wpa_key_mgmt=SAE WPA-PSK is required for WPA3-AES Transitional
#wpa_key_mgmt=SAE WPA-PSK
# wpa_key_mgmt=SAE is required for WPA3-SAE
#wpa_key_mgmt=SAE
#wpa_group_rekey=1800
# ieee80211w=1 is required for WPA-3 SAE Transitional
# ieee80211w=2 is required for WPA-3 SAE
#ieee80211w=1
# if parameter is not set, 19 is the default value.
#sae_groups=19 20 21 25 26
# sae_require_mfp=1 is required for WPA-3 SAE Transitional
#sae_require_mfp=1
# if parameter is not 9 set, 5 is the default value.
#sae_anti_clogging_threshold=10

# Note: Capabilities can vary even between adapters with the same chipset
#
# IEEE 802.11n
ieee80211n=1
wmm_enabled=1
#
# Note: Only one ht_capab= line and one vht_capab= should be active. The
# content of these lines is determined by the capabilities of your adapter.
#
# rtl8812au - rtl8811au -  rtl8812bu - rtl8811cu - rtl8814au
# band 1 - 2g - 20 MHz channel width
#ht_capab=[SHORT-GI-20][MAX-AMSDU-7935]
# band 2 - 5g - 40 MHz channel width
#ht_capab=[HT40+][HT40-][SHORT-GI-20][SHORT-GI-40][MAX-AMSDU-7935]
#
# mt7612u
# to support 20 MHz channel width on 11n
#ht_capab=[LDPC][SHORT-GI-20][TX-STBC][RX-STBC1]
# to support 40 MHz channel width on 11n
ht_capab=[LDPC][HT40+][HT40-][GF][SHORT-GI-20][SHORT-GI-40][TX-STBC][RX-STBC1]
#

# IEEE 802.11ac
ieee80211ac=1
#
# rtl8812au - rtl8811au -  rtl8812bu - rtl8811cu - rtl8814au
# band 2 - 5g - 80 MHz channel width
#vht_capab=[MAX-MPDU-11454][SHORT-GI-80][HTC-VHT]
# Note: [TX-STBC-2BY1] causes problems
#
# mt7612u
# band 2 - 5g - 80 MHz channel width on 11ac
vht_capab=[RXLDPC][SHORT-GI-80][TX-STBC-2BY1][RX-STBC-1][MAX-A-MPDU-LEN-EXP3][RX-ANTENNA-PATTERN][TX-ANTENNA-PATTERN]
#
# Required for 80 MHz width channel operation on band 2 - 5g
vht_oper_chwidth=1
#
# Use the next line with channel 36  (36 + 6 = 42) band 2 - 5g
vht_oper_centr_freq_seg0_idx=42
#
# Use the next line with channel 149 (149 + 6 = 155) band 2 - 5g
#vht_oper_centr_freq_seg0_idx=155

# Event logger - as desired
#logger_syslog=-1
#logger_syslog_level=2
#logger_stdout=-1
#logger_stdout_level=2

# WMM - as desired
#uapsd_advertisement_enabled=1
#wmm_ac_bk_cwmin=4
#wmm_ac_bk_cwmax=10
#wmm_ac_bk_aifs=7
#wmm_ac_bk_txop_limit=0
#wmm_ac_bk_acm=0
#wmm_ac_be_aifs=3
#wmm_ac_be_cwmin=4
#wmm_ac_be_cwmax=10
#wmm_ac_be_txop_limit=0
#wmm_ac_be_acm=0
#wmm_ac_vi_aifs=2
#wmm_ac_vi_cwmin=3
#wmm_ac_vi_cwmax=4
#wmm_ac_vi_txop_limit=94
#wmm_ac_vi_acm=0
#wmm_ac_vo_aifs=2
#wmm_ac_vo_cwmin=2
#wmm_ac_vo_cwmax=3
#wmm_ac_vo_txop_limit=47
#wmm_ac_vo_acm=0

# TX queue parameters - as desired
#tx_queue_data3_aifs=7
#tx_queue_data3_cwmin=15
#tx_queue_data3_cwmax=1023
#tx_queue_data3_burst=0
#tx_queue_data2_aifs=3
#tx_queue_data2_cwmin=15
#tx_queue_data2_cwmax=63
#tx_queue_data2_burst=0
#tx_queue_data1_aifs=1
#tx_queue_data1_cwmin=7
#tx_queue_data1_cwmax=15
#tx_queue_data1_burst=3.0
#tx_queue_data0_aifs=1
#tx_queue_data0_cwmin=3
#tx_queue_data0_cwmax=7
#tx_queue_data0_burst=1.5

# end of hostapd.conf
```
-----

Establish hostapd conf file and log file locations.

Note: Make sure to change <your_home> to your home directory.

Code:
```
sudo nano /etc/default/hostapd
```
Add to bottom of file
```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
DAEMON_OPTS="-d -K -f /home/<your_home>/hostapd.log"
```
-----

Block the eth0 and wlan0 interfaces from being processed, and let dhcpcd
configure only br0 via DHCP.

Code:
```
sudo nano /etc/dhcpcd.conf
```
Add the following line above the first `interface xxx` line, if any
```
denyinterfaces eth0 wlan0
```
Go to the end of the file and add the following line
```
interface br0
```
-----

Enable systemd-networkd service. Website - [systemd-network](https://www.freedesktop.org/software/systemd/man/systemd.network.html)

Code:
```
sudo systemctl enable systemd-networkd
```
-----

Create bridge interface br0.

Code:
```
sudo nano /etc/systemd/network/10-bridge-br0-create.netdev
```
File contents
```
[NetDev]
Name=br0
Kind=bridge
```
-----

Bind ethernet interface.

Code:
```
sudo nano /etc/systemd/network/20-bridge-br0-bind-ethernet.network
```
File contents
```
[Match]
Name=eth0

[Network]
Bridge=br0
```
-----

Configure bridge interface.

Code:
```
sudo nano /etc/systemd/network/21-bridge-br0-config.network
```
Note: The contents of the Network block below should reflect the needs of your network.

File contents
```
[Match]
Name=br0

[Network]
Address=192.168.1.100/24
Gateway=192.168.1.1
DNS=8.8.8.8
```
-----

Ensure WiFi radio not blocked.

Code:
```
$ sudo rfkill unblock wlan
```
-----

Reboot system.

Code:
```
$ sudo reboot
```
-----

End of installation.

-----

Notes:

-----

Restart systemd-networkd service.

Code:
```
$ sudo systemctl restart systemd-networkd
```
-----

Check status of the services.

Code:
```
$ systemctl status hostapd

$ systemctl status systemd-networkd
```
-----

Autostarting iperf3

Code:
```
sudo apt install iperf3
```
Code:
```
sudo nano /etc/systemd/system/iperf3.service
```
File contents
```
[Unit]
Description=iPerf3 Service
After=syslog.target network.target auditd.service

[Service]
Type=simple
ExecStart=/usr/bin/iperf3 -s

[Install]
WantedBy=multi-user.target
```
Code:
```
sudo systemctl enable iperf3
```
Check iperf3 status

Code:
```
sudo systemctl status iperf3
```

-----
