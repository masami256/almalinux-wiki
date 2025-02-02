---
title: "ELevate Quickstart Guide"
---

# ELevate Quickstart Guide

::: warning
Before beginning, we **HIGHLY** recommend that you follow system administration best practices and make sure you have backups and/or snapshots of your system before you proceed. It is recommended to do a trial run in a sandbox to verify that migration worked as expected before you attempt to migrate any production system. Please report any issues encountered to the [AlmaLinux Bug Tracker](https://bugs.almalinux.org) and/or [AlmaLinux Chat Migration Channel](https://chat.almalinux.org/almalinux/channels/migration)
:::

:::danger
The ELevate project only supports official operating systems repositories. It doesn’t support external repositories like EPEL. Please, check the [ELevate Frequent Issues](/elevate/ELevate-frequent-issues) page for known and frequent issues.
:::

This guide contains steps on how to upgrade your RHEL-based operating system to the next major version.

Currently, the following migration directions are available:

![image](/images/ELevate-scheme.svg)

\* - migration to CentOS Stream 9 is currently in process and will be available later. <br>
\** - migration to Oracle Linux 9 is available with the [Oracle Leapp utility](https://blogs.oracle.com/linux/post/upgrade-oracle-linux-8-to-oracle-linux-9-using-leapp) and will not be supported by ELevate project.

You need CentOS 7, AlmaLinux 8, EuroLinux 8 or Rocky Linux 8 system installed to use this guide.

* Fully updated system is required to accomplish the upgrade. So, install the latest updates first, and reboot.
```
sudo yum update -y
sudo reboot
```


* Install `elevate-release` package with the project repo and GPG key.
```
sudo yum install -y http://repo.almalinux.org/elevate/elevate-release-latest-el$(rpm --eval %rhel).noarch.rpm
```


* Install leapp packages and migration data for the OS you want to upgrade. Possible options are:
    * leapp-data-almalinux
    * leapp-data-centos
    * leapp-data-eurolinux
    * leapp-data-oraclelinux
    * leapp-data-rocky
```
sudo yum install -y leapp-upgrade leapp-data-almalinux
```

* Start a preupgrade check. In the meanwhile, the Leapp utility creates a special */var/log/leapp/leapp-report.txt* file that contains possible problems and recommended solutions. No rpm packages will be installed at this phase.

:::warning
Preupgrade check will fail as the default install doesn't meet all requirements for migration.
:::

```
sudo leapp preupgrade
```

This summary report will help you get a picture of whether it is possible to continue the upgrade.

:::tip
In certain configurations, Leapp generates */var/log/leapp/answerfile* with true/false questions. Leapp utility requires answers to all these questions in order to proceed with the upgrade.
:::

* The following fixes from *the /var/log/leapp/leapp-report.txt* file are the most popular for CentOS 7, but it's recommended to review the whole file.
```
sudo rmmod pata_acpi
echo PermitRootLogin yes | sudo tee -a /etc/ssh/sshd_config
sudo leapp answer --section remove_pam_pkcs11_module_check.confirm=True
```

* Here are the most popular fixes for RHEL8-based operating systems:
```
sudo sed -i "s/^AllowZoneDrifting=.*/AllowZoneDrifting=no/" /etc/firewalld/firewalld.conf
sudo leapp answer --section check_vdo.no_vdo_devices=True
```
Check the [ELevate Frequent Issues](/elevate/ELevate-frequent-issues) page for known and frequent issues and guidance steps to solve them.

* Start an upgrade. You'll be offered to reboot the system after this process is completed.
```
sudo leapp upgrade
sudo reboot
```

* A new entry in GRUB called `ELevate-Upgrade-Initramfs` will appear. The system will be automatically booted into it.
   See how the update process goes in the console.

* After reboot, login to the system and check how the migration went. Verify that the current OS is the one you need. Check logs and packages left from previous OS version, consider removing them or update manually.
```
cat /etc/redhat-release
cat /etc/os-release
rpm -qa | grep el7 # for 7 to 8 migration
rpm -qa | grep el8 # for 8 to 9 migration
cat /var/log/leapp/leapp-report.txt
cat /var/log/leapp/leapp-upgrade.log
```

### Demo Video

Check Demo of a CentOS 7.x to AlmaLinux 8.x migration using the software and data provided by the AlmaLinux ELevate Project. 

<iframe width="856" height="482" src="https://www.youtube.com/embed/Vzl9QxG5mvo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

