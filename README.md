# pfSense-pfBlockerNG
pfBlockerNG can add other security enhancements such as blocking known bad IP addresses with blocklists. For example, getting rid of adverts and pop-ups from websites. If you don’t already have a blocklist functionality in place on your pfSense (such as PiHole), I would strongly suggest adding pfBlockerNG Devel to your new OpenVPN Gateways (VPNGGATEWORLD and VPNGATELOCAL).

Network prerequisites are:
- [x] Layer 2 Network Switches
- [x] Network Gateway is `192.168.1.5`
- [x] Network DNS server is `192.168.1.5` (Note: your Gateway hardware should enable you to a configure DNS server(s), like a UniFi USG Gateway, so set the following: primary DNS `192.168.1.254` which will be your PiHole server IP address; and, secondary DNS `1.1.1.1` which is a backup Cloudfare DNS server in the event your PiHole server 192.168.1.254 fails or os down)
- [x] Network DHCP server is `192.168.1.5`

Other Prerequisites are:
- [x] Proxmox node fully configured as per [PROXMOX-NODE BUILDING](https://github.com/ahuacate/proxmox-node/blob/master/README.md#proxmox-node-building)
- [x] pfSense is fully configured as per [pfSense - Setup](https://github.com/ahuacate/pfsense-setup/blob/master/README.md#pfsense---setup)

Tasks to be performed are:
- [1.00 Install pfBlockerNG Package](#100-install-pfblockerng-package)
- [2.00 Configure General Settings](#200-configure-general-settings)
- [3.00 Configure IP Settings](#300-configure-ip-settings)
- [4.00 Configure DNSBL Settings](#400-configure-dnsbl-settings)
	- [4.01 Enable DNSBL](#401-enable-dnsbl)
	- [4.02 Configure DNSBL feeds](#402-configure-dnsbl-feeds)
	- [4.03 Force DNSBL Feed Updates](#403-force-dnsbl-feed-updates)
- [5.00 Check if pfBlockerNG is working](#500-check-if-pfblockerng-is-working)
- [00.00 Patches and Fixes](#0000-patches-and-fixes)


## 1.00 Install pfBlockerNG Package
In the pfSense WebGUI go to `System` > `Package Manager` > `Available Packages` and type ‘pfblocker’ into the search criteria and then click `Search`.

Make sure you click `+ Install` on the version with ‘-devel’ (i.e pfBlockerNG-devel) at the end of it, and then `Confirm` on the next page. Installation may take a short while as it downloads and updates certain packages.  

## 2.00 Configure General Settings
At this point, you have already installed the package. Next, you will need to enable it from pfSense WebGUI `Firewall` > `pfBlockerNG` and the option to exit out of the wizard. A configuration page should appear, Click on the `General Tab`, and fill out the necessary fields as follows:

| General Settings | Value | Value | Value | Value | Notes
| :---  | :--- | :--- | :--- | :--- | :---
| pfBlockerNG | `☑` Enable | 
| Keep Settings | `☑` Enable |
| CRON Settings | Once a day | 00 | 0 | 0 | *Generally Leave Default settings*

Then Click `Save` at the bottom of the page.

## 3.00 Configure IP Settings
Go to pfSense WebGUI `Firewall` > `pfBlockerNG` > `IP Tab` and fill out the necessary fields as follows. Whats NOT shown in the below table leave as default.

| IP Configuration | Value | Other Values | Notes
| :---  | :--- | :--- | :---
| De-Duplication | `☑` Enable | |*Check*
| CIDR Aggregation | `☑` Enable ||*Check*
| Suppression | `☑` Enable ||*Check*
| Global Logging | `☐` ||*Uncheck*
| Placeholder IP Address | 127.1.7.7||*Leave Default*
| MaxMind Localized Language | English||*Leave Default*
| MaxMind Updates | `☐` Check to disable MaxMind updates ||*Uncheck*
| Global Logging | `☐` ||*Uncheck*
| **IP Interface/Rules Configuration**
| Inbound Firewall Rules | `VPNGATEWORLD01` || *Select ONLY VPNGATEWORLD and VPNGATELOCAL*
|| `VPNGATEWORLD02`
|| `VPNGATELOCAL01`
|| `VPNGATELOCAL02`
|| `VPNGATELOCAL03`
| Outbound Firewall Rules | `VPNGATEWORLD01` || *Select ONLY VPNGATEWORLD and VPNGATELOCAL*
|| `VPNGATEWORLD02`
|| `VPNGATELOCAL01`
|| `VPNGATELOCAL02`
|| `VPNGATELOCAL03`
| Floating Rules | ☑ Enabled || *Check*
| Firewall 'Auto' Rule Order | pfB_Pass/Match/Block/Reject\All other Rules\(Default Format) || *Leave Default*
| Firewall 'Auto' Rule Suffix | `auto rule`
| Kill States | `☑` Enable

And click `Save IP Settings`

![alt text](https://raw.githubusercontent.com/ahuacate/pfsense-pfblockerng/master/images/pfblockerng_ip_01.png)

## 4.00 Configure DNSBL Settings
Because we have multiple internal interfaces, we are using a Qotom Mini PC Q500G6-S05 with 6x Gigabit NICs, you would want to protect them with DNSBL, so you will need to pay attention to the ‘Permit Firewall Rules’ section.

### 4.01 Enable DNSBL

First, place a checkmark in the ‘Enable’ box of `Permit Firewall Rules`. Then, select the various interfaces (to the right, in a box) by holding down the ‘Ctrl’ key and left-click selecting the interfaces you choose to protect with pfBlockerNG (i.e OPT1, OPT2).

Note, don’t forget to Click the `Save DNSBL settings` at the bottom of the page.

Also if your pfSense OS has plenty of memory enable, 4Gb or more, you may use TLD. Normally, DNSBL (and other DNS blackhole software) block the domains specified in the feeds and that’s that. What TLD does differently is it will block the domain specified in addition to all of a domain’s subdomains. As a result, a instrusive domain can’t circumvent the blacklist by creating a random subdomain name such as abcd1234.zuckermine.com (if zuckermine.com was in a DNSBL feed). If you have the RAM enable it - although I choose not too.

Next go to pfSense WebGUI `Firewall` > `pfBlockerNG` > `DNSBL Tab` and fill out the necessary fields as follows. Whats NOT shown in the below table leave as default. 

| DNSBL | Value | Other Values | Notes
| :---  | :--- | :--- | :---
| DNSBL | `☑` Enable | 
| TLD | `☐` Enable | | *Note: You need at least 3Gb of RAM for this feature*
| **DNSBL Webserver Configuration**
| Virtual IP Address | 10.10.10.1 || *Leave Default*
| VIP Address Type | IP Alias | Leave Blank (Enter Carp Password) | *Leave Default*
| Port | 8081 || *Leave Default*
| SSL Port | 8443 || *Leave Default*
| Webserver Interface | LAN || *Leave Default*
| **DNSBL Configuration**
| Permit Firewall Rules | `☑` Enable |`OPT1` | *Select ONLY OPT1 and OPT2 - Use the Ctrl key to toggle selection*
||| `OPT2`
| Blocked Webpage | dnsbl_default.php || *Leave Default*
| Resolver Live Sync | `☐` Enable || *Uncheck*
| **DNSBL IPs**
| List Action | `Disabled`
| Enable Logging | `Enable`

Now click `Save DNSBL settings` at the bottom of the page.

![alt text](https://raw.githubusercontent.com/ahuacate/pfsense-pfblockerng/master/images/pfblockerng_dnsbl_01.png)

### 4.02 Configure DNSBL feeds
Using the pfSense WebGUI  `Firewall` > `pfBlockerNG` > `Feeds Tab` (not DNSBL Feeds) at the top. Here you will see all of the pre-configured feeds for the IPv4, IPv6, and DNSBL categories.

Scroll down to the `DNSBL Category` header then to the Alias/Group labeled `ADs`. Click the blue colour **`+`** next to the `ADs` header (column should be all ADs) to add all the feeds related to ADs category. Note, if you instead clicked the `+` to the far right of each line, you will instead only add that individual feed - this is not what we want.

| Category | Alias/Group | Feed/Website | Header/URL
| :---  | :--- | :--- | :---
| **DNSBL Category**
| DNSBL `I` **`+`** | `ADs` | `Adaway` | `Adaway`

If you clicked the **`+`** next to the ADs category, you are taken to a `DNSBL feeds` page with all of the feeds under that category pre-populated. All of the feeds in the list will initially be in the `OFF` state. You can go through and enable each one individually or you can click `Enable All` at the bottom of the list - then all will switch/change to `ON` state. Then change the Action field to `Unbound`.

Next, make sure you switch the `Action` from Disabled to `Unbound`.

| Value | Value | Value | Value
| :---  | :--- | :--- | :---
| **DNSBL Source Definitions**
| `Auto` | `ON` | `https://adaway.org/hosts.txt` | `Adaway`
| **Settings**
| Action | `Unbound`

Now Click the `Save DNSBL Settings` at the bottom of the page and you should receive a message at the top along the lines of `Saved [ Type:DNSBL, Name:ADs ] configuration`.

To check all went okay go to the `Firewall` > `pfBlockerNG` > `DNSBL` > `DNSBL Feeds` tab and you will see a DNSBL feeds summary. Your feeds summary should look similar to the one below:

| Name | Description | Action | Frequency | Logging
| :---  | :--- | :--- | :--- | :---
| **DNSBL Feeds Summary**
| ADs | ADs - Collection | Unbound | Once a day | Enabled

Now lets add some more. Go back to `Firewall` > `pfBlockerNG` > `Feeds` tab up top and then scroll down to `DNSBL category` section again. We’re going to add another category (after making some changes), but let’s explain everything you see here. Looking at the `DNSBL Catergory` you’ll see the `ADs` category checkmark **`+`** is replaced with a :heavy_check_mark: means this category already exists and is active in the DNSBL ADs category. This distinction is important to recognize because when you add the next category we do not need to enable every feed for a particular category.

Also worth mentioning before we add the `Malicious` category. Some feeds have selectable options such as feed category `Internet Storm Center`. I recommend switching the feed from `ISC_SDH` (high) to `ISC_SDL` (low) as the high feed has under 20 entries and the low feed includes the high feed.

After making the switch to `ISC_SDL`, click the blue colour **`+`** next to the `Malicious` header (column should be all Malicious) to add all the feeds related to that category.

![alt text](https://raw.githubusercontent.com/ahuacate/pfsense-pfblockerng/master/images/pfblockerng_dnsbl_feeds_01.png)

If you clicked the **`+`** next to the Makicious category, you are taken to a `DNSBL feeds` page with all of the feeds under that category pre-populated. As when we added the ADs list, go ahead and click `Enable All` at the bottom of the list - all will switchchange to `ON` state. Then change the Action field to `Unbound`. **Don’t hit save just yet!**

**Important:** Now look for any `Header/label` called **`Pulsedive`** and/or **`Malekal`** and delete them (they were not there in my pfBlockerNG version). You don't want these as they are subscription (paid) services. On deletion they will disappear.

Now Click the `Save DNSBL Settings` at the bottom of the page and you should receive a message at the top along the lines of `Saved [ Type:DNSBL, Name:ADs ] configuration`.

![alt text](https://raw.githubusercontent.com/ahuacate/pfsense-pfblockerng/master/images/pfblockerng_dnsbl_feeds_02.png)

To check all went okay go to the `Firewall` > `pfBlockerNG` > `DNSBL` > `DNSBL Feeds` tab and you will see a DNSBL feeds summary. Your feeds summary should look similar to the one below:

| Name | Description | Action | Frequency | Logging
| :---  | :--- | :--- | :--- | :---
| **DNSBL Feeds Summary**
| ADs | ADs - Collection | Unbound | Once a day | Enabled
| Malicious | Malicious - Collection | Unbound | Once a day | Enabled

Now lets add some more. Go back to `Firewall` > `pfBlockerNG` > `Feeds` and scroll down to the `DNSBL Category` header then to the Alias/Group labeled `Easylist`. Click the blue colour **`+`** next to the `Easylist` header (column should be all Easylist) to add all the feeds related to that category. You are taken to a `DNSBL feeds` page with all of the feeds under that category pre-populated. 

**Important:** Now look for any `Header/label` called `EasyPrivacy` and delete it. On deletion the line will disappear.

All of the feeds in the list will initially be in the `OFF` state. You can go through and enable each one individually or you can click `Enable All` at the bottom of the list - all will switch/change to `ON` state. Then change the Action field to `Unbound` and the Update Frequency to `Every 4 hours`.

Now Click the `Save DNSBL Settings` at the bottom of the page and you should receive a message at the top along the lines of `Saved [ Type:DNSBL, Name:ADs ] configuration`.

Now repeat the procedure for:
*  BBcan177 - From the creater of pfBlockerBG. Make sure its the Alias/group labeled `BBcan177`.
*  hpHosts (all of them) - From Malwarebytes
*  BBC (BBC_DGA_Agr) – From Bambenek Consulting
*  Cryptojackers (all of them) – This blocks cryptojacking software and in-browser miners, but it also blocks various coin exchanges.

After adding all of the above go to the `Firewall` > `pfBlockerNG` > `DNSBL` > `DNSBL Feeds` tab and you will see a DNSBL feeds summary. Your feeds summary should look similar to the one below:

| Name | Description | Action | Frequency | Logging
| :---  | :--- | :--- | :--- | :---
| **DNSBL Feeds Summary**
| ADs | ADs - Collection | Unbound | Once a day | Enabled
| Malicious | Malicious - Collection | Unbound | Once a day | Enabled
| hpHosts | Malwarebytes - Collection | Unbound | Once a day | Enabled
| BBcan177 | BBcan177 - Collection | Unbound | Once a day | Enabled
| BBC | BBC-DGA type - Collection | Unbound | Once a day | Enabled
| Cryptojackers | Cryptojackers - Collection | Unbound | Once a day | Enabled
| Easylist | EasyList Feeds | Unbound | Every 4 hours | Enabled

### 4.03 Force DNSBL Feed Updates
You need to force a update to to `Reload` DNSBL new or changed settings. You must do this to check if your pfBlockerNG is working.

Next go to pfSense WebGUI `Firewall` > `pfBlockerNG` > `Update Tab` and fill out the necessary fields as follows. Whats NOT shown in the below table leave as default. 

| Update Settings | Value | Vale | Value | Notes
| :---  | :--- | :--- | :--- | :---
| Links 
| Select Force option | `☐` Update | `☐` Cron | `☑` Reload | *Select Reload*
| Select Reload option | `☑` All | `☐` IP | `☐` DNSBL | *Select All*

Now Click the `RUN` below the options and you should see the Logs being created on the page. It may take a while. Be patient.

## 5.00 Check if pfBlockerNG is working
First connect a device (i.e mobile, tablet etc) to either *.vpngate-local or *vpngate-world network. Go and browse a few websites like a news website. Then go to pfSense WebGUI `Firewall` > `pfBlockerNG` > `Reports` > `Alerts Tab` and you should see the DNSBL entry being populated with intercepted data. 

| Date | IF | Source | Domain/Referer/URI/Agent | Feed
| :---  | :--- | :--- | :--- | :---
| **DNSBL Section**
 Jul 26 10:49:13 [65]|OPT2|192.168.40.151|graph.instagram.com [ DNSBL ]S|Yoyo
 ||| Galaxy-Note8|DNSBL-HTTP|DNSBL_ADs
|Jul 26 11:06:05|OPT2|192.168.40.151|connect.facebook.net [ TLD ]|AntiSocial_BD
||||DNSBL-HTTPS | |DNSBL_Malicious

If you see nothing in the DNSBL section then pfBlockerNG is NOT working. Check your configurations for DNS resolve. Remember after any edits or changes always perform a pfBlockerNG Update by following the procedures in **4.03 Force DNSBL Feed Updates**.

If I am left scratching my heading wondering what I've done wrong I find deleting and recreating the pfSense firewall floating rules often fixes things. My procedure is as follows:
*  Step 1: Go to pfSense WebGUI `Firewall` > `pfBlockerNG` > `General Tab` and disable pfBlockerNG. Click `Save` at the bottom of the page.
*  Step 2: Next go to pfSense WebGUI `Firewall` > `pfBlockerNG` > `DNSBL Tab` and disable DNSBL. Click `Save` at the bottom of the page.
*  Step 3: Next go to pfSense WebGUI `Firewall` > `Rules` > `Floating Tab` and delete all 3 rules and click `Save`.
*  Step 4: Next go to pfSense WebGUI `Diagnostics` > `States` > `Reset States` select `Reset the firewall state table` and click `Reset`.
*  Step 5: Re-enable `pfBlockerNG` and `DNSBL` shown in Step 1 and 2.
*  Step 6: Now perform a pfBlockerNG Update by following the procedures in **4.03 Force DNSBL Feed Updates**.

Another MUST DO step is the creation of DNS Accept and Block Firewall rules for your network interface(s) shown [HERE](https://github.com/ahuacate/pfsense-setup/blob/master/README.md#905-dns-allow-and-block-rules-on-opt1---vpngate-world).

---

## 00.00 Patches and Fixes
