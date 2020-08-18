<!--markdown-->#### 自定义名称解析
[wireshark手册](https://www.wireshark.org/docs/wsug_html_chunked/ChAppFilesConfigurationSection.html)

On Windows:
- The personal configuration folder for Wireshark is the Wireshark sub-folder of that folder, i.e. %APPDATA%\\Wireshark.
- The global configuration folder for Wireshark is the Wireshark program folder and is also used as the system configuration folder. 

On Unix-like systems:
- The personal configuration folder is $XDG_CONFIG_HOME/wireshark. For backwards compatibility with Wireshark before 2.2, if $XDG_CONFIG_HOME/wireshark does not exist and $HOME/.wireshark is present, then the latter will be used.
- If you are using macOS and you are running a copy of Wireshark installed as an application bundle, the global configuration folder is APPDIR/Contents/Resources/share/wireshark. Otherwise, the global configuration folder is INSTALLDIR/share/wireshark.
- The /etc folder is the system configuration folder. The folder actually used on your system may vary, maybe something like: /usr/local/etc.

**ethers**

    When Wireshark is trying to translate an hardware MAC address to a name, it consults the ethers file in the personal configuration folder first. If the address is not found in that file, Wireshark consults the ethers file in the system configuration folder.

    Each line in these files consists of one hardware address and name separated by whitespace. The digits of hardware addresses are separated by colons (:), dashes (-) or periods(.). The following are some examples:

    ff-ff-ff-ff-ff-ff    Broadcast
    c0-00-ff-ff-ff-ff    TR_broadcast
    00.2b.08.93.4b.a1    Freds_machine

    The settings from this file are read in when a MAC address is to be translated to a name, and never written by Wireshark.
    
**hosts**

    Wireshark uses the entries in the hosts files to translate IPv4 and IPv6 addresses into names.

    At program start, if there is a hosts file in the global configuration folder, it is read first. Then, if there is a hosts file in the personal configuration folder, that is read; if there is an entry for a given IP address in both files, the setting in the personal hosts file overrides the entry in the global hosts file.

    This file has the same format as the usual /etc/hosts file on Unix systems.

    An example is:

    # Comments must be prepended by the # sign!
    192.168.0.1 homeserver

    The settings from this file are read in at program start and never written by Wireshark.

 	