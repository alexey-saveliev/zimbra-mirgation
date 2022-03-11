# How to migrate Zimbra OSE to new server
Steps below was tested with Zimbra OSE 8.8.12GA on Ubuntu 18.04.
## Moving steps
1. Set TTL value for Zimbra server name and MX servers of domains served by Zimbra as low as possible
1. Install **new server** OS  the same version and patch level as on ***old server***
1. Install **new server** Zimbra OSE exactly the same version as on ***old 
server***
    ```
    # Run command below as root
    # It will install Zimbra OSE software only
    # You must choose same component set as on the old server
    ./install.sh -s
    ```
1. On the **new server** move newly installed Zimbra to temp directory
    ```
    mv /opt/zimbra /opt/zimbra.fresinstall
    ```
1. Make initial sync of Zimbra directory to **new server**
    ```
    # Run command below as root on new server 
    rsync -e ssh -axvzKHS  old_server:/opt/zimbra /opt/
    ```
1. Fix permissions on the **new server**
    ```
    # Run command below as root on new server 
    /opt/zimbra/libexec/zmfixperms -e -v
    ```
1. As root rerun the installer without the -s option:
    ```
    ./install.sh 
    ```
    The installer will detect ZCS already installed, and will ask if you want to upgrade. Select Yes.
1. Verify and redeploy SSL certificate and restart Zimbra
    ```
    # Run commands below as zimbra on new server 
    zmcertmgr verifycrt comm
    zmcertmgr deploycrt comm
    zmcontrol restart
    ```
Check if moved Zimbra works as expected. Now you are ready to final move.
1. Stop Zimbra on the ***old server***
    ```
    # Run as root on the old server
    su - zimbra -c 'zmcontrol stop'
    ```
1. On the **new server**
    ```
    # Run command below as root on new server 
    rsync -e ssh -axvzKHS  old_server:/opt/zimbra /opt/
    /opt/zimbra/libexec/zmfixperms -e -v
    ```
1. If you've faced any errors while running `install.sh` on previous run you need to fix it now. (See [Possible problems](https://github.com/alexey-saveliev/zimbra-mirgation#possible-problems) section)
1. As root rerun the installer without the -s option:
    ```
    ./install.sh 
    ```
    The installer will detect ZCS already installed, and will ask if you want to upgrade. Select Yes.
1. Verify and redeploy SSL certificate and restart Zimbra
    ```
    # Run commands below as zimbra on new server 
    zmcertmgr verifycrt comm
    zmcertmgr deploycrt comm
    zmcontrol restart
    ```
1. Check if moved Zimbra works as expected and set DNS records point to new server


 ## Possible problems
- Got error messages while run `install.sh`
    1. _Error:_ **UUID.c**: loadable library and perl binaries are mismatched (got handshake key 0xdb00080, needed 0xde00080)
        Run `apt install --reinstall zimbra-perl-data-uuid` to solve.

    2. _Error:_ **Linux.c**: loadable library and perl binaries are mismatched (got handshake key 0xdb00080, needed 0xde00080)
        Run `apt install --reinstall zimbra-perl-socket-linux` to solve.
- Log file `/opt/zimbra/zmstat/zmstat.out` full of message like  `$line in pattern match (m//) at /opt/zimbra/libexec/zmstat-io line 87.`
        `sed -i s/Device\:/Device/g /tmp/zmstat-io` may solve the problem