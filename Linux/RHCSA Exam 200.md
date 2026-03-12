---
title: Exam 200 실전 문제
description: kyoulee blog
preview: /post/templates/images/default-og-image.jpg
created: 2026-02-07T03:02:54+09:00
updated: 2026-02-07T03:02:54+09:00
tags:
  - Exam200
  - RHCSA
  - Redhat
status: public
id: 0
writer: kyoulee
---

RedHat Systems Administrator 9/10 (EX200) Practice Exam.
youtube.com/@OzzoyBits
Max Score :300 Points
Time to Execute: 2.5 Hours
Passing Score : 70% or 210/300 Points
Please Take Care:
FirewallD and SeLinux must be On/Enabled at all times (and persist reboots).
There is no specific way to execute the tasks, use the method you prefer. (CLI, TUI, GUI, etc)
Can do the tasks in any order.
All  the Names, IP addresses, hostnames etc in this practice exam may/will not match the real exam.
Two Servers (A and B) tasks must be executed on specific
servers:
Exam Tasks, in the order they present it(and i can remember them :) )Some "stuff" was adapted to make it easy on us (like server names) others are just like the exam. The prep exam has more questions than the real exam. But all can pop.
Server A and B

1. Configure the Network. ServerA now ServerB comes after password reset.
    Assign this ip address config to your virtual machines as follows:
    Hostname servera.lab.example.com and serverb.lab.example.com

    IP Address: 192.168.1.150(1)
    NetMask:255.255.255.0
    GateWay: 192.168.1.1
    Name Server- 192.168.1.100

    If you want to be fancy:
    (use nmtui :))

    Example Solution:
```sh
nmcli connection show
Ip a
Nmcli connection modify (use profile name here) ipv4.address
192.168.1.150/24 ipv4.gateway 192.168.1.1 mm.dns 8.8.8.8
ipv4.method static
nmcli connection up (profile)
ping something
```

2. Configure the repositories available at the repo server at:
    http://repo.lab.example.com/rocky9.5/repo/BaseOS
    http://repo.lab.example.com/rocky9.5/repo/AppStream/

```sh
dnf repolist all (optional)
cd /etc/yum.repos.d
vim exam.repo (any thing repo)

# file to exam.repo
## [BaseOS]
## Name = BaseOS
## Baseur = http://repo.lab.example.com/rocky9.5/repo/BaseOS
## enabled=1
## gpgcheck=0

## [AppStream]
## Name = AppStream
## Baseurl = http://repo.lab.example.com/rocky9.5/repo/AppStream/
## enabled=1
## gpgcheck=0
```

3. The content via http has been configured in port 82 at the /var/www/html dir (they ask not to change, add or remove content), just make it accessible.
    
    This will add simulated content:
  
```sh
yum install httpd-y
systemctl enable -now httpd
echo "find this" > /var/www/html/index.html


# (something like this)
# To resolve:
semanage porl -I grep httpd #(check to see if port 82 is enabled for http) (if not to fix:)

dnf install -y policycoreutils* (to install Selinux management tools)

# To add port 82 for httpd

semanage port -a -t http_port_t-p tcp 82
semanage porl - grep http

# Semanage man page has it all just copy paste

# Firewall:
firewall-cmd -permanent -add-port=82/tcp
firewalll-cmd -reload

# Maybe this helps. (or test using default website httpd.conf Listen directve)

vim /etc/httpd/conf/httpd.conf or /etc/httpd/conf.d/FILENAME

# Add to httpd.conf
## <virtualhost 192.168.1.150:82>
## Servername servera.lab.example.com
## Documentroot /var/www/html </virtualhost>

```


4. Create the following users, groups and add users to the groups as instructed:(actual names from exam :))

    a) A group named admin
    b) User named harry that belongs to group admin
    c) User named natasha who belongs to admin
    d) User sarah who does not have access to an interactive shell and is not member of admin.
    e) All users must have the password (password). ( on the exam they ask a different password, but let's save brain cells here  :) )
    
```sh
groupadd admin
useradd -G admin harry
useradd -G admin natasha
```

5. Create a directory for collaboration named /common/admin and configure as it follows:
    a) The group owner of the directory is the the group admin
    b) The directory should be readable, writable, and accessible to the members of admin, and no other. Root is root "
    c) New files created must belong to the admin group.

```sh
mkdir -p /common/admin 
chgrp admin /common/admin 
chmod 770 /common/admin 
chmod g+s /common/admin

# This should be it, create a new file and see if it belongs go the admin group (ie touch /common/admin/testfile)
```

