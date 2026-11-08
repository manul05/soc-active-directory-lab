[Back to README](../README.md)

# 02. Splunk Server (Ubuntu 22.04)

## Step 1: Set a static IP

The forwarders send logs to a fixed address, so the Splunk server needs a permanent IP. Ubuntu uses Netplan.

1. Edit the Netplan file (it may be named `50-cloud-init.yaml` or `00-installer-config.yaml`; check
   `ls /etc/netplan`):
   ```bash
   sudo nano /etc/netplan/50-cloud-init.yaml
   ```
2. Use this content (confirm your interface name with `ip a`; it is often `enp0s3`):
   ```yaml
   network:
     version: 2
     ethernets:
       enp0s3:
         dhcp4: no
         addresses:
           - 192.168.10.10/24
         nameservers:
           addresses: [8.8.8.8]
         routes:
           - to: default
             via: 192.168.10.1
   ```
   > [!WARNING]
   > YAML must be indented with **spaces, never tabs**, two per level. A tab breaks the file and can take the
   > network down.
3. Apply and verify:
   ```bash
   sudo netplan apply
   ip a                # expect 192.168.10.10
   ping -c 3 google.com
   ```

## Step 2: Get the Splunk installer onto the VM

Ubuntu Server has no browser, so download the Splunk `.deb` on your host and pass it in through a VirtualBox shared
folder. (Splunk Enterprise needs a free Splunk account to download.)

1. Install Guest Additions support:
   ```bash
   sudo apt-get update
   sudo apt-get install -y virtualbox-guest-additions-iso virtualbox-guest-utils
   sudo adduser $USER vboxsf
   ```
2. In VirtualBox: **Devices > Shared Folders**, add the folder that holds the `.deb`.
3. Mount it and install:
   ```bash
   mkdir -p ~/share
   sudo mount -t vboxsf -o uid=1000,gid=1000 <SharedFolderName> ~/share
   sudo dpkg -i ~/share/<splunk-package>.deb
   ```

## Step 3: Start Splunk and enable it at boot

```bash
sudo -u splunk bash
/opt/splunk/bin/splunk start --accept-license    # sets the admin username and password
exit
sudo /opt/splunk/bin/splunk enable boot-start -user splunk
```

Use a lab-only password and store it in a password manager.

## Step 4: Configure Splunk

1. From your host browse to `http://192.168.10.10:8000` and sign in.
2. **Open the receiving port:** **Settings > Forwarding and receiving > Configure receiving > New receiving port**,
   enter `9997`.
3. **Create the index:** **Settings > Indexes > New Index**, name it `endpoint`.

If the Ubuntu firewall is enabled, allow the ports:

```bash
sudo ufw allow 8000/tcp
sudo ufw allow 9997/tcp
```

Next: [03. Windows telemetry](03-windows-telemetry.md).
