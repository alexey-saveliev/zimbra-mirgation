# How to migrate Zimbra OSE to new server
Steps below was tested with Zimbra OSE 8.8.12GA on Ubuntu 18.04.
## Moving steps
1. Install **new server** OS  the same version and pathc level as on ***old server***
1. Install **new server** Zimbra OSE exactly the same version as on ***old 
server***
    ```
    # Run command below as root
    # It will install Zimbra OSE software only
    # You'll must to choose same component set as on the old server
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
 ## Possible problems
- Got error messages while run `install.sh`
    1. _Error:_ **UUID.c**: loadable library and perl binaries are mismatched (got handshake key 0xdb00080, needed 0xde00080)
        Run `apt install --reinstall zimbra-perl-data-uuid` to solve.

    2. _Error:_ **Linux.c**: loadable library and perl binaries are mismatched (got handshake key 0xdb00080, needed 0xde00080)
        Run `apt install --reinstall zimbra-perl-socket-linux` to solve.
- Log file `/opt/zimbra/zmstat/zmstat.out` full of message like  `$line in pattern match (m//) at /opt/zimbra/libexec/zmstat-io line 87.`
        `sed -i s/Device\:/Device/g /tmp/zmstat-io` may solve the problem