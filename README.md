# Dell-SONIC-ZTP-with-TPCM
> Instructions and sample files for Dell SONIC ZTP (Zero Touch Provisioning) with DHCP and TFTP as Docker containers

## TABLE of CONTENTS
[General Information](#general-information)  
[Cautionary Notes](#cautionary-notes)  
[Reference Documents](#reference-documents)  
[Software Versions](#Software-Versions)  
[Prep Switch for DHCP and TFTP Containers](#prep-switch-for-dhcp-and-tftp-containers)  
[Install DHCP and TFTP Containers on Dell SONiC Switch](#install-dhcp-and-tftp-containers-on-dell-sonic-switch)  
[Auto Remove OS10 and Install SONiC](#auto-remove-os10-and-install-sonic)  
[Dell SONiC ZTP](#dell-sonic-ztp) 
[DHCP Setup](#dhcp-setup)  
[ztp-ent-stand.json and ztp-lite.json files](#ztp-ent-stand.json-and-ztp-lite.json-files)  
[Switch Specific config_db.json files](#switch-specific-config_db.json-files)  

### General Information [(Goto TOC)](#table-of-contents)  
- Instructions and sample files to show the operation of Dell SONiC ZTP (Zero Touch Provisioning) using TPCM (Third Party Container Management) to install basic docker conatainers for DHCP and TFTP services on a Dell SONiC switch.  Prior to the infrastructure being setup the MAC address of each switch needs to be recorded either from the label on the cardboard or the pull out tab from each switch.  The MAC address will be leveraged in the DHCP scope to assign specific json switch configuration files to each specific switch.  Some Dell Ethernet switch models like the S5448 ship from the factory with legacy OS10 switch softare installed.  DHCP and ONIE can be leveraged to automatically remove OS10 and install SONiC.

### Cautionary Notes  [(Goto TOC)](#table-of-contents)  
- All references to SONiC refer to Dell SONiC which is slightly different from the Community version of SONiC.
- Dell Enterprsie SONiC (DES) is intended to run on Dell as well as a small number of 3rd party Ethernet switches.
- ZTP and ZTD are used interchangably and refer to the same automated process
- On a test switch the SONIC-CLI should be used to create the basic switch running-config ( /etc/sonic/config_db.json ). Dell recommends NOT to directly edit the switch config_db.json file as there are often many interdependencies and also there is no error checking.

### Reference Documents [(Goto TOC)](#table-of-contents)  
- OS10 Third Party NOS Install Guide = detailed information for automatically uninstalling OS10 and installing a third party switch OS like Dell SONiC
- ZTD SONiC white paper = detaied information for Dell SONiC Zero Touch Deployment

### Software Versions    
  [(Goto TOC)](#table-of-contents)  
- Dell SONiC version 4.4.0

### Prep Switch for DHCP and TFTP Containers  [(Goto TOC)](#table-of-contents)  
- This step can be skipped if DHCP and TFTP servers already exist and you do not wish to install them on a Dell switch  

- Create ZTP directory structure on switch running DHCP and TFTP containers, change permissions, and SCP files
```    
  $ sudo mkdir /var/tftpboot/ ; default tftp download directory
  $ sudo mkdir /home/admin/data/ ; exact directory needed for tftp docker container and TPCM
``` 
- change ownership and permission on apache2 http server or TFTP server
```
  $ sudo chown -R admin /var/www/html/
  $ sudo chwn -R admin /var/tftpboot/
  $ sudo chown -R admin /home/admin/data/
```
- change permissions so admin owner can read, write, execute, and group can read and execute
``` 
  $sudo chmod -R 755 /var/tftpboot/
  $sudo chmod -R 755 /var/www/html/
  $sudo chmod -R 755 /home/admin/data/ ; highest permission level if needed
```
- Use WinSCP or linux bash shell to transfer binary and json files for ZTP delivery  
    - transfer sonic-ent-stand-4.4.bin (s5448) and sonic-lite- 4.4.bin (e3248P) and transfer onie-installer.bin (OS10 switches) to /var/tftpboot/ for tftp transfers  
    - onie-installer.bin is same file as sonic-ent-stand-4.4.bin except filename different  
    - In the ztp-ent-stand.json (s5448) or ztp-lite.json (e3248P) firmware stanza If the SONIC version equals the running version on the switch it will only download the image and silently discard the image  
    - ztp-ent-stand.json and ztp-lite.json must be different in order to push different SONIC images to different switch models  
    - -v verbose, -r recursive all files -C compression can be used to speed up transfer   
    $ scp -v -r -C /home/tom/ztp/ admin@192.168.10.248:/var/tftpboot/  

  - SCP binary files per below if using Apache2 or similar HTTP server instead of TFTP  
    $ scp -v -r -C /home/tom/ztp/ admin@192.168.10.248:/var/www/html/  

  - create/copy unique config_db.json for each switch and simply edit management0 IP address  
  - each config_db.json is a bare minimum config to minimize keyboard typing and ensure IP reachability  
  - full switch config delivered by Ansible or similar automation software after ZTP completes
```
    /var/tftpboot/running-configs/n3248pxetop_config_db.json  
    /var/tftpboot/running-configs/s5248bot_config_db.json  
    /var/tftpboot/running-configs/s5248new_config_db.json
```
  - copy dhcpd.conf file to TPCM switch /home/admin/data/ directory  
  - if dhcpd.conf is not copied before dhcp docker container installed the service will fail  
    /home/admin/data/dhcpd.conf  

### Install DHCP and TFTP Containers on Dell SONiC Switch  [(Goto TOC)](#table-of-contents)  
  - Dell SONiC TPCM Tips and Tricks  
  - TPCM container is not installed by default with SONIC base image  
  - All third party containers automatically start during fast, warm, and cold system reboots. After TPC image is loaded upon reboots TPC service starts automatically by the system manager  
  - A SONIC image upgrade seamlessly migrates all currently installed third-party containers into the newly installed SONIC image  
  - A TPC image can be ugraded independently of a SONIC image upgrade  
  - You can install more than one light weight TPC container image on SONIC  
  - By default TPC uses the default VRF to see user ports and OOB ports for DHCP services  
  - For ease of configuration don’t use vrf mgmt. for management0  
  - if only desire dhcp services on OOB network then using vrf mgmt can be used  
  - If dhcp TPC is not on same layer 2 network then configure DHCP relay (ip helper as needed)  

  - TFTP container installation and auto start  
    - docker “args” = used to publish the container’s port to the switch to allow external connections as well as the switch installation directory  
    - if the switch has a vrf mgmt then must specify that as well  
    - may have to run command more than once if fails the first time until receive SUCCESS  
      S5448top1# tpcm install name tftpd pull pghalliday/tftp args "-p 0.0.0.0:69:69/udp -v /var/tftpboot:/var/tftpboot --network=host"  
    - OR if using vrf mgmt  
      S5448top1# tpcm install name tftpd pull pghalliday/tftp args "-p 0.0.0.0:69:69/udp -v /var/tftpboot:/var/tftpboot --network=host"  vrf-name mgmt  

      Pulling the TPC-tftpd image  
      Auto starting the TPC-tftpd  
      [ SUCCESS ] Installation complete  

  - DHCP container installation and auto start  
    - container will advertise automatically on the OOB port <mgmt vrf> without specifying <mgmt vrf> with command parameters  
      S5248F-ON-01# tpcm install name isc_dhcp pull networkboot/dhcpd args "-v /home/admin/data:/data --network=host --security-opt seccomp=unconfined –memory=1000M"  

      mgmt vrf is enabled, hence pulling through mgmt vrf  
      Pulling the TPC-isc_dhcp image  
      Auto starting the TPC-isc_dhcp  
      [ SUCCESS ] Installation complete  

  - Troubleshooting commands from sonic-cli  
      S5448top1# show tpcm list  
      S5448top1# show tpcm name isc_dhcp  
      S5448top1# ls /var/tftpboot ; same as SONIC shell $ ls /var/tftpboot/  

  - Troubleshooting commands from bash-shell  
      $ systemctl status  
      $ docker images ls -a  
      $ docker images list -all  
      $ cat /var/log/syslog | grep dhcp | more ; review messages for troubleshooting  
      $ docker ps  
      $ sudo docker ps -all  
      $ sudo docker ps -a  
      $ docker ps --format "table {{.Image}}\t{{.Names}}\t{{.Size}}"  
      $ docker logs <container id>  
      $ sudo tpcm list  
      $ sudo docker restart tftpd ; better to use docker ID instead of name  
      $ sudo docker restart isc-dhcp ; better to use the docker ID instead of name  
      $ sudo docker restart <container id>  
      $ docker exec -it tftpd /bin/sh  

### Auto Remove OS10 and Install SONiC  [(Goto TOC)](#table-of-contents)  
- Some Dell Ethernet switches ship with legacy OS10. The DHCP scope sample file can be used to automatically remove OS10 and install SONiC  
- Once SONiC is pushed to the switch the switch will reboot and begin the SONiC ZTP process  
- OS10 ZTD with DHCP option 67 (TFTP boot filename) and 114 (default URL) must be received thru the switch mgmt port not the front panel ports  
- Option 240 (ztd provision url script) can be received via mgmt port and front panel ports  
- Option 114  with "onie-installer" and URL in DHCP offer system does the following  
    - boots to ONIE prompt  
    - uninstalls OS10 image (takes about 10 min to remove partition  
    - processes the URL from DHCP scope  
    - installs SONIC which defaults to ZTD mode  
    - the complete process takes 5-10 minutes since the switch reboots a few times but it is fully automated  
    - auto method via DHCP docker container http://192.168.10.200/onie-installer.bin  
    - must rename SONIC binary to "onie-installer.bin" per the file naming convention and waterfall effect in OS10 3rd party NOS install doc  

### Dell SONiC ZTP  [(Goto TOC)](#table-of-contents)  
- The MAC address should be recorded or scanned during install.  It is located on the outside of the switch cardboard box and also on the switch pull out plastic tag.  
- ZTP will push a basic running-config file and the anchor image version of the SONiC binary to each switch to provide network reachability  
- On a test switch the SONIC-CLI should be used to create the basic switch running-config ( /etc/sonic/config_db.json ). Dell recommends NOT to directly edit the switch config_db.json file as there are often many interdependencies and also there is no error checking.  
- Then Ansible or some other automation tool can manage the complete switch config for each individual switch
  
### DHCP setup  [(Goto TOC)](#table-of-contents)  
- The DHCP server (Dell switch with TPCM or other) must be setup for directories, permissions, files per above [Prep Switch for DHCP and TFTP Containers](#prep-switch-for-dhcp-and-tftp-containers)  
- The dhcpd.conf will only create a fixed IP address for the switch OOB port, Management0 and push the proper SONiC software version  
- In the case of a switch which is preloaded with OS10 the onie-installer.bin file needs to have this filename is a renamed version of the SONiC binary based on switch model.  
- The sample dhcpd.conf is separated into groups to push the proper SONiC binary to specific switches based on switch model type.  

### ztp-ent-stand.json and ztp-lite.json files [(Goto TOC)](#table-of-contents)   
- During Dell SONiC ZTP the ztp.json provides the proper SONiC binary version based on switch model and a dymanic URL for a switch specific running-config file (config_db.sjon)  
- This example uses ztp-ent-stand.json (S5248) and ztp-lite.json (E3248P) because each switch model must receive a different SONiC binary  
- If the SONIC binary version equals the running version on the switch it will only download the image and silently discard the image  

### Switch Specific config_db.json files  [(Goto TOC)](#table-of-contents)  
- The TFTP server (Dell switch with TPCM or other) must be setup for directories, permissions, files per above [Prep Switch for DHCP and TFTP Containers](#prep-switch-for-dhcp-and-tftp-containers)  
- Each switch will have a unique config_db.json with the hostname as the prefix  
- The <hostname>_config_db_.json will be copied to each switch /etc/sonic/config_db.json  
- The objective of this file is to make it as basic as possible to provide IP reachable.  This minimizes keystrokes for creating each unique switch .json file  
- Once each switch has basic IP reachability another tool such as Ansible or similar will be used to provide the full switch config based on switch model, function, etc  
