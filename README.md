# Dell-SONIC-ZTP-with-TPCM
> Instructions and sample files for Dell SONIC ZTP (Zero Touch Provisioning)

## TABLE of CONTENTS
- [General Information](#general-information)
- [Cautionary Notes](#cautionary-notes)
- [Reference Documents](#reference-documents)
- [Software Versions](#software-versions)
- [Prep Switch for DHCP and TFTP Containers](#prep-switch-for-dhcp-and-tftp-containers)
- [Auto Remove OS10 and Install SONiC](#auto-remove-os10-and-install-sonic)
- [Dell SONiC ZTP](#dell-sonic-ztp)

### General Information
- Instructions and sample files to show the operation of Dell SONiC ZTP (Zero Touch Provisioning) using TPCM (Third Party Container Management) to install basic docker conatainers for DHCP and TFTP services on a Dell SONiC switch.  Prior to the infrastructure being setup the MAC address of each switch needs to be recorded either from the label on the cardboard or the pull out tab from each switch.  The MAC address will be leveraged in the DHCP scope to assign specific json switch configuration files to each specific switch.  Some Dell Ethernet switch models like the S5448 ship from the factory with legacy OS10 switch softare installed.  DHCP and ONIE can be leveraged to automatically remove OS10 and install SONiC.

### Cautionary Notes
- All references to SONiC refer to Dell SONiC which is slightly different from the Community version of SONiC.  Dell Enterprsie SONiC (DES) is intended to run on Dell as well as a small number of 3rd party Ethernet switches.
- ZTP and ZTD are used interchangably and refer to the same automated process

### Reference Documents
- OS10 Third Party NOS Install Guide = detailed information for automatically uninstalling OS10 and installing a third party switch OS like Dell SONiC
- ZTD SONiC white paper = detaied information for Dell SONiC Zero Touch Deployment

### Software Versions
- Dell SONiC version 4.4.0

### Prep Switch for DHCP and TFTP Containers
- This step can be skipped if DHCP and TFTP servers already exist and you do not wish to install them on a Dell switch

- Create ZTP directory structure on switch running DHCP and TFTP containers, change permissions, and SCP files
  - ! create /var/tftpboot/ root directory for tftp server
    $ sudo mkdir /var/tftpboot/ ; default tftp download directory
    $sudo mkdir /home/admin/data/ ; exact directory needed for tftp docker container and TPCM

  - ! change ownership and permission on apache2 http server or TFTP server
    $ sudo chown -R admin /var/www/html/
    $ sudo chown -R admin /var/tftpboot/
    $ sudo chown -R admin /home/admin/data/

  - ! change permissions so admin owner can read, write, execute, and group can read and execute 
    $sudo chmod -R 755 /var/tftpboot/
    $sudo chmod -R 755 /var/www/html/
    $sudo chmod -R 755 /home/admin/data/ ; highest permission level if needed

  - ! use WinSCP or linux bash shell to transfer binary and json files for ZTP delivery
    ! transfer sonic-ent-stand-4.4.bin (s5448) and sonic-lite- 4.4.bin (e3248P) and transfer onie-installer.bin (OS10 switches) to /var/tftpboot/ for tftp transfers
    ! onie-installer.bin is same file as sonic-ent-stand-4.4.bin except filename different
    ! In the ztp-ent-stand.json (s5448) or ztp-lite.json (e3248P) firmware stanza If the SONIC version equals the running version on the switch
    ! it will only download the image and silently discard the firmware
    ! ztp-ent-stand.json and ztp-lite.json must be different in order to push different SONIC images to different switch models
    ! -v verbose, -r recursive all files -C compression can be used to speed up transfer
    $ scp -v -r -C /home/tom/ztp/ admin@192.168.10.248:/var/tftpboot/

  - ! SCP binary files per below if using Apache2 or similar HTTP server instead of TFTP
    $ scp -v -r -C /home/tom/ztp/ admin@192.168.10.248:/var/www/html/

  - ! create/copy unique config_db.json for each switch and simply edit management0 IP address
    ! each config_db.json is a bare minimum config to minimize keyboard typing and ensure IP reachability
    ! full switch config delivered by Ansible or similar automation software after ZTP completes
    /var/tftpboot/running-configs/n3248pxetop_config_db.json
    /var/tftpboot/running-configs/s5248bot_config_db.json
    /var/tftpboot/running-configs/s5248new_config_db.json

  - ! copy dhcpd.conf file to TPCM switch /home/admin/data/ directory
    ! if dhcpd.conf is not copied before dhcp docker container installed the service will fail
    /home/admin/data/dhcpd.conf

### Auto Remove OS10 and Install SONiC
- Some Dell Ethernet switches ship with legacy OS10. The DHCP scope sample file can be used to automatically remove OS10 and install SONiC
  - Once SONiC is pushed to the switch the switch will reboot and begin the SONiC ZTP process
  - OS10 ZTD with DHCP option 67 (TFTP boot filename) and 114 (default URL) must be received thru the switch mgmt port not the front panel ports
  - Option 240 (ztd provision url script) can be received via mgmt port and front panel ports
  - Option 114  with "onie-installer" and URL in DHCP offer system does the following
    - boots to ONIE prompt
    - uninstalls OS10 image (takes about 10 min to remove partition)
    - processes the URL from DHCP scope
    - installs SONIC which defaults to ZTD mode
    - the complete process takes 5-10 minutes since the switch reboots a few times but it is fully automated
    - auto method via DHCP docker container http://192.168.10.200/onie-installer.bin
    - must rename SONIC binary to "onie-installer.bin" per the file naming convention and waterfall effect in OS10 3rd party NOS install doc

### Dell SONiC ZTP
- ZTP will push a basic config_db.json config file and the anchor image of SONiC binary to each switch to provide network reachability
- On a test switch the SONIC-CLI should be used to create the basic switch config which pushes to the switch /etc/sonic/config_db.json
- Then Ansible or some other automation tool can manage the complete switch config for each individual switch
- The MAC address should be recorded or scanned during install.  They are on the outside of the cardboard boxes and also on the switch pull out plastic tag
- 
