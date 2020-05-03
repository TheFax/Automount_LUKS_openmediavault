# Automount LUKS disk using remote key #

## Goals ##

In my house there is a NAS based on OpenMediaVault that I want to protect from theft. To make it possible the data disks inside the NAS are crypted with LUKS, using the plugin `openmediavault-luksencryption`. This seems to be a status-of-the-art solution, but there is a single problem: every time the server starts, you need to enter into the WegGUI in order to unlock the encrypted drive.
In my case, the server automatically shutdown at 23:59 every day, and is turned on manually only when I need it. The access via WebGUI is very uncomfortable because you need a computer: unlock the drive via smartphone or via tablet is very hard and tricky (the web interface is not comfortable to use on a smartphone), and you always need to type by hand the administrator password and the LUKS password.
I searched over the internet a solution, but nobody seems to have my problem… Strange! Probably 99% of people keeps its NAS powered-on 24/7. It's not a solution for me: I use it only few hours per week, and keeping it powered is a waste of energy.

The solution I adopted is this: store the LUKS password in a file into the local router (based on OpenWRT or LEDE) and use a little script for automatic unlocking the drives at every startup.
If the NAS will be stolen, it will not be able to retrieve the key stored locally and the data will be protected. Obviously this solution have its weaknesses: the bigger one is when the NAS && the router are stolen together. In this scenary the thief has got the crypted data and the key. But this is not a problem for me, because the server contains only photos and family data, and I want to protect them from normal thief or low-level users, not from brute-force attacks or Russians spies.
With a little modify to the script, you will be able to store the key/password in a remote server or over the internet (with pros… and cons…).

## OpenWRT or LEDE ##

Let's start creating the password file into the router.
Gain access via SSH and create a new file into the folder /www/:
( :!: warning :!: this password will be accessible to anyone who knows the name of the file and have access to a local connection to the router. You can use another name for the file, example: /www/j-if9_8n4ikZWvblkhp.txt)
```
root@OpenWrt:~# echo "MySecretPassword" > /www/key.txt
```

Easy, right?

## Openmediavault script ##

Now create the script into the OpenMediaVault server.
1. Login via SSH, and go to the folder `/home/root`.
2. Download from this repository the file `automount_cryptodisk`
3. Make it executable:
```
root@openmediavault:/# chmod 755 automount_cryptodisk
```
4. Edit the script, and set the location of your remote key changing the line `REMOTEKEY=“192.168.0.1/key.txt”`:
```
root@openmediavault:/# nano automount_cryptodisk
```
5. Now go to the administration page of your NAS and create a new job:

Reboot your server and enjoy your auto-unlock :-)

## Principle of operation ##

As I wrote in the point number 5., the script will be executed at every startup of OpenMediaVault.
The sequence of operations it performs are:
* download the password file from the router into a temporary file (placed in a tmpfs==RAM)
* load the password into a variable
* delete the temporary file (please note that it is only for refinement: the file was placed in RAM and it would still be lost on next power-off of the server)
* search for all LUKS-encrypted devices installed into the system (saving their dev/names into a bash array)
* for each device in the array:
  * extract the device name (eg: sdb)
  * try to unlock the device, with the password loaded on the step 2.

## Troubleshooting ##

Every time the server will be turned on, a LOG file will be created in the `/tmp` directory. You can see it:
```
cat /tmp/automount_crypto.
```

Please analize the content of the file in event of trouble.

## TO-DO ##

* Wipe the temp file downloaded from the router before removing it

## Compatibility ##

Tested on OMV3 / Openmediavault 3
Tested on OMV4 / Openmediavault 4
Tested on OMV5 / Openmediavault 5

## External links ##

http://forum.openmediavault.org/index.php/Thread/15532-LUKS-auto-unlock-via-keyfile-from-a-network-device/#post140091
