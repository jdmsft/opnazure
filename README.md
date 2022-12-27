# OPNsense NVA on Azure FreeBSD VM

This OPNsense solution is installed in FreeBSD (Azure Image).

Design of two Nic deployment | Design of Active-Active deployment |
|--------|--------|
|![opnsense design](./images/two-nics.png)|![opnsense design](./images/active-active.png)|

Here is what you will see when you deploy this Template:

There are 3 different deployment scenarios:

1. Active-Active:
  * VNET with Two Subnets and OPNsense VM with two NICs.
  * VNET Address space is: 10.0.0.0/16 (suggested Address space, you may change that).
  * External NIC named Untrusted Linked to Untrusted-Subnet (10.0.0.0/24).
  * Internal NIC named Trusted Linked to Trusted-Subnet (10.0.1.0/24).
  * It creates a NSG named OPN-NSG which allows incoming SSH and HTTPS. Same NSG is associated to both Subnets.
  * Active-Active a Internal and External loadbalancer will be created.
  * Two OPNsense firewalls will be created.
  * OPNsense will be configured to allow loadbalancer probe connection.
  * OPNsense HA settings will be configured to sync rules changed between both Firewalls.
  * Option to deploy Windows management VM. (This option requires a management subnet to be created)

2. TwoNics:
  * VNET with Two Subnets and OPNsense VM with two NICs.
  * VNET Address space is: 10.0.0.0/16 (suggested Address space, you may change that).
  * External NIC named Untrusted Linked to Untrusted-Subnet (10.0.0.0/24).
  * Internal NIC named Trusted Linked to Trusted-Subnet (10.0.1.0/24).
  * It creates a NSG named OPN-NSG which allows incoming SSH and HTTPS. Same NSG is associated to both Subnets.
  * Option to deploy Windows management VM. (This option requires a management subnet to be created)

3. SingleNic:
  * VNET with single Subnet and OPNsense VM with single NIC.
  * VNET Address space is: 10.0.0.0/16 (suggested Address space, you may change that).
  * External NIC named Untrusted Linked to Untrusted-Subnet (10.0.0.0/24).
  * It creates a NSG named OPN-NSG which allows incoming SSH and HTTPS.
  * Option to deploy Windows management VM. (This option requires a management subnet to be created)

## Deployment

[![Deploy To Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.svg?sanitize=true)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjdmsft%2Fopnazure%2Fmaster%2FARM%2Fmain.json%3F/uiFormDefinitionUri/https%3A%2F%2Fraw.githubusercontent.com%2Fjdmsft%2Fopnazure%2Fmaster%2FARM%2FuiFormDefinition.json)

The template allows you to deploy an OPNsense Firewall VM using the opnsense-bootsrtap installation method. It creates an FreeBSD VM, does a silent install of OPNsense using a modified version of opnsense-bootstrap.sh with the settings provided.

The login credentials are set during the installation process to:

- Username: root
- Password: opnsense (lowercase)

*** **Please** *** Change *default password!!!* (In case of using Active-Active scenario the password must be changed in both Firewalls and under Highavailability settings)

After deployment, you can go to <https://PublicIP>, then input the user and password, to configure the OPNsense firewall.
In case of Active-Active the URL should be <https://PublicIP:50443> for Primary server and <https://PublicIP:50444> for Secondary server.


### Considerations

Here are few considerations to deploy this solution correctly:

- When you deploy this template, it will leave only TCP 22 listening to Internet while OPNsense gets installed.
- To monitor the installation process during template deployment you can just probe the port 22 on OPNsense VM public IP (psping or tcping).
- When port is down which means OPNsense is installed and VM will get restarted automatically. At this point you will have only TCP 443.

**Note**: It takes about 10 min to complete the whole process when VM is created and a new VM CustomScript is started to install OPNsense.

### Post-deployment

- First access can be done using <HTTPS://PublicIP.> Please ignore SSL/TLS errors and proceed. In case of Active-Active the URL should be <https://PublicIP:50443> for Primary server and <https://PublicIP:50444> for Secondary server.
- Your first login is going to be username "root" and password "opnsense" (**PLEASE change your password right the way**).
- To access SSH you can either deploy a Jumpbox VM on Trusted Subnet or create a Firewall Rule to allow SSH to Internet.
- To send traffic to OPNsense you need to create UDR 0.0.0.0 and set IP of trusted NIC IP (10.0.1.4) as next hop. Associate that NVA to Trusted-Subnet.
- **Note:** It is necessary to create appropriate Firewall rules inside OPNsense to desired traffic to work properly.

## Release note

### October-2022
- Updated FreeBSD to 13.1
- Updated OPNSense to 22.7
- Updated Azure Linux Agent to 2.8.0
- Updated Python symbolic link to 3.9

### April-2022
- Updated FreeBSD 13 and OPNSense 22.1
- Added support for Floating IPs in External Load Balance Rules to allow Port Forwarding without causing assymetric issues.
- Enabled session Sync between Firewalls.
- Add Virtual IP of the External Load Balancer to support Floating Rules.
- Add support for a Windows Management VM in a management network.
- Create a new simplified deployment wizard.
- Bicep template refactory to support the new UI deployment wizard.

### Nov-2021
- Added Active-Active deployment option (using Azure Internal and External Loadbalancer and OPNsense HA settings).
- Templates are now auto-generated under the folder ARM from a Bicep template using Github Actions.

## Credits

This project is a fork of <https://github.com/dmauser/opnazure>