6. Configure Autofs. (this one is a wildcard, meaning, not everyone gets this one.). (i do not remember this one very well but is something like this 

    a) Auto mount the following NFS Shares on servera.lab.example.com at the /automount directory: nfs.lab.example.com:/public and nfs.lab.example.com:/private
    b) Public share has read only to all
    c) Private read write to all
    d) Shares get unmounted if not in use for 30 seconds ( i forgot this and got all marks, just dot it and do not be a dumb like me)
    
```sh
mkdir -p /automount/private
mkdir -p/automount/public
dnf install nfs* -y
vim /etc/auto.master (or drop a file in /etc/auto.master.d)

# Add to auto.master file
## /automount /etc/auto.automout-timeout=30 (the timeout thing i forgot)

vim /etc/auto.automount

# Add to auto.automount
## public -ro,sync nfs.lab.example.com:/public 
## private -r,sync nfs.lab.example.com:/public

cd /automount # cd private #cd... #cd public
```
 
7. Set a Cron job, using the harry user. On 12:30 print "EX200" with echo. (or something along these lines)

```sh
su - harry 
crontab -e
30 12 * * * /bin/echo "EX200"

# Second part.
# Deny natasha the use of crontab
vim /etc/cron.deny 

# Add to cron.deny
## natasha
```

8. AULS, long and boring but doable. I he instructions were long, bit it's something like this:
    Copy the /etc/stab to /var/tmp and set the perms like this:
    a) The file is owned by root should be by default)
    b) The file belongs to the group root (should be by default)
    c) The file is not executable by no one
    d) harry can read and write the file
    e) Natasha cannot read nor write
    f) Everyone else , users that exist and the ones that will be created can read the 
    
```sh
cp /etc/fstab /var/tmp
setfacl -m u:harry:w /var/tmp/fstab
setfacl -m u:natasha:---/var/tmp/fstab

# Check to see
getfacl /var/tmp/fstab
```

9. NTP
    a) Configure your system to be client of ntp.lab.example.com
```sh 
dnf install chrony* -y #vim /etc/chrony.conf

# Comment all and add
## Server ntp.lab.example.com iburst
systemctl restart chronyc service
chronyc sources
```

10. Local and copy files
    
    Find all files greater than 4MB in the /etc directory and copy them to /find/largefiles
```sh
mkdir-p/find/largefiles
find /etc -type f -size +4M -exec cp {) /find/largefiles/ \;
Is /find/largefiles
```

11. Create a new user with UID 6969 (gididy) and the password password and the user  is called billy

```sh 
useradd -u 6969 billy
passwd billy
cat /etc/passwd
```

13. create an archive: Backup and compress the /var/tmp folder to /root/ex200.tar.gz )

```sh
tar -zcvf /root/ex200.tar.gz /var/tmp (check man page for gz2, or gzip etc)
```

14. Config Permissions

    a) New files created by natasha as -r—--- as default permission.
    b) New new dir by natasha has as dr-x—--- as default permission
    
    Basically umask stuff (check perms video)

```sh 
su - natasha
vim.bash_profile

# Add bash_profile
## Umask 0277
```

14. The password for new users in servera.lab.example.com must expire after 20 days

```sh
vim /etc/login.defs

# add login.defs
## PASS_MAX_DAYS 20

# Test

useradd test
chage -l test
```

15. Allow sudo

    The admin group can use sudo without password
    
```sh
vim /etc/sudoers

# Add to sudoers
## %admin ALL=ALL NOPASSWD: ALL

```

16. Create a bash script for:(something like this)

    a) Create a script called gofind to find files in the /usr/share dir that are smaller than 1M.
    b) The script copies the found files to the / root/find directory
```sh
mkdir -p/root/find
vim gofind.sh
find /usr/share -type f-size - 1M -exec cp {y/root/find I;

chmod +x gofind.sh
./gofind.sh
```


17. The admin of serverb.lab.example.com forgot the root password.

    Reset the password to be "redhat"
```sh
#Fix Grub timeout
vim /etc/default/grub
GRUB_TIMEOUT_=30
grub2-mkconfig-o/boot/grub2/grub.cfg
```

reset muchine
    
- start boot option
- press `e` key

```os
load_video
set gfxpayload=keep
insmod gzio
linux ($root)/vmlinuz-5.14.0-503.14.1.el9_5.x86_64 root=/dev/mapper/rl_unkn\
own000c29d5082d-root ro crashkernel=1G-4G:192M,4G-64G:256M,64G-:512M resume\
=/deu/mapper/rl_unknown000c29d5082d-swap rd.lvm.lv=rl_unknown000c29d5082d/r\
oot rd. lvm.lu=rl_unknown000c29d5082d/swap rhgb quiet
initrd ($root)/initramfs-5.14.0-503.14.1.el9_5.x86_64.img $tuned_initrd
```

- add rd.break
    
