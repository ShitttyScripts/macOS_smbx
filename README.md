# macOS_smbx
Apple macOS smbx information

Things that might make smbx suck a little less and scripts to deploy them.

Each one of the items in this document can be deployed with one of the scripts in the [in the Scripts directory](https://github.com/ShitttyScripts/macOS_smbx/tree/master/Scripts) of this reposatory.

## DO NOT TEST IN PRODUCTION

This is a compalation of data from Apple, multiple storage vendors, and the internet at large, of various issues and workarounds to make Apple's smbx implimentation play a little nicer.

# Disable directory caching - macOS 10.13+

https://support.apple.com/en-us/HT208209

https://www.dellemc.com/resources/en-us/asset/white-papers/products/storage/h17613_wp_isilon_mac_os_performance_optimization.pdf - Page 10

macOS caches file and folder metadata temporarily in local memory. This improves browsing speeds, especially on high-latency networks. Systems with more memory installed can cache more file information.

# Disable file and icon previews in Finder

https://www.jamf.com/jamf-nation/discussions/24619/disabling-icon-preview

https://www.google.com/search?q=mac+smb+preview+file+lock&oq=mac+smb+preview+file+lock&aqs=chrome..69i57j69i64.7547j0j7&sourceid=chrome&ie=UTF-8

Having previews enabled for files and icons in different Finder view settings can result in file lock/permissions errors for all other users on the same share. This is a result of the interaction between the way macOS generates the preview and the status that sets on the file.

Set the cover-flow preview setting to off
Set the icon preview setting to off
Set the list icon preview setting to off
Set the column icon preview setting to off
Set the column preview column setting to off

## Disable SMB packet signing - macOS 10.13.3 and earlier

https://support.apple.com/en-us/HT205926

https://www.dellemc.com/resources/en-us/asset/white-papers/products/storage/h17613_wp_isilon_mac_os_performance_optimization.pdf - Page 9

### In macOS 10.13.4 and later, packet signing is off by default. Packet signing for SMB 2 or SMB 3 connections turns on automatically when needed if the server offers it. The instructions in this article apply to macOS 10.13.3 and earlier. When using an SMB 2 or SMB 3 connection, packet signing is turned on by default.

You might want to turn off packet signing if:
- Performance decreases when you connect to a third### party server.
- You can’t connect to a server that doesn’t support packet signing.
- You can’t connect a third### party device to your macOS SMB server.

## Disable SMB session signing - macOS 10.13+ and SMB3

https://support.apple.com/en-us/HT205926

https://www.dellemc.com/resources/en-us/asset/white-papers/products/storage/h17613_wp_isilon_mac_os_performance_optimization.pdf - Page 9

### SMB3 introduces security enhancements that help prevent man### in### the### middle attacks during the initiation of an SMB client connection to an SMB server. This is different from SMB signing which adds a digital signature to each packet. SMB session signing can be described as a way to protect an SMB session from being tampering with as it commences. In certain scenarios, particularly where macOS client systems are bound anonymously to a directory server, this may cause authentication errors when a system tries to connect to an SMB share.

In situations where proper network credentials are not working from macOS systems running version 10.13+, disabling SMB session signing may resolve the issue. Similar to disabling SMB signing, this reduces the security of an SMB connection and is recommended on systems running on private, secure networks.

## Disable writing of .DS_Store files to network shares

https://support.apple.com/en-us/HT208209

https://www.dellemc.com/resources/en-us/asset/white-papers/products/storage/h17613_wp_isilon_mac_os_performance_optimization.pdf - Page 8

### Speeds up SMB file browsing by preventing macOS from reading .DS_Store files on SMB shares. This makes the Finder use only basic information to immediately display each folder's contents in alphanumeric order.

## Configure Finder to gather all metadata before displaying content - macOS 10.13+

https://support.apple.com/en-us/HT208209

https://www.dellemc.com/resources/en-us/asset/white-papers/products/storage/h17613_wp_isilon_mac_os_performance_optimization.pdf - Page 8

### Configure the Finder in macOS 10.13 High Sierra and later to always collect complete metadata before displaying folder contents, matching the file browsing behavior of macOS 10.12 Sierra and earlier.

## TCP delayed acknowledgement (delayed_ack)

https://www.dellemc.com/resources/en-us/asset/white-papers/products/storage/h17613_wp_isilon_mac_os_performance_optimization.pdf - Page 7

http://www.stuartcheshire.org/papers/NagleDelayedAck/ 

### There are many explanations for TCP delayed acknowledgement (delayed_ack), such as the description in the article TCP Performance problems caused by interaction between Nagle’s Algorithm and Delayed ACK. For most uses of TCP, delayed acknowledgement is a good thing and makes network communication more efficient. TCP acknowledgements are added to subsequent data packets. However, in practice on macOS, particularly with network connections lower than 1 GbE, TCP delayed There are four options in macOS for setting the characteristics of acknowledgement can cause performance degradation.

There are four options in macOS for setting the characteristics of delayed_ack:

- delayed_ack=0: Responds after every packet (OFF)
- delayed_ack=1: Always employs delayed_ack; 6 packets can get 1 ack- 
- delayed_ack=2: Immediate ack after 2nd packet; 2 packets per ack (Compatibility Mode)
- delayed_ack=3: Should auto detect when to employ delayed ack; 3 packets per ack

The default setting for macOS clients is delayed_ack=3. However, testing has shown that in some environments — particularly where the client is reading and writing simultaneously (such as what happens when an application is rendering a video sequence) — changing the setting to delayed_ack=0 significantly improves performance. It should be noted that environments vary, and that the settings should be tested in each environment. Sometimes, a setting of delayed_ack=1 or delayed_ack=2 will work best. It is important to also understand the impact of the change on other parts of the system.
