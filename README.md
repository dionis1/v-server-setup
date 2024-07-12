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

## Setup SSH and Remove Password Authentication

1. **Generate an SSH key pair on your local machine (if you don’t have one):**

    ```sh
    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    ```

2. **Add your SSH key to the SSH agent:**

    ```sh
    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_rsa
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

    - Restart the SSH service:

        ```sh
        sudo systemctl restart ssh
        ```

## Add Git SSH Key

1. **Generate an SSH key pair on your V-Server:**

    ```sh
    ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
    ```

2. **Add the SSH key to the SSH agent:**

    ```sh
    eval "$(ssh-agent -s)"
    ssh-add ~/.ssh/id_rsa
    ```

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

