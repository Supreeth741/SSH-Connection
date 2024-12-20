# Setting Up SSH Access to Your Linux System Using Cloudflare Tunnel

## Understanding the Process

Before we start, let's break down what we're doing:

1. **Cloudflare Tunnel:** This is a service provided by Cloudflare that creates a secure tunnel between your server and the internet.
2. **SSH:** This is a protocol that allows you to securely connect to your server and run commands remotely.

By combining these two, we'll be able to access your Linux system remotely, even if it's behind a firewall or has a dynamic IP address.

## Step-by-Step Guide

### 1. Set Up Cloudflare Tunnel

#### a. Create a Cloudflare Account:
- If you don't have one, sign up for a free Cloudflare account.

#### b. Install Cloudflare Tunnel:
- Download the `cloudflared` binary for your Linux distribution from the Cloudflare website.
- Follow the installation instructions for your specific distribution.

#### c. Configure Cloudflare Tunnel:
- Run the following command to generate a tunnel token:
  ```bash
  echo "Running: cloudflared tunnel login"
  cloudflared tunnel login
  ```
- Follow the on-screen instructions to create a tunnel.

### 2. Configure Your Linux System

#### a. Install SSH Server (if not already installed):
- On Debian/Ubuntu:
  ```bash
  echo "Installing OpenSSH server on Debian/Ubuntu"
  sudo apt install openssh-server
  ```
- On CentOS/RHEL:
  ```bash
  echo "Installing OpenSSH server on CentOS/RHEL"
  sudo yum install openssh-server
  ```

#### b. Start the SSH Service:
- Start the SSH service:
  ```bash
  echo "Starting SSH service"
  sudo systemctl start ssh
  ```
- Enable the SSH service to start automatically on boot:
  ```bash
  echo "Enabling SSH service to start on boot"
  sudo systemctl enable ssh
  ```

#### c. Allow SSH Through the Firewall (if applicable):
- If you have a firewall (like `ufw` or `firewalld`), you'll need to allow incoming SSH connections.
  - For `ufw`:
    ```bash
    echo "Allowing SSH through ufw firewall"
    sudo ufw allow ssh
    ```
  - For `firewalld`:
    ```bash
    echo "Allowing SSH through firewalld"
    sudo firewall-cmd --permanent --zone=public --add-port=22/tcp
    echo "Reloading firewalld configuration"
    sudo firewall-cmd --reload
    ```

### 3. Create a Cloudflare Tunnel for SSH

#### a. Log into the Cloudflare Dashboard:
- Go to the Zero Trust > Tunnels section.

#### b. Create a New Tunnel:
- Click "Create a Tunnel."
- Choose "Cloudflared" as the connection method.

#### c. Add a Public Hostname:
- Under the tunnel settings, click "Add a Public Hostname."
- Enter a desired hostname (e.g., `ssh.yourdomain.com`).
- Set the service type to "SSH."
- Set the service to point to `localhost:22`.

### 4. Connect to Your Linux System

#### a. Start the Cloudflared Tunnel:
- Run the following command in your terminal:
  ```bash
  echo "Starting Cloudflared tunnel"
  cloudflared tunnel run
  ```

#### b. Connect to the Public Hostname:
- Use an SSH client (like `ssh` or a terminal emulator) to connect to the public hostname you created in step 3c:
  ```bash
  echo "Connecting to the Linux system via SSH"
  ssh user@ssh.yourdomain.com
  ```
- Replace `user` with your username on the Linux system.

## Additional Tips

- **Security:**
  - Use strong passwords or SSH keys for authentication.
  - Consider using two-factor authentication for added security.
- **Firewall Rules:**
  - Ensure that your firewall rules allow traffic to the port where Cloudflared is listening.
- **Cloudflare Access:**
  - For advanced security, you can integrate Cloudflare Access to control access to your SSH tunnel.

By following these steps, you should be able to securely access your Linux system remotely through Cloudflare Tunnel.
