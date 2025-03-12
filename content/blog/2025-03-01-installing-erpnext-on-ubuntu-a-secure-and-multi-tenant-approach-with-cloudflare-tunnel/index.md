---
title:  Installing ERPNext on Ubuntu, A Secure and Multi-Tenant Approach with Cloudflare Tunnel
description: "This post builds upon my previous guide for setting up a secure Ubuntu server in Hyper-V. Now, we'll install ERPNext, a powerful open-source ERP system, on that server. We'll leverage Cloudflare Tunnel for secure, SSL-encrypted access, and I'll show you how to set up a second sandbox ERPNext instance for testing and development."
date: 2025-03-01
lastmod: 2025-03-12
draft: false
tags:
    - Coding
type: default
---

This post builds upon my previous guide for setting up a secure Ubuntu server in Hyper-V. Now, we'll install ERPNext, a powerful open-source ERP system, on that server. We'll leverage Cloudflare Tunnel for secure, SSL-encrypted access, and I'll show you how to set up a second "sandbox" ERPNext instance for testing and development.

**Part 1: Exposing Your Server Securely with Cloudflare Tunnel**

Before installing ERPNext, we need a secure way to access the server. Cloudflare Tunnel provides a robust, SSL-encrypted connection without exposing your server's IP address directly to the internet.

1.  **Cloudflare Setup:**
    *   Log in to your Cloudflare account.
    *   Click "Websites" and select the domain you want to use for your ERPNext instance.
    *   Navigate to "Tunnels" under the "Traffic" section and click "Create a tunnel."
    *   Choose a name for your tunnel (e.g., `erpnext-tunnel`).
    *   Select the Linux/Debian option for your server.
2.  **Run the Cloudflare Tunnel Connector:**
    *   Cloudflare will provide a set of commands to run on your Ubuntu server.
    *   **Important:** Log in to your server via SSH as your regular user (the one you use for `sudo` commands). *Do not use the root user*.
    *   Copy and paste the provided commands into your server's terminal and run them. This installs the `cloudflared` connector.
3.  **Verify Tunnel Connection:**
    *   Back in the Cloudflare dashboard, you should see a connector listed in the "Connectors" section at the bottom of the page, with a status of "Connected."
    *   Click "Next."
4.  **Configure Public Hostnames:**
    *   This step defines how traffic is routed to your server.
    *   **Subdomain:** Choose a subdomain for your ERPNext instance (e.g., `erp`).
    *   **Domain:** Select your domain from the dropdown.
    *   **Type:** Select `HTTP`.
    *   **URL:** Enter `localhost`.
    *   Click "Save tunnel."

With the Cloudflare Tunnel configured, your server is now accessible via `https://erp.yourdomain.com` (replace `erp.yourdomain.com` with your actual subdomain and domain).

**Part 2: Installing ERPNext using the Easy Install Script**

Now that our server is securely accessible, let's install ERPNext. I recommend using the "ERPNext Quick Install" script for a streamlined setup.

1.  **SSH Access:** Ensure you're logged into your Ubuntu server via SSH as your regular user (the one that can use sudo).
2.  **Download and Run the Script:**
    *   Clone the ERPNext Quick Install repository and run it using the following set of commands:

    ```bash
    sudo apt update && sudo apt -y upgrade
    cd ~
    git clone https://github.com/flexcomng/erpnext_quick_install
    cd erpnext_quick_install
    chmod +x erpnext_install.sh
    source erpnext_install.sh
    ```
3.  **Follow the Prompts:** The script will guide you through the installation process. Here's a breakdown of the inputs you'll need to provide (adjust as needed):

    *   **ERPNext Version:** Choose the version you want to install (e.g., `3` for Version 15).
    *   **Continue?** `yes`
    *   **SQL Root Password:** `your_secure_sql_password` (choose a strong password)
    *   **Confirm Password:** `your_secure_sql_password`
    *   **Site Name:**  **Crucial:** Enter your fully qualified domain name (FQDN) that you configured in Cloudflare (e.g., `erp.yourdomain.com`). This is essential for SSL to work correctly.
    *   **Administrator Password:** `your_erpnext_admin_password` (choose a strong password)
    *   **Install ERPNext?** `yes`
    *   **Production Install?** `yes`
    *   **Install HRMS?** `yes` (or `no`, depending on your needs)
    *   **Install SSL?** `yes`
    *   **Email Address:** `your_email@example.com`

4.  **Post-Installation:**
    *   After the script completes, run the following commands (these are sometimes necessary to ensure proper Nginx configuration):

    ```bash
    sudo bench setup nginx
    sudo service nginx restart
    ```

You should now be able to access your ERPNext instance by visiting `https://erp.yourdomain.com` in your web browser.

**Part 3: Creating a Second "Sandbox" ERPNext Instance (Multi-Tenancy)**

It's highly recommended to have a separate ERPNext instance for testing, development, and exploring new features without risking your production data. Here's how to set up a second instance using ERPNext's multi-tenancy features:

1.  **Add a New Cloudflare Tunnel Public Hostname:**
    *   In Cloudflare, navigate to your tunnel configuration.
    *   Add a new public hostname entry for your second instance.
    *   **Subdomain:** Choose a different subdomain (e.g., `erp2`).
    *   **Domain:** Select your domain.
    *   **Type:** `HTTP`
    *   **URL:** `localhost` (both instances can share the same port)
    *   Save the changes.
2.  **Create a New ERPNext Site:**
    *   SSH into your server and navigate to the `frappe-bench` directory of ERPNext.
    *   Run the following command to create a new site:

    ```bash
    bench new-site erp2.yourdomain.com
    ```

    (Replace `erp2.yourdomain.com` with your actual subdomain and domain).
3.  **Enable DNS Multi-Tenancy:**

    ```bash
    bench config dns_multitenant on
    ```
4.  **Install ERPNext and HRMS on the New Site:**

    ```bash
    bench --site erp2.yourdomain.com install-app erpnext
    bench --site erp2.yourdomain.com install-app hrms
    ```

5.  **Restart Nginx:**

    ```bash
    bench setup nginx
    service nginx restart
    ```

You can now access your second ERPNext instance at `https://erp2.yourdomain.com`. You'll have a completely separate database and configuration, allowing you to experiment freely.

**Important Considerations:**

*   **sshd_config.d Directory:** Be aware that configurations in the `/etc/ssh/sshd_config.d/` directory can *override* settings in the main `sshd_config` file. Check these files to ensure password authentication is truly disabled.

**Final Thoughts:**

I hope this comprehensive guide empowers you to deploy ERPNext securely and efficiently, along with a dedicated sandbox environment.  Feel free to customize the installation further to suit your specific needs. I am confident this set up will prove to be invaluable as you explore the full potential of ERPNext!