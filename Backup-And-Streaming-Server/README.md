# Backup Guide

This guide provides instructions on how to set up and use `rsync` for backing up a project. The backup process involves utilizing `rsync` both locally and over the network between two virtual machines (VMs) running CentOS 7 and Ubuntu.

## Local Backup Using rsync

### Ubuntu VM Setup

1. Install Git:
   ```
   sudo apt-get install git
   ```
2. Clone the E-Learning Platform project:
   ```
   git clone https://github.com/rv0fficial/E-Learning-Platform.git
   ```
3. Install rsync:
   ```
   sudo apt-get install rsync
   ```
4. Create a backup directory:
   ```
   mkdir Desktop/backup
   ```
5. Perform the backup using rsync:
   ```
   rsync -zvr E-Learning-Platform/ Desktop/backup/
   ```

- `-z`: This option enables compression during data transfer, which can help reduce the amount of data sent over the network, especially for large files.

- `-v`: This option stands for "verbose" and makes `rsync` display detailed information about the files being synced, showing progress and any errors encountered.

- `-r`: This option stands for "recursive" and tells `rsync` to sync directories and their contents recursively, meaning it will sync all files and subdirectories within the specified directory.


Ensure to include the trailing '/' after "E-Learning-Platform" to avoid creating an unnecessary folder.

### Understanding rsync Efficiency

- **Preserving Timestamps and Permissions**: Use the `-a` option to preserve ownership, permissions, and timestamps during backup.
  ```
  rsync -avz simple-php-website/ Desktop/backup/
  ```

## Network Backup Using rsync

In this part try to use rsync to upload or backup files to a remote location. The remote location is our Centos machine.

### CentOS VM Setup

1. Create a backup directory:
   ```
   mkdir Desktop/backup
   ```

### Ubuntu VM Setup

1. Perform rsync over the network to the CentOS VM (replace `<hostname>` with the actual hostname or IP address):
   ```
   rsync -avz E-Learning-Platform/ <hostname>:/home/rv/Desktop/backup
   ```
   Then enter the remote server user's password.

2. Retrieving the removed data:

   To test this remove certain files and run the following command.

   ```
   sudo rsync -avz <destination_username>@<hostname>:/home/rv/Desktop/backup/ /home/rv/Desktop/E-Learning-Platform/
   ```

Replace `<hostname>` with the appropriate hostname or IP address of your target machine.

## Advanced SSH Options with rsync

### Ubuntu VM Setup

if you have noticed in the previously when we used the SSH command we were forced to enter the password whenever our sync was used. 
so what if you want to run or sync in an unattended mode?

1. Generate SSH key pair:
   ```
   ssh-keygen
   ```
   Press `enter` three times.

2. Copy the public key to the CentOS VM:
   ```
   ssh-copy-id <hostname>
   ```
3. Install and start SSH service (if not already installed):
   ```
   sudo apt install openssh-server
   sudo systemctl enable ssh
   sudo systemctl start ssh
   ```
   If you want to connect Centos to Ubuntu VM, you have to install this and perform steps 1 and 2 in the Centos 

4. Automate backup using cron:
   ```
   crontab -e
   0 0 * * * rsync -avz /home/rv/Desktop/E-Learning-Platform/ <hostname>:/home/rv/Desktop/backup
   ```

Replace `<hostname>` with the appropriate hostname or IP address of your target machine.

## Advanced rsync Over SSH

### CentOS VM Setup

when you want to connect to the remote machine over SSH but without using the default port which is 22

1. Modify SSH configuration to use a custom port (e.g., 2222):
   ```
   sudo vim /etc/ssh/sshd_config
   ```
   - Uncomment the line `Port 22` and change it to `Port 2222`.
   - Restart SSH service:
     ```
     sudo service sshd restart
     ```
2. Adjust firewall settings:
   ```
   sudo firewall-cmd --permanent --add-port=2222/tcp
   sudo firewall-cmd --reload
   ```
3. Update SELinux settings (CentOS only):
   ```
   sudo semanage port -a -t ssh_port_t -p tcp 2222
   ```

### Ubuntu VM Setup

1. Connect to CentOS VM on port 2222:
   ```
   ssh -p 2222 <hostname>
   ```
2. Perform rsync over SSH on port 2222:
   ```
   rsync -avz -e "ssh -p 2222" /home/rv/Desktop/E-Learning-Platform/ <hostname>:/home/rv/Desktop/backup
   ```

Follow these instructions to effectively back up your E-Learning Platform project using rsync across CentOS and Ubuntu VMs, locally and over the network. Replace `<hostname>` with the appropriate hostname or IP address of your target machine.

## Advanced rsync Backup Options

In certain scenarios, when transferring large files over a network, it's helpful to monitor the progress of the transfer and identify any potential issues. `rsync` provides the `--progress` option for this purpose.

### Monitoring Transfer Progress

To create an intentionally large file and monitor its transfer progress:

1. Create a 1GB file:
   ```
   sudo dd if=/dev/zero of=data.bin bs=102400 count=10240
   ```

2. Perform the rsync operation with progress monitoring:
   ```
   rsync -avz --progress -e "ssh -p 2222" /home/rv/Desktop/E-Learning-Platform/ <hostname>:/home/rv/Desktop/backup
   ```

This command displays a progress bar indicating the transfer status.

### Intelligent Delete Option

The `--delete` option enables intelligent deletion, ensuring files removed from the source are also removed from the destination.

1. Remove a file from the source:
   ```
   sudo rm -rf Desktop/E-Learning-Platform/data.bin
   ```

2. Perform rsync with intelligent delete:
   ```
   rsync -avz --delete -e "ssh -p 2222" /home/rv/Desktop/E-Learning-Platform/ <hostname>:/home/rv/Desktop/backup
   ```

### Syncing Existing Files Only

The `--existing` option ensures only files existing on both the source and destination are synchronized.

```
rsync -avz --existing -e "ssh -p 2222" /home/rv/Desktop/E-Learning-Platform/ <hostname>:/home/rv/Desktop/backup
```
Any new files created on the source (If I created any new file in ubuntu VM) are not going to be transferred as long as they are not present at the destination 

### Comparing Directories

To view differences between two directories without syncing them:

```
rsync -avzi -e "ssh -p 2222" /home/rv/Desktop/E-Learning-Platform/ <hostname>:/home/rv/Desktop/backup
```

### Including and Excluding Specific Files

To include or exclude specific file types during synchronization:

1. Create a directory and files in the `E-Learning-Platform`
   ```
   sudo mkdir images
   sudo touch file{1..10}.jpg
   touch file11.php
   ```

2. Include only `.php` files and exclude `.jpg` files:
   ```
   rsync -avz --include '*.php' --exclude '*.jpg' -e "ssh -p 2222" /home/rv/Desktop/E-Learning-Platform/ <hostname>:/home/rv/Desktop/backup
   ```
   As you can see it is going to transfer the image directory as it is and it is going to transfer only files added to the PHP. 
   Now you are interested in transferring only the `jpg` files the ones ending in `.jpg` to the remote machine or the remote destination (use following step).
   
3. Include only `.jpg` files and exclude `.php` files:
   ```
   rsync -avz --include '*.jpg' --exclude '*.php' -e "ssh -p 2222" /home/rv/Desktop/E-Learning-Platform/ <hostname>:/home/rv/Desktop/backup
   ```

Ensure to replace `<hostname>` with the appropriate hostname or IP address of your target machine.
