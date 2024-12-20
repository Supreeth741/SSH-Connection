```markdown
# Step-by-Step Guide to Secure SSH Access with Cloudflare Tunnel

This document outlines the procedures to establish secure SSH connections using Cloudflare Tunnel. It primarily focuses on the recommended **SSH with Access for Infrastructure** method, while also detailing alternative approaches.

## 1. SSH with Access for Infrastructure (Recommended)

This is the most secure and feature-rich method, utilizing short-lived certificates and offering command logging.

### 1.1. Server Setup

1.  **Create a Cloudflare Tunnel**:
    *   Follow the official Cloudflare documentation to create a new tunnel via the dashboard.
    *   You can also create a tunnel via the command-line interface (CLI).

2.  **Configure Private Networks**:
    *   In the tunnel's **Private Networks** tab, add your server's private IP address or a range that encompasses it. This step ensures Cloudflare can route traffic to your server.
    
3.  **Install `cloudflared`**:
    *   Download and install the `cloudflared` daemon on your server. It's available as a standalone binary, Docker image, or through package managers like `apt` or `yum`.
    *   For example, on Debian-based systems, you might use:
        ```bash
        sudo apt update
        sudo apt install cloudflared
        ```
    *   For macOS using Homebrew:
         ```bash
         brew install cloudflared
         ```

4.  **Create an API Token**:
    *   In the Cloudflare dashboard, create an API token with the following permissions:
        *   Type: `Account`
        *   Item: `Zero Trust`
        *   Permission: `Edit`

### 1.2. Client Setup

1.  **Deploy WARP Client**:
    *   Install the Cloudflare WARP client on all devices that will need to connect to the SSH server.
2.  **Enable Gateway Proxy for TCP**:
    *   Ensure the Gateway proxy for TCP traffic is enabled on the client devices within the WARP client settings.
3. **Create Device Enrollment Rules**:
    *  Establish rules to manage which devices can enroll in your Zero Trust organization.

### 1.3. Route Private Network IPs through WARP

1.  **Configure Split Tunnels**:
    *   Set up split tunnels so that the IP/CIDR of your private network is routed through WARP. This is necessary because, by default, WARP excludes traffic to private IP addresses.

### 1.4. Configure Access for Infrastructure

1.  **Add a Target**:
    *   Navigate to **Zero Trust** > **Networks** > **Targets** in the Cloudflare dashboard.
    *   Click **Add a target**.
    *   Enter a **hostname** for the target resource and its IPv4 and/or IPv6 address. For example:
        *   Hostname: `my-ssh-server`
        *   IPv4: `192.168.1.100`
    *   If the IP address overlaps across multiple private networks, you will need to select the corresponding virtual network.

2.  **Add an Infrastructure Application**:
    *   Go to **Zero Trust** > **Access** > **Applications**.
    *   Select **Add an application** and choose **Self-hosted**.
    *   Enter an application name, for example, `SSH Access`.
    *   Under **Application domain**, specify a subdomain that will be used to reach the server.
    *   In **Target criteria**:
        *   Select the target hostname (e.g., `my-ssh-server`) created earlier.
        *   Set the **Protocol** to `SSH` and **Port** to `22` or the port your SSH server is listening on.
    *    **Set up an Access Policy**: Create a new access policy (e.g., "Allow Team Access") and define which users or groups can connect to the target. You can configure the usernames the users can log in with, or to let them log in with their email alias.
    *  Create an API token with the following permissions:
        * Type: `Account`
        * Item: `Access: Apps & Policies`
        * Permission: `Edit`

3.  **Generate Cloudflare SSH CA**:
    *   Generate a Cloudflare SSH CA using the API or use an existing one. You will need to use an API token with the following permissions:
        * Type: `Account`
        * Item: `Access: SSH Auditing`
        * Permission: `Edit`
    *   To create a new CA, use this curl command:
        ```bash
        curl --request POST \
          "https://api.cloudflare.com/client/v4/accounts/{account_id}/access/gateway_ca" \
          --header "Authorization: Bearer <API_TOKEN>"
        ```
    *   If a CA already exists you can get the public key using:
          ```bash
         curl https://api.cloudflare.com/client/v4/accounts/{account_id}/access/gateway_ca \
          --header  "Authorization: Bearer <API_TOKEN>"
         ```
    *   Copy the `public_key` value returned in the response.

### 1.5. Configure SSH Server

1.  **Save the Public Key**:
    *   On the remote target machine, change directories to the SSH configuration directory using:
        ```bash
        cd /etc/ssh
        ```
    *   Create or edit the `ca.pub` file using:
        ```bash
        sudo vim ca.pub
        ```
    *   Paste the copied `public_key` value into the `ca.pub` file and save.

2.  **Modify `sshd_config`**:
    *   Open the `/etc/ssh/sshd_config` file using:
        ```bash
        sudo vim /etc/ssh/sshd_config
        ```
    *   Uncomment the line `PubkeyAuthentication yes` by removing the `#`.
    *   Add a new line below `PubkeyAuthentication`:
        ```
        TrustedUserCAKeys /etc/ssh/ca.pub
        ```
    *   Save and close the file. You may need to use the command `:w !sudo tee % :q!` to save depending on your system permissions.

