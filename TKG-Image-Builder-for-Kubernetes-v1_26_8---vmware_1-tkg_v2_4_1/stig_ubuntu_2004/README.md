Note: This is a generic STIG hardening recipe, it is NOT CUSTOMIZED for the US Govt or any other specific
regional use case.  In order to customize it, see the variables below !!!

# ubuntu_20.04_STIG

Cookbook to automate STIG implementation for Ubuntu 20.04

## Cookbook variables:

Comnpensating controls may exist that satisfy the STIG requirement for a security measure.  
Variables definded in [vars.yml](vars/main.yml) allow skipping package insatallation where compensating controls exist.

|Name|Default|Description|
|----|:-------:|-----------|
|`install_fips`| `yes`|Install FIPs certified kernel, openssh, openssl and strongswan modules. Requires `UBUNTU_ADVANTAGE_PASSWORD` and `UBUNTU_ADVANTAGE_PASSWORD_UPDATES` variables to be set. There are **no compensating control** from FIPS STIG requirement. **Ubuntu 20.04 is not FIPS compliant/certified.**|
|`install_aide`| `yes`|`aide` is an open source host based file and directory integrity checker. `install_aide` can be set to `no` if any other integrity checker (e.g. Tripwire) is installed on the VM instead of being baked into the image.|
|`install_chrony`| `no`| `chrony` provides fast and accurate time synchronization. `install_chrony` can be set to `no` if any other time synching package (e.g. `timesyncd`) is used.|
|`install_audispd_plugins`|`yes`| `audispd_plugins` relays audit events to remote machines.  `audispd_plugins` can be set to `no` if any other mechanism of relaying logs to remote server (e.g fluentd) is being used.|
|`remove_existing_ca_certs`|`no`| STIG hardening requires `/etc/ssl/certs` only contain certificate files whose sha256 fingerprint match the fingerprint of DoD PKI-established certificate. If the value is set to `yes`, all other certficates under `/etc/ssl/certs` except DoD CA certs will be deleted.|
`install_sshd_login_banner`| `yes`| Replaces default ubuntu ssh login banner with [login banner specified by STIG](vars/main.yml)|
|`set_sshd_config_ciphers`|`yes`|Configure the SSH daemon to only implement FIPS-approved algorithms. |
|`set_sshd_config_macs`|`yes`|Configure the SSH daemon to only use Message Authentication Codes (MACs) that employ FIPS 140-2 approved ciphers. |
|`hold_packages`|`no`| k8s image-builder sets `apt-mark hold` on all installed packages preventing unintentional node images upgrades. This also prevents uninstalling unused packages by this playbook. Setting the value to `yes` will allow uninstalling unused packages and will also sets `apt-mark hold` on packages installed by this playbook |
|`UBUNTU_ADVANTAGE_PASSWORD`| |Env variable in `<USERNAME>:<PASSWORD>` format required to access Ubunutu `FIPS (ppa:ubuntu-advantage/fips)` private Personal Package Archive(ppa). Required if `install_fips` is set to `yes`.|
|`UBUNTU_ADVANTAGE_PASSWORD_UPDATES`| |Env variable in `<USERNAME>:<PASSWORD>` format required to access Ubunutu `FIPS Updates (ppa:ubuntu-advantage/fips-updates)` private Personal Package Archive(ppa). Required if `install_fips` is set to `yes`.|
|`UBUNTU_FIPS_SUBSCRIPTION_ID`| |Ubuntu Advantage(ua) Subscription ID. This environemnt variable can be used instead of `UBUNTU_ADVANTAGE_PASSWORD` and `UBUNTU_ADVANTAGE_PASSWORD_UPDATES`|

## Exceptions:

