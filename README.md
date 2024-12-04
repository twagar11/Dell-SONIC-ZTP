# Dell-SONIC-ZTP-with-TPCM
> Instructions and sample files for Dell SONIC ZTP (Zero Touch Provisioning)

## TABLE of CONTENTS
- [General Information](#general-information)
- [Cautionary Notes](#cautionary-notes)
- [Reference Documents](#reference-documents)
- [Software Versions](#software-versions)
- [Auto Remove OS10 and Install SONiC](#auto-remove-os10-and-install-sonic)

### General Information
- Instructions and sample files to show the operation of Dell SONiC ZTP (Zero Touch Provisioning) using TPCM (Third Party Container Management) to install basic docker conatainers for DHCP and TFTP services on a Dell SONiC switch.  Prior to the infrastructure being setup the MAC address of each switch needs to be recorded either from the label on the cardboard or the pull out tab from each switch.  The MAC address will be leveraged in the DHCP scope to assign specific json switch configuration files to each specific switch.  Some Dell Ethernet switch models ship from the factory with legacy OS10 switch softare installed.  DHCP and ONIE can be leveraged to automatically remove OS10 and install SONiC.

### Cautionary Notes
- All references to SONiC refer to Dell SONiC which is slightly different from the Community version of SONiC.  Dell Enterprsie SONiC (DES) is intended to run on Dell as well as a small number of 3rd party Ethernet switches.
- ZTP and ZTD are used interchangably and refer to the same automated process

### Reference Documents
- OS10 Third Party NOS Install Guide = detailed information for automatically uninstalling OS10 and installing a third party switch OS like Dell SONiC
- ZTD SONiC white paper = detaied information for Dell SONiC Zero Touch Deployment

### Software Versions
- Dell SONiC version

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
    - auto method via DHCP docker container http://192.168.10.200/onie-installer.bin
    - must rename SONIC binary to "onie-installer.bin" per the file naming convention and waterfall effect in OS10 3rd party NOS install doc

