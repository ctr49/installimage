# Full disk encryption (FDE) install latest Debian on a Hetzner Cloud Server


1. enable rescue mode (on a fresh or to be rebuild cloud server)
  * make sure to specify linux64 as the rescue image and actually supply a ssh public key (this will also be the key that can be used to remote unlock)
2. (re)start the cloud server, causing it to boot into rescue mode
3. ssh to the cloud server and run:
```
git clone https://github.com/ctr49/installimage.git
/root/installimage/installimage -p /boot:ext4:512M,lvm:vg0:all -v vg0:swap:swap:swap:4G,vg0:root:/:ext4:5G,vg0:var:/var/:ext4:3G,vg0:tmp:/tmp/:ext4:500M,vg0:home:/home/:ext4:500M -i /root/images/Debian-103-buster-64-minimal.tar.gz -K /root/.ssh/robot_user_keys -x luks -a
```
4. copy the (randomly generated) LUKS passphrase from the output out of section 4, output is like
```
4/16  :  Creating LUKS container                         busy The LUKS passphrase is THIS_IS_YOUR_LUKS_PASSPHRASE
```
5. Reboot
6. Unlock using the above supplied LUKS passphrase either via kvm or remote unlock via ssh like:
```
$ ssh -p 65222 root@CLOUD_SERVER_IP
Please unlock disk cryptroot:
```
 * note: your input is not visible, so just enter the LUKS passphrase blindly and hit enter
7. Enjoy your fully encrypted system.

## CAVEATS:
* partition layout (single disk sda, no raid) are  currently hard-coded.
* logical volume names and sizes are passed as installimage parameter (-v), adjust as needed. above example results in the following setup

|logical volume name or partition| mount point|size|
| --- | --- | ---- |
|/dev/sda1|/boot|0.5GB|
|/dev/sda2|LUKS container|remaining space|
|swap||4GB|
|root|/|5GB|
|var|/var|3GB|
|tmp|/tmp|0.5GB|
|home|/home|0.5GB|

* remaining space in the volume group is available for a data volume or to extend existing volumes
* compared to the default setup, only LVM and LUKS with above mentioned layout are added otherwise the installed system is the same as if you installed directly through Hetzner
 * the Hetzner install images user from Cloud Console and rescue-system/installimage are not identical. The post-install script herein attempts to bring the image into a state most similar to the image installed via Cloud Console.