|	ID	|	Title	|	Exception	|
| -----| ----| -----|
|	V-238197	|The Ubuntu operating system must enable the graphical user logon banner to display the Standard Mandatory DoD Notice and Consent Banner before granting local access to the system via a graphical user logon. 	|	GUI not installed	|
|	V-238198	|The Ubuntu operating system must display the Standard Mandatory DoDNotice and Consent Banner before granting local access to the system via agraphical user logon. 	|	GUI not installed	|
|	V-238199	|The Ubuntu operating system must retain a user's session lock untilthat user reestablishes access using established identification andauthentication procedures. 	|	GUI not installed	|
|	V-238201	|The Ubuntu operating system must map the authenticated identity to theuser or group account for PKI-based authentication. 	|	Cluster VM do not use `pam-pkcs11`	|
|	V-238208	|The Ubuntu operating system must require users to reauthenticate forprivilege escalation or when changing roles. 	|	`<user>` requires sudo to run cloud-init. `ubuntu` in this case	of AWS AMI|
|	V-238229	|The Ubuntu operating system, for PKI-based authentication, mustvalidate certificates by constructing a certification path (which includesstatus information) to an accepted trust anchor. 	|	Cluster VM do not use `pam-pkcs11`	|
|	V-238232	|The Ubuntu operating system must electronically verify PersonalIdentity Verification (PIV) credentials. 	|	Cluster VM do not use `pam-pkcs11`	|
|	V-238233	|The Ubuntu operating system for PKI-based authentication, mustimplement a local cache of revocation data in case of the inability to accessrevocation information via the network. 	|	Cluster VM do not use `pam-pkcs11`	|
|	V-238306	|The Ubuntu operating system audit event multiplexor must be configuredto off-load audit logs onto a different system or storage media from the systembeing audited. 	|	Site-specific control	|
|	V-238307	|The Ubuntu operating system must immediately notify the SA and ISSO(at a minimum) when allocated audit record storage volume reaches 75% of therepository maximum audit record storage capacity. 	|	 `space_left`  setting cannot be determined at image build time	|
|	V-238321	|The Ubuntu operating system must have a crontab script running weeklyto offload audit events of standalone systems. 	|	Site-specific control	|
|	V-238328	|The Ubuntu operating system must be configured to prohibit or restrictthe use of functions, ports, protocols, and/or services, as defined in the PPSMCAL and vulnerability assessments. 	|	Network protection is provided by VPC for perimeter protection and iptables for container networks	|
|	V-238331	|The Ubuntu operating system must automatically remove or disableemergency accounts after 72 hours. 	|	Manual verification	|
|	V-238332	|The Ubuntu operating system must set a sticky bit on all publicdirectories to prevent unauthorized and unintended information transferred viashared system resources. 	|	Directories failing the test are contianer directories.	|
|	V-238335	|Ubuntu operating systems handling data requiring  data at rest protections must employ cryptographic mechanisms to prevent unauthorizeddisclosure and modification of the information at rest. 	|	Encryption of data at rest is handled by the IaaS	|
|	V-238361	|The Ubuntu operating system must allow the use of a temporary passwordfor system logons with an immediate change to a permanent password. 	|	Manual verification	|
|	V-238362	|The Ubuntu operating system must be configured such that PluggableAuthentication Module (PAM) prohibits the use of cached authentications afterone day. 	|	Cluster VM do not use `pam-pkcs11`	|
|	V-238364	|The Ubuntu operating system must only allow the use of DoDPKI-established certificate authorities for verification of the establishmentof protected sessions. 	|	Customer can use BYOI to remove existing CAs and add DoD CA to the image	|
|	V-238365	|Ubuntu operating system must implement cryptographic mechanisms toprevent unauthorized modification of all information at rest. 	|	Encryption of data at rest is handled by the IaaS	|
|	V-238366	|Ubuntu operating system must implement cryptographic mechanisms toprevent unauthorized disclosure of all information at rest. 	|	Encryption of data at rest is handled by the IaaS	|
|	V-238367	|The Ubuntu operating system must configure the uncomplicated firewallto rate-limit impacted network interfaces. 	|	Status listings checks must be preformed manually	|
|	V-238379	|The Ubuntu operating system must disable the x86 Ctrl-Alt-Delete keysequence if a graphical user interface is installed. 	|	GUI not installed	|
|	V-252704	|The Ubuntu operating system must disable all wireless network adapters.	|	Wireless networking is not used	|

## Cloud provider specific tasks:
This repo has been tested on AWS only. 
For setting [cloud provider specific modules](tasks/V-219151.yml#L35-43) refer to https://security-certs.docs.ubuntu.com/en/fips-cloud-containers
