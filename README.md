# **🖧 Ubuntu Server Static IP Configuration (VM & Bare Metal)**

## **📌 Overview**
When configuring **Ubuntu Server**, you may encounter an issue where the **Static IP Address** reverts to **DHCP** after a reboot. This problem is common in both **Virtual Machines (VMs)** (e.g., **Proxmox, VirtualBox, VMware**) and **bare-metal servers**.

This guide will help you troubleshoot and resolve this issue by properly configuring your network settings and disabling any conflicting services like **cloud-init**.

---

## **🔍 Understanding the Issue**
### **⚠️ Cause**
By default, **Ubuntu Server** uses **cloud-init** to manage network settings dynamically. While this is helpful in cloud environments, it can override manual network configurations, causing:
- ❌ **Static IP changes to DHCP after reboot**
- ❌ **Netplan configuration files being ignored or overwritten**
- ❌ **Inconsistent networking behavior**

### **🔄 Effect**
If not resolved, this can result in:
- 🔗 Loss of network connectivity after reboot
- 🌍 Difficulty accessing the server remotely
- 🛑 Conflicts in network infrastructure when multiple devices receive unintended DHCP addresses

### **✅ Solution**
To permanently set a **Static IP**, we will:
1. **Disable cloud-init** (to prevent automatic configuration changes)
2. **Manually configure the network** using **Netplan**
3. **Apply and verify the changes**

---

## **1️⃣ Disable Cloud-Init**
Cloud-init manages the default network configuration, but it often interferes with manual settings. To disable it:

```bash
sudo touch /etc/cloud/cloud-init.disabled
```

Then, remove any existing network configuration files created by cloud-init:

```bash
sudo rm -rf /etc/netplan/50-cloud-init.yaml
```

Disable the cloud-init service:

```bash
sudo systemctl disable --now cloud-init
```

(Optional) If you want to completely remove cloud-init:

```bash
sudo apt purge cloud-init -y
```

---

## **2️⃣ Manually Configure a Static IP**
Now that **cloud-init** is disabled, we can manually set up **Netplan**.

### **🖥️ Step 1: Identify the Network Interface**
Run the following command to find your active network interface:

```bash
ip link show
```

You will see output like this:

```
2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP mode DEFAULT group default qlen 1000
```

Take note of the interface name (**e.g., enp0s3**), as we will use it in the next step.

### **📝 Step 2: Create a Static IP Configuration**
Create a new **Netplan** configuration file:

```bash
sudo nano /etc/netplan/50-static-ip.yaml
```

Add the following configuration (adjusting based on your network setup):

```yaml
network:
  version: 2
  renderer: networkd
  ethernets:
    enp0s3:  # Change this to your actual network interface
      addresses:
        - 192.168.1.100/24
      routes:
        - to: default
          via: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
      dhcp4: no
```

Save the file (**Ctrl + X → Y → Enter**).

Ensure the file has the correct permissions:

```bash
sudo chmod 0600 /etc/netplan/50-static-ip.yaml
```

---

## **3️⃣ Apply and Verify Configuration**
After setting up the static IP, apply the changes:

```bash
sudo netplan generate
sudo netplan try
```

If everything looks good, make it permanent:

```bash
sudo netplan apply
```

Reboot the server:

```bash
sudo reboot
```

After rebooting, confirm the **Static IP** is applied:

```bash
ip a
```

---

## **🛠️ Troubleshooting & Additional Tips**
### **🔎 a. Verify the Correct Network Interface**
If your changes do not take effect, ensure the correct interface is being used:

```bash
ip link show
```

Update your **Netplan** configuration with the correct name.

### **🛑 b. Check Netplan for Errors**
Run this command to validate your Netplan configuration:

```bash
sudo netplan try
```

If an error appears, double-check your YAML syntax.

### **📡 c. Confirm DHCP is Disabled**
Check if your interface is still requesting DHCP:

```bash
journalctl -u systemd-networkd --no-pager | grep -i 'dhcp'
```

If DHCP is still active, ensure that **dhcp4** is set to **no** in your Netplan file.

---

## **📌 Conclusion**
By following these steps, you can ensure that your **Ubuntu Server** retains a **Static IP Address** across reboots, whether running on a **Virtual Machine (VM)** or **bare-metal hardware**.

### **✅ Summary of Key Steps:**
✔️ Disable **cloud-init** to prevent conflicts  
✔️ Manually configure **Netplan** with a static IP  
✔️ Apply changes and verify the setup  
✔️ Troubleshoot if necessary  

Now your **Ubuntu Server** should maintain its network configuration consistently!

---

## **📚 References**
- [Ubuntu Netplan Documentation](https://netplan.io/)
- [Cloud-Init Documentation](https://cloudinit.readthedocs.io/en/latest/)
- [Ubuntu Server Networking Guide](https://ubuntu.com/server/docs/network-configuration)
