# Dell-SONIC-ZTP-with-TPCM
> Instructions and sample files for Dell SONIC ZTP (Zero Touch Provisioning)

## TABLE of CONTENTS
- [General Information](#general-information)
- [Reference Documents](#reference-documents)
- [Software Versions](#software-versions)

### General Information
- Instructions and sample files to show the operation of Dell SONiC ZTP (Zero Touch Provisioning) using TPCM (Third Party Container Management) to install basic docker conatainers for DHCP and TFTP services on a Dell SONiC switch.  Prior to the infrastructure being setup the MAC address of each switch needs to be recorded either from the label on the cardboard or the pull out tab from each switch.  The MAC address will be leveraged in the DHCP scope to assign specific json switch configuration files to each specific switch.  Some Dell Ethernet switch models ship from the factory with legacy OS10 switch softare installed.  DHCP and ONIE can be leveraged to automatically remove OS10 and install SONiC.

### Reference Documents
- OS10 install guide

### Software Versions
- Dell SONiC version 
