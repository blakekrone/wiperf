# Contents

- [Wiperf - Configuration on the WLAN Pi](#wiperf---configuration-on-the-wlan-pi)
    - [Hostname](#hostname)
    - [Configuration File (config.ini)](#configuration-file-configini)
    - [Wireless Client Configuration (wpa_supplicant.conf)](#wireless-client-configuration-wpa_supplicantconf)
- [Testing](#testing)
- [Updating](#updating)
- [Known Issues](#known-issues)
    - [iperf Test Fail](#iperf-tests-fail)
    - [Interface Bounce Failures](#interface-bounce-failures)
    - [DNS Lookup Order Issue](#dns-lookup-order-issue)
    - [Hostname Change Related Issues](#hostname-change-related-issues)
    - [Comfast CF-912 80MHz Width Channels](#comfast-cf-912-80mhz-width-channels)
    - [MCS & Rx Phy rates Missing From Reports](#mcs--rx-phy-rates-missing-from-reports)
    - [Tests Fail To Start Due to DNS Failures](#tests-fail-to-start-due-to-dns-failures)
- [Troubleshooting](#troubleshooting)
- [Additional Features](#additional-features)
    - [Watchdog](#watchdog)
    - [Security](#security)
    - [CLI Mode Switch](#cli-mode-switch)

# Wiperf - Configuration on the WLAN Pi

This instruction paper assumes you are running Wiperf on a WLAN Pi on an image version of v1.9 or later (which has Wiperf installed and available as part of the image.)

(**Special note for WLAN Pi image v1.9.0:** please see the [Known Issues](#known-issues)) section of this document)

The Wiperf probe is activated via the front panel menu system (FPMS) of the WLAN Pi. But, before flipping in to the Wiperf mode, a few configuration steps need to be completed:

[top](#contents)

## Hostname

It is strongly advised that you configure the hostname of your WLAN Pi before following the steps detailed below. The data sent to and stored in Splunk will be associated with the WLAN Pi hostname that is used when the data is forwarded. If you decide to subsequently change the hostname, then historical data from the unit will not be associated with the data sent with the new hostname.

If you are running multiple WLAN Pi units, then you MUST change their hostnames, as you will not be able to differentiate their data within Splunk. All data, from all units, will be shown under the default 'wlanpi' hostname.

To change the WLAN Pi hostname, please check out the following [FAQ][hostname_faq] page and complete all suggested steps: [link][hostname_faq]

[top](#contents)

## Configuration File (config.ini)

The operation of Wiperf is configured using the file `'/home/wanpi/wiperf/config.ini'` This needs to be edited prior to entering Wiperf mode.

Prior to the first use of Wiperf, the config.ini file does not exist in the required WLAN Pi directory. However, a default template config file (`config.default.ini`) is supplied that can be used to create the `config.ini` file. Here is the suggested workflow:

Connect to the WLAN Pi, create a copy of the config template file and edit the newly created config (as the wlanpi user):

```
        cd /home/wlanpi/wiperf
        cp ./config.default.ini ./config.ini
        nano ./config.ini
```

By default, the configuration file is set to run all tests. However, there is a minimum configuration that must be applied for Wiperf mode to run out-of-the-box. Here are the minimum configuration parameters you need to configure (just to get you going...):

```
[General]
; interface name over which mgt traffic is sent (i.e. how we get to Splunk) - options: wlan0, eth0, zt
mgt_if: wlan0

; Splunk host IP/name - set this to the address of your Splunk server
data_host: 192.168.0.99

; Splunk token to access Splunk server created by Splunk (example token: 84adb9ca-071c-48ad-8aa1-b1903c60310d)
; (you need to have your Splunk server up & configure to get this)
splunk_token: 84adb9ca-071c-48ad-8aa1-b1903c60310d

[Iperf3_tcp_test]
; IP address of iperf3 server - set this to your iperf3 server (if you have one)
server_hostname: 192.168.0.14

[Iperf3_udp_test]
; IP address of iperf3 server - set this to your iperf3 server (if you have one)
server_hostname: 192.168.0.14
```

If there are some tests you'd like to disable (e.g. if you don't have an iperf3 server set up), then you'll need to open up the config.ini file and look through each section for the "enabled" parameter for that test and set it to "no". For example, to disable the iperf tcp test: 

```
[Iperf3_tcp_test]
; yes = enabled, no = disabled
enabled: no
```

For a full description of the configuration file parameters, please review the following page: [config.ini reference guide](README_Config.ini.md). The Splunk token is obtained from your Splunk server (see [Splunk build guide][splunk_build]). 

[top](#contents)

## Wireless Client Configuration (wpa_supplicant.conf)

When the WLAN Pi is flipped in to Wiperf mode, it will need to join the SSID under test to run the configured tests. We need to provide a configuration (that is only used in Wiperf mode) to allow the WLAN Pi to join a WLAN.

Edit the following file with the configuration and credentials that will be used by the WLAN Pi to join the SSID under test once it is switched in to Wiperf mode (make edits logged in as the wlanpi user):

```
        cd /home/wlanpi/wiperf/conf/etc/wpa_supplicant
        nano ./wpa_supplicant.conf
```

[top](#contents)

# Testing

Once the required edits have been made to configure Wiperf mode, flip the WLAN Pi in to Wiperf mode using the following FPMS options:

```
        Actions > Wiperf > Confirm
```

If no errors are observed on the FPMS during flip-over, inspect the following files to double-check for errors & verify that test data is generated (as indicated in the log messages):
```    
    cat /home/wlanpi/wiperf/logs/agent.log
    cat /home/wlanpi/wiperf/wiperf.log 
```
Note that by default the tests are run every 5 mins unless the interval has been changed in the `config.ini` file. Wait at least this interval before determining that there is an issue - the test cycle will NOT begin immediately upon entering Wiperf mode.

Check your instance of Splunk and verify that data is being received.

[top](#contents)

# Updating

To get the latest updates from the GitHub repo (when available), use the following commands when logged in as the wlanpi user:

```
        cd ~/wiperf
        git pull https://github.com/wifinigel/wiperf.git
```

(note that this will update config.default.ini but not config.ini, so remember to re-edit it after a pull if the config file format changes)

[top](#contents)

# Known Issues:

## iperf Tests Fail

There is an issue with the v1.9.0 WLAN Pi image that means that the iperf tests fail when running in wiperf mode. To get the fixed version, follow the update process detailed in the [Updating](#updating) section of this document (3rd Jan 2020)

[top](#contents)

## Interface Bounce Failures

There is an issue in WLAN Pi image versions v1.9.0 & v1.9.1 with some required commands missing from the ```/etc/sudoers.d/wlanpidump``` file. This issue manifests itself with errors about WLAN interface bounce operations failing in the agent.log file. To fix this issue, please apply the following fix:

 Add the following entries to the /etc/sudoers.d/wlanpidump file:

```
/sbin/ifdown 
/sbin/ifup
```
The modified file content should be as follows:
```
wlanpi ALL = (root) NOPASSWD: /sbin/iwconfig, /usr/sbin/iw, /sbin/dhclient, /sbin/ifconfig, /sbin/reboot, /bin/kill, /bin/date, /sbin/ifdown, /sbin/ifup
```
Reboot the WLAN Pi after applying this file update

[top](#contents)

## DNS Lookup Order Issue

By default in WLAN Pi image versions up to and including v1.9.1, the DNS server value ```8.8.8.8``` has been hard-coded in to the DNS server lookup list. This has been achieved by configuring the following line in to the file ```/etc/resolvconf/resolv.conf.d/head```:

```
nameserver 8.8.8.8
```
This ensures that the 8.8.8.8 address is always used as the first DNS lookup entry, even if the WLAN Pi receives a DNS server address during its DHCP process. This can be verified by performing a ```cat /etc/resolv.conf``` when the WLAN Pi is in wiperf mode - even though there may be two or more entries in the file, 8.8.8.8 will always be shown at the top of the file listing, and be used as the first lookup server. This may not be desirable behaviour, as 8.8.8.8 may not be available or DHCP assigned servers may be preferred. 

To ensure that the desired DNS environment is used, the following options exist:

1. Remove the 8.8.8.8 entry completely: edit the file  ```/etc/resolvconf/resolv.conf.d/head``` and remove the offending entry completely. This will ensure only the DHCP assigned DNS server(s) will be used.
2. Replace the 8.8.8.8 entry by editing the  ```/etc/resolvconf/resolv.conf.d/head``` and replacing it with a desired value
3. Move the 8.8.8.8 entry from the ```/etc/resolvconf/resolv.conf.d/head``` file to the ```/etc/resolvconf/resolv.conf.d/tail``` file, so that 8.8.8.8 is still used, but is now the last option in the DNS server list (only used when other preceding servers do not return a result)

[top](#contents)

## Hostname Change Related Issues 

There have been a number of issues reported that have been reported that are due to the WLAN Pi hostname being changed from the default, but it has not been updated in both the ```/etc/hostname``` AND ```/etc/hosts``` file. Please ensure you have followed [this process][hostname_faq] : [Link][hostname_faq]

The issue tends to manifest itself as various "weird" issues such as "sudo" commands failing for no apparent reason. 

[top](#contents)

## Comfast CF-912 80MHz Width Channels

There seems to be an issue with the Comfast CF-912 adapter when using it with the WLAN Pi and associating as a client to SSIDs that use 80MHz width channels. If you hit an issue where the WLAN Pi seems to lock up or does not boot correctly, try a different adapter or a network that does not use 80Mhz channels.

[top](#contents)

## MCS & Rx Phy rates Missing From Reports

[top](#contents)

In several dashboard reports, the reported MCS values & Rx Phy rate may be blank. This is because these values are only reported by MediaTek wireless NICs. Therefore, the CF-912 will not show these values (as it is a Realtek NIC). Sorry, not much I can do about this.

[top](#contents)

## Tests Fail To Start Due to DNS Failures

In versions of wiperf before version v0.10, the wiperf probe performed a series of tests to verify the health of the wireless connection prior to tests running. One of these tests included a DNS lookup to "bbc.co.uk" to verify Internet connectivity.

In some environments, this may not be a valid test. To fix this issue, a new configuring parameter was added to the config.ini file that allows a custom lookup target to be provided, if requried: 
```
connectivity_lookup: google.com
```

[top](#contents)

# Troubleshooting:

If things seem to be going wrong, try the following:

- Connect to the WLAN Pi using the USB OTG connection to check log files: 
    - `cat /home/wlanpi/wiperf/logs/agent.log`
    - `cat /home/wlanpi/wiperf/wiperf.log`
- SSH to the device & tail the agent log file in real-time, watching for errors and dumps of test results being performed:`tail -f /home/wlanpi/wiperf/logs/agent.log`
- Flip back in to classic mode and activate Wiperf mode from the CLI of the WLAN Pi, watching for errors:
    - `cd /home/wlanpi/wiperf`
    - `sudo ./wiperf_switcher`
- Try disabling tests & see if one specific test is causing an issue
- Make sure all pre-reqs have definitely been fulfilled
- Make sure your WLAN Pi and Splunk servers are NTP sync'ed
- Flip back to classic mode and re-check the edits made to the `config.ini` & `wpa_supplicant.conf` files
- If you have changed the WLAN Pi hostname from its default, make sure you have updated both the `/etc/hosts` **AND** the `/etc/hostname` file as per the instructions [here][hostname_faq] (this can cause some very weird issues!)
- Check the order of DNS servers being used by running the command `cat /etc/resolv.conf` on the CLI of the WLAN Pi when it is in wiperf mode and connected to the wireless network. If there are multiple servers shown and you see `8.8.8.8` at the top of the list, you may need to move the 8.8.8.8 entry from the file `/etc/resolvconf/resolv.conf.d/head` to `/etc/resolvconf/resolv.conf.d/tail` and then reboot. This should shift 8.8.8.8 to be bottom of the list in `cat /etc/resolv.conf` and may fix your name resolution issues

[top](#contents)

# Additional Features:

## Watchdog

Wiperf has a watchdog feature that it uses to try to reset things when it is having connectivity related difficulties.

There may be instances when tests are continually failing or wireless connectivity is intermittent due to perhaps being stuck on a remote AP that is sub-optimal from a connectivity perspective.

If persistent issues are detected, then Wiperf will reboot the WLAN Pi to try to remediate the issue. This will provide the opportunity to the reset all network connectivity and internal processes.

Note that this is a last ditch mechanism. Wiperf will try bouncing the WLAN interface to remediate any short-term connectivity issues, which will likely fixe many issues without the need for a full reboot.

If you observe your WLAN Pi rebooting on a regular basis (e.g. a couple of times a hour), then check its logs as it is very unhappy about something.

[top](#contents)

## Security

Wiperf employs the following security mechanisms in an attempt to harden the WLAN Pi when deployed in Wiperf mode:

- No forwarding is allowed between interfaces
- The internal UFW firewall is configured to only allow incoming connectivity on port 22 on the wlan0 & eth0 interfaces

[top](#contents)

## CLI Mode Switch

If you are remote from your WLAN Pi you may not be able to flip it in to Wiperf mode using the front panel buttons. However, it is possible to flip it in to Wiperf mode using the CLI (via an SSH session).

To flip in to Wiperf mode using the CLI, SSH to your WLAN Pi and execute the following on the CLI (**CAVEAT:** make sure you have correctly configured the /home/wlanpi/wiperf/conf/etc/wpa_suplicant/wpa_supplicant.conf file before do this...otherwise you may lose comms with the WLAN Pi after it reboots if you rely on the WLAN connection for access):

```
 # performed as the wlanpi user
 cd ~/wiperf
 sudo ./wiperf_switcher on
 ```
 After executing this command, the WLAN Pi will reboot in to the Wiperf mode.
 
If you'd like to flip back from Wiperf mode, SSH to the WLAN Pi and execute:

```
 # performed as the wlanpi user
 cd ~/wiperf
 sudo ./wiperf_switcher off
 ```

[top](#contents)



<!-- link list -->
[wlanpi_build]: docs/README_WLANPi_Image_Build.md
[wlanpi_config]: docs/README_WLANPi_Config.md
[config_ini]: docs/README_Config.ini.md
[splunk_build]: https://github.com/wifinigel/wiperf/raw/master/docs/WLANPi%20Wiperf%20Probe%20-%20Splunk%20Build.pdf
[hostname_faq]: https://wlan-pi.github.io/wlanpi-documentation/faq/#how-do-i-change-the-hostname-of-my-wlan-pi