```os
oot rd. lvm.lu=rl_unknown000c29d5082d/swap rhgb quiet rd.break
```
- 
- control + x to reboot

```sh
mount -o remount.rw /sysroot/
chroot /sysroot/

    id 
    passwd
    touch /.autorelabel
    exit

exit
```

- check login root
- login root password press


18. Create a 512MB swap partition

On SeverB

```sh
lsblk # (shoud see a sdb disc in my scenario we will use ano    ther secondary disc, in exam you should have sda, sdb etc #fdisk /dev/sdb
fdisk /dev/sdb

# partition
p # partition info
# new partition
n
p # select partition primary
1 # partition number
2048  # first sector
+512M # last sector make size
t # type change 
82 # hex code or alias
p # parition info
w # write and save 

lsblk # check 
makswap /dev/sda1 # make swap memory 
blkid # check UUID

vim /etc/fstab

# Add 
## UUID=“xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx” swap swap defaults 0 0 

swapon -a
swapon -s # check swap 

```

19. Create on logical volume named database, it should be on the datastore volume group, 50 extent size and formatted with ext3. Datastore must have a extent size must be 8MB. Mount persistently in /mnt/database

Let use the sdb disk for simplicity

```sh
fdisk /dev/sdb
# Create a 2GB partition (we on)

# partition
p # partition info
# new partition
n
p # select partition primary
1 # partition number
2048  # first sector
+2G # last sector make size
t # type change 
8e # hex code or alias
p # parition info
w # write and save

lsblk # check sdb stroge

pvcreate /dev/sdb1 # create 
pvs # show pv volume
vgcreate -s 8m datastore /dev/sdb1 # group datastore 8MB
pvs # show pv volume
vgs # show vg 
vgscan # show volume of lvm
vgdisplay # show all lvm and check pe size

lvcreate -l 50 -n database datastore # logical volume create
lvs # check logical volume

lsblk # check volume

mkfs.ext3 /dev/datastore/database # create filesystem
blkid # check type of volume size

vim /etc/fstab
# Add for fstab
## /dev/datastore/database /mnt/database ext3 defaults 0 0 

lvs # check logical volume show
mount -a # check
df # check

```

 20. Create the volume data with VDO, logical size 50 GB, mount under /data

```sh
dnf install vdo lvm2 kmod-kvdo -y

vgs
lsblk      

pvcreate /dev/sdc # create volume
vgs # check volume group show

man vgs # check man page

lvcreate —type vdo -n vdo-logicalvolume -L 50G -V50G vdo-volumegroup/vdo-pool

lvs # check vdo-pool size

blkid


mkfs.ext3 /dev/vdo-volumegroup/vdo-logicalvolume # create filesystem

lsblk
lvscan

blkid # check type of volume size

vim /etc/fstab
# Add for fstab
## UUID=“xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx” /data xfs defaults 0 0 

mount -a # moute all
df -h # check volume
```

21. Resize the previous created logical volume database to +100 extents

```sh
lvs

lvscan 

lvextend -l +100 -r /dev/datastore/database

lvscan 
lvdisplay
```

22. Enable the recommended tuned profile on your system.

```sh
dnf install tuned -y

systemctl —now enable tuned
tuned-adm profile # check options

tuned-adm recommend
tuned-adm profile virtual-guest
systemctl restart tuned
tuned-adm active
```

23. 

```sh

dnf sreach podman
dnf install podman container-tools -y # install podman

podman login registry.lab.example.com # —-tls-cerift=false

vim /etc/containers/registries.conf
# Add registries.conf
## unqualified-search-registries = [‘registry.access.redhat.com”, “registry.redhat.io”, “docker.io” ]
mkdif -p /home/student/.config/containers
su - student # login student
mkdir -p /home/student/.config/containers
vim /home/student/.config/containers/registries.conf

# Add registries.conf
## unqualified-search-registries = [‘registry.lab.example.com’]
## [[registry]]
## location = “registry.lab.example.com”
## insecure = true
## blocked = false

podman search httpd | grep ubi # find httpd container

podman pull registry.access.redhat.com/ubi10/httpd-24 # download container ubi10 
podman images # check images 
podman run -d —name testweb registry.access.redhat.com/ubi10/httpd-24
podman ps # check container
pod man exec -it xxxxxxxxxxxxx /bin/bash # use bash shell on container

vi /etc/httpd/conf/httpd/conf # find Listen port

# Find on conf
## Listen 0.0.0.0:8080

exit

pod man ps # check 8080 port is open 
cd /home/student/webserver/ 
ls 
cat index.html # make sure that is created

podman run -d —name webserver -p 8080:8080 -v /home/student/webserver/:/var/www/html:Z registry.access.redhat.com/ubi10/httpd-24

podman ps

curl http://serverb.lab.example.com
```

