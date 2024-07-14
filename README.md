# V-Server Setup Guide

This guide covers the configuration of a V-Server, including the installation of Nginx, SSH setup with password authentication disabled, and adding a Git SSH key for secure repository interactions.

## Table of Contents

- [Configure V-Server](#configure-v-server)
- [Install Nginx](#install-nginx)
- [Setup SSH and Remove Password Authentication](#setup-ssh-and-remove-password-authentication)
- [Add Git SSH Key](#add-git-ssh-key)

## Configure V-Server

1. **Connect to your V-Server via SSH:**

    ```sh
    ssh username@your_server_ip
    ```

2. **Update package lists:**

    ```sh
    sudo apt update
    ```

## Install Nginx

1. **Install Nginx:**

    ```sh
    sudo apt install nginx -y
    ```

2. **Start Nginx and enable it to start on boot:**

    ```sh
    sudo systemctl start nginx
    sudo systemctl enable nginx
    ```

3. **Adjust firewall to allow Nginx traffic:**

    ```sh
    sudo ufw allow 'Nginx Full'
    sudo ufw enable
    ```

4. **Verify Nginx Installation:**

    After Nginx is running and the firewall is configured, you can check if Nginx is installed correctly and reachable. Open a web browser and enter the IP address of your VM. You should see the Nginx welcome page.

5. **Configure Nginx for Your Application:**

    If you want to deploy your own applications, you'll need to configure Nginx accordingly. The configuration files are located in `/etc/nginx`:

    - **Main Configuration File:** `/etc/nginx/nginx.conf`
    - **Sites-Available and Sites-Enabled** (only for Ubuntu/Debian):

        Create a new configuration file for your site in `/etc/nginx/sites-available/` and create a symbolic link in `/etc/nginx/sites-enabled/`.

        Example:

        ```sh
        sudo nano /etc/nginx/sites-available/my_site
        ```

        Add a simple configuration:

        ```nginx
        server {
            listen 80;
            server_name your_domain_or_ip;
            root /var/www/html;
            index index.html index.htm index.nginx-debian.html;

            location / {
                try_files $uri $uri/ =404;
            }
        }
        ```

        Then create the symbolic link:

        ```sh
        sudo ln -s /etc/nginx/sites-available/my_site /etc/nginx/sites-enabled/
        ```

    - **Test and Reload Nginx:**

        ```sh
        sudo nginx -t
        sudo systemctl reload nginx
        ```

## Setup SSH and Remove Password Authentication

1. **Generate an SSH key pair on your local machine:**

    For stronger security and better performance, use ED25519:

    ```sh
    ssh-keygen -t ed25519 -C "your_email@example.com"
    ```

    If you need compatibility with older systems, use RSA with a 4096-bit key:

    ```sh
    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    ```

2. **Add your SSH key to the SSH agent:**

    ```sh
    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_ed25519  # or ~/.ssh/id_rsa if using RSA
    ```

3. **Copy your public key to the V-Server:**

    ```sh
    ssh-copy-id username@your_server_ip
    ```

4. **Disable password authentication on the V-Server:**

    - Open the SSH configuration file:

        ```sh
        sudo nano /etc/ssh/sshd_config
        ```

    - Modify or add the following lines:

        ```sh
        PasswordAuthentication no
        ChallengeResponseAuthentication no
        UsePAM no
        PubkeyAuthentication yes
        ```

        **Reason:** Disabling password authentication and only allowing SSH key authentication greatly enhances security. Passwords can be guessed or brute-forced, especially if they're weak or reused across different services. SSH keys, on the other hand, are much more secure because they use cryptographic algorithms that are infeasible to crack with current technology.

    - Restart the SSH service:

        ```sh
        sudo systemctl restart ssh
        ```

## Add Git SSH Key

1. **Generate an SSH key pair on your V-Server:**

    ```sh
    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    ```

    **Reason:** Generating an SSH key pair (consisting of a private and public key) provides a more secure way to authenticate with GitHub compared to traditional username/password authentication. SSH keys use strong cryptographic algorithms that make it infeasible for attackers to guess or brute-force access.

2. **Add the SSH key to the SSH agent:**

    ```sh
    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_rsa
    ```

    **Reason:** Adding the SSH key to the SSH agent allows you to manage your keys more easily and securely. The SSH agent stores your keys in memory and provides them to the SSH client when needed, without requiring you to enter your passphrase every time you use the key.

3. **Copy the public key to your clipboard:**

    ```sh
    cat ~/.ssh/id_rsa.pub
    ```

4. **Add the SSH key to your GitHub account:**

    - Go to [GitHub SSH keys settings](https://github.com/settings/keys).
    - Click on `New SSH key`, provide a title (e.g., "V-Server SSH Key"), and paste the copied public key.
    - Click `Add SSH key`.

5. **Test the connection to GitHub:**

    ```sh
    ssh -T git@github.com
    ```

    You should see a message like:

    ```
    Hi your_username! You've successfully authenticated, but GitHub does not provide shell access.
    ```

Now your V-Server is configured with Nginx installed, SSH setup with password authentication disabled, and a Git SSH key added for secure interactions with your GitHub repositories.

## Troubleshooting

- Ensure your server's firewall allows SSH traffic.
- Verify the correct SSH key is being used if you encounter authentication issues.
- Check Nginx status if it's not serving web pages properly:

    ```sh
    sudo systemctl status nginx
    ```

For further assistance, refer to the documentation of your V-Server provider or reach out to the community for support.
