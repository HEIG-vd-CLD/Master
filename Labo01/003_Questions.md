* What is the smallest and the biggest instance type (in terms of
  virtual CPUs and memory) that you can choose from when creating an
  instance?

```
The smallest one is the t2.nano with 0.5 memory and one CPU.
The biggest one is the u-6tb1.112xlarge with 6144 memory and 448 CPU.
```

* How long did it take for the new instance to get into the _running_
  state?

```
It takes between two or tree minutes but the exact time can vary depending on the type and configurations.
```

* Using the commands to explore the machine listed earlier, respond to
  the following questions and explain how you came to the answer:

    * What's the difference between time here in Switzerland and the time set on
      the machine?
```bash
devopsteam12@ip-10-0-0-5:~$ date
Wed Mar 13 15:26:49 UTC 2024
devopsteam12@ip-10-0-0-5:~$ timedatectl
               Local time: Wed 2024-03-13 15:26:51 UTC
           Universal time: Wed 2024-03-13 15:26:51 UTC
                 RTC time: Wed 2024-03-13 15:26:51
                Time zone: Etc/UTC (UTC, +0000)
System clock synchronized: yes
              NTP service: active
          RTC in local TZ: no
devopsteam12@ip-10-0-0-5:~$
```

```bash
PS C:\Users\diazs> date

mercredi, 13 mars 2024 16:28:19

PS C:\Users\diazs> Get-TimeZone

Id                         : W. Europe Standard Time
DisplayName                : (UTC+01:00) Amsterdam, Berlin, Berne, Rome, Stockholm, Vienne
StandardName               : Europe de l’Ouest
DaylightName               : Europe de l’Ouest (heure d’été)
BaseUtcOffset              : 01:00:00
SupportsDaylightSavingTime : True
```

```
The difference is one hours but could be two if the time zone of "W. Europe Standard Time" changes to UTC+02:00. 
```

    * What's the name of the hypervisor?
```
sudo dmidecode -t system

```

    * How much free space does the disk have?
```bash
devopsteam12@ip-10-0-0-5:~$ df -h
$Filesystem      Size  Used Avail Use% Mounted on
udev            476M     0  476M   0% /dev
tmpfs            98M  484K   97M   1% /run
/dev/xvda1      7.7G  1.4G  5.9G  19% /
tmpfs           488M     0  488M   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
/dev/xvda15     124M   12M  113M  10% /boot/efi
tmpfs            98M     0   98M   0% /run/user/1019
tmpfs            98M     0   98M   0% /run/user/1012
```

```
It has 5.9 Go (like 81% free space).
```

* Try to ping the instance ssh srv from your local machine. What do you see?
  Explain. Change the configuration to make it work. Ping the
  instance, record 5 round-trip times.

```bash
PS C:\Users\diazs> ping 10.0.12.10

Envoi d’une requête 'Ping'  10.0.12.10 avec 32 octets de données :
Délai d’attente de la demande dépassé.
Délai d’attente de la demande dépassé.
Délai d’attente de la demande dépassé.
Délai d’attente de la demande dépassé.

Statistiques Ping pour 10.0.12.10:
    Paquets : envoyés = 4, reçus = 0, perdus = 4 (perte 100%),
PS C:\Users\diazs> ping 15.188.43.46

Envoi d’une requête 'Ping'  15.188.43.46 avec 32 octets de données :
Délai d’attente de la demande dépassé.
Délai d’attente de la demande dépassé.
Délai d’attente de la demande dépassé.
Délai d’attente de la demande dépassé.

Statistiques Ping pour 15.188.43.46:
    Paquets : envoyés = 4, reçus = 0, perdus = 4 (perte 100%),


for 5 round-trip-times = ping -c 5 <IP_address_or_DNS_name>
```

* Determine the IP address seen by the operating system in the EC2
  instance by running the `ifconfig` command. What type of address
  is it? Compare it to the address displayed by the ping command
  earlier. How do you explain that you can successfully communicate
  with the machine?

```bash
devopsteam12@ip-10-0-0-5:~$ ip addres show
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host noprefixroute
       valid_lft forever preferred_lft forever
2: enX0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc fq_codel state UP group default qlen 1000
    link/ether 06:d9:0b:3d:1a:85 brd ff:ff:ff:ff:ff:ff
    inet 10.0.0.5/28 metric 100 brd 10.0.0.15 scope global dynamic enX0
       valid_lft 1809sec preferred_lft 1809sec
    inet6 fe80::4d9:bff:fe3d:1a85/64 scope link
       valid_lft forever preferred_lft forever
```

```
Its address is 10.0.0.5/28 in IPv4 and it is private.
It can communicate because of the ssh tunnel to your Drupal instance.
```