3.  **Restart SSH Service**:
    *   Restart the SSH service. The command may vary based on your OS. For example, on Debian/Ubuntu, use:
        ```bash
        sudo systemctl restart ssh
        ```
    *   On older Debian/Ubuntu use:
         ```bash
         sudo service ssh restart
         ```

4.  **Set File Permissions**:
    *   For some distributions, you might need to set the `ca.pub` file permissions to `600`:
    ```bash
    sudo chmod 600 /etc/ssh/ca.pub
    ```

### 1.6. Connect as a User

1.  **SSH Connection**:
    *   Users can connect to the SSH server using any SSH client, using the target IP address or domain, as long as they are logged into the WARP client.

    ```bash
        ssh <username>@<target_IP>
    ```

## 2. Other SSH Methods

Here’s a brief overview of the other SSH methods supported by Cloudflare:

### 2.1. Self-Managed SSH Keys

*   Use traditional SSH key pairs.
*   Require deploying the WARP client on the user's device.
*   Users can connect using their SSH keys when they are logged in to the WARP client.

### 2.2. Browser-Rendered SSH Terminal

*   Enables SSH access via a web browser.
*   Does not require the WARP client or managing SSH keys.
*   Traffic is proxied through a public hostname, and authentication uses Cloudflare Access credentials.
*   You need to add a public hostname with the SSH service type.

### 2.3. SSH with Client-Side `cloudflared` (Legacy)

*   Not recommended for new deployments.
*   Requires `cloudflared` to be installed on both the client and server.
*   Users connect via the command line and authenticate through a browser window.
*   Users will need to modify their SSH configuration file to use `cloudflared`.
* Example config changes:
    ```
    Host ssh.example.com
    ProxyCommand /usr/local/bin/cloudflared access ssh --hostname %h
    ```
    The `cloudflared` path will vary depending on your OS and package manager.

## 3. SSH Command Logs

*   SSH command logs capture the actual commands that users run on the target.
*   These logs are encrypted using a public key provided by the customer, and are not visible to Cloudflare.

### 3.1. Enable SSH Command Logging

1.  **Download the `ssh-log-cli` utility**: Download the utility from Cloudflare.
2.  **Generate a key pair**:
    ```bash
    ./ssh-log-cli generate-key-pair -o sshkey
    ```
    This will output `sshkey.pub` (public key) and `sshkey` (private key) files.
3.  **Upload the Public Key**:
    *   In Zero Trust, go to **Settings** > **Network**.
    *   Paste the content of `sshkey.pub` into **SSH encryption public key** and save.

### 3.2. Disable SSH Command Logging

1.  **Remove the Public Key**:
    *   In Zero Trust, go to **Settings** > **Network** > **SSH encryption public key**.
    *   Select **Remove** and confirm.

### 3.3. View SSH Logs

1.  **Retrieve Logs**:
    *   Go to **Logs** > **Access** in Zero Trust.
    *   Download the command log for a specific session.
2.  **Decrypt Logs**:
    *   Use the `ssh-log-cli` utility with the private key:
    ```bash
    ./ssh-log-cli decrypt -i sshlog -k sshkey
    ```
    This will output `sshlog-decrypted.zip` containing the decrypted logs.

## 4. General Steps

*   **Install `cloudflared`**: Install the `cloudflared` daemon on your server.
*   **Create a Subdomain**: Create a subdomain for your server and secure it with Cloudflare Access.

This comprehensive guide provides the necessary steps for setting up secure SSH access using Cloudflare Tunnel, emphasizing the recommended method for maximum security and control.
```
