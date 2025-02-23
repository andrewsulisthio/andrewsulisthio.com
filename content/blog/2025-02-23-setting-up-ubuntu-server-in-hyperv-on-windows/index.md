---
title:  Setting Up Ubuntu Server in Hyper-V on Windows
description: "This guide outlines the steps I take to set up a new Ubuntu 24.04 server within a Hyper-V virtual machine on Windows. The primary focus is establishing secure SSH access, allowing convenient remote management from your preferred IDE. We'll cover VM creation, basic Ubuntu configuration, and secure SSH key-based authentication."
date: 2025-02-23T02:42:26.838Z
draft: false
tags:
    - Coding
type: default
---

This guide outlines the steps I take to set up a new Ubuntu 24.04 server within a Hyper-V virtual machine on Windows. The primary focus is establishing secure SSH access, allowing convenient remote management from your preferred IDE. We'll cover VM creation, basic Ubuntu configuration, and secure SSH key-based authentication.

**Creating the Hyper-V Virtual Machine:**

1.  Open Hyper-V Manager: Launch Hyper-V Manager from the Start Menu.
2.  Create a New Virtual Machine: In the "Actions" pane, click "New" and then "Virtual Machine." This will launch the New Virtual Machine Wizard.
3.  Follow the Wizard: Click "Next" to begin.
4.  Specify a Name and Location: Give your VM a descriptive name. Choose a location for the VM files.
5.  Choose Generation 1: Select "Generation 1." While Generation 2 offers some advantages, it can sometimes cause boot issues with Ubuntu in Hyper-V. Generation 1 offers a more stable experience.
6.  Assign Memory: Allocate RAM to the VM. A good starting point is 4 GB (4096 MB). **Important:** *Uncheck* "Use Dynamic Memory for this virtual machine" to ensure the VM consistently uses the allocated RAM, preventing resource fluctuations on your host machine.
7.  Configure Networking: Select a "Connection" using an external switch that is connected to the internet. This allows your VM to access the internet through your host machine's network connection.
8.  Create Virtual Hard Disk: A 64 GB virtual hard disk should be adequate for most general-purpose server tasks.
9.  Installation Options: Select "Install an operating system from a bootable CD/DVD-ROM" and choose "Image file (.iso)." Browse to the location of your downloaded Ubuntu 24.04 ISO file (e.g., `ubuntu-24.04.x-desktop-amd64.iso`).
10. Finish the Wizard: Review your settings and click "Finish" to create the VM.

**Basic Ubuntu Configuration:**

1.  Connect to the VM: In Hyper-V Manager, right-click your newly created VM and select "Connect."
2.  Start the VM: Click the "Start" button to power on the VM and begin the Ubuntu installation process.
3.  Follow the Ubuntu Installer: Use the default configuration options unless you have specific requirements. During the installation, if prompted, allow the installer to download updates.
4.  Install OpenSSH Server: During the installation process, select the option to install the "OpenSSH server." This is crucial for remote access.

**Securing Static IP Address:**

1.  Identify VM's IP Address: After the VM boots, use the command `ip addr` in the terminal to find the IP address assigned to the VM by your router's DHCP server.
2.  Reserve the IP Address in Your Router: Access your router's configuration page (usually through a web browser). Locate the DHCP settings and find the option to reserve an IP address for a specific MAC address. Enter the VM's MAC address and the IP address you identified in the previous step. This guarantees that the VM will always receive the same IP address when it connects to the network.

**Configuring SSH Key-Based Authentication:**

1.  Generate SSH Key Pair (using PuTTYgen on Windows):
    *   Download and run PuTTYgen.
    *   Set the "Number of bits in a generated key" to **4096** for enhanced security.
    *   Click "Generate" and move your mouse around to create randomness.
    *   Set a Key passphrase.
    *   The generated Key fingerprint should be kept as record in case of emergency (e.g. Server compromised)
2.  Copy the Public Key: Carefully copy the *entire* contents of the "Public key for pasting into OpenSSH authorized_keys file" field. **Important:** Make sure you copy *everything*, including the `ssh-rsa` at the beginning. Incorrectly copying this can lead to authentication failures.
3.  Connect to the VM via SSH (initially using password authentication) using Visual Studio Code:
    *   Use Visual Studio Code to connect to the new virtual machine.
    *   You will be prompted to authenticate with the user's password
    *   Check if you have an authorized_keys file in ~/.ssh/
4.  Paste the Public Key: Paste the public key you copied from PuTTYgen into the `authorized_keys` file. Save the file.

**Exporting the Private Key and Setting Permissions:**

1.  Export to OpenSSH Format: In PuTTYgen, go to "Conversions" -> "Export OpenSSH key." Save the private key to a secure location on your local machine (e.g., `C:\Users\YourUser\.ssh\id_rsa`).
2.  Set File Permissions (Important!): The private key file *must* have restricted permissions. If not, SSH will refuse to use it.
    *   Right-click the private key file in File Explorer and select "Properties."
    *   Go to the "Security" tab and click "Advanced."
    *   Click "Disable inheritance." Choose to "Remove all inherited permissions from this object."
    *   Click "Add." Enter the username of your Windows account and click "OK."
    *   Grant your user account "Full control."
    *   Click "OK" to save the changes. The only user with access to the private key file should be your Windows account.

**Connecting with Your IDE (e.g., Visual Studio Code):**

1.  Configure SSH Connection: In your IDE's SSH settings, specify:
    *   The VM's IP address.
    *   Your Ubuntu username.
    *   The path to your *private* key file (e.g., `C:\Users\YourUser\.ssh\id_rsa`).
2.  Test the Connection: Attempt to connect to the VM via SSH. You should *not* be prompted for a password. If you are, double-check all the previous steps, especially the key copying and file permissions.

**Disabling Password Authentication (Important Security Measure):**

1.  Edit the SSHD Configuration File: Using Visual Studio Code (or your preferred IDE), connect to the VM via SSH and open the SSH configuration file: `/etc/ssh/sshd_config`

2.  Find and Modify the following line:

    *   Change `PasswordAuthentication yes` to `PasswordAuthentication no`.
3.  Save the changes to the `sshd_config` file.
4.  Restart the SSH Service: Using VS Code's integrated terminal, run the following command:

    ```bash
    sudo systemctl restart sshd
    ```

**Important Considerations:**

*   **sshd_config.d Directory:** Be aware that configurations in the `/etc/ssh/sshd_config.d/` directory can *override* settings in the main `sshd_config` file. Check these files to ensure password authentication is truly disabled.

**Final Thoughts:**

I hope this guide helps you set up your secure Ubuntu server in Hyper-V with ease. Happy coding!
