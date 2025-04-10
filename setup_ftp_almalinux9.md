
# Setup FTP Server on AlmaLinux 9 (vsftpd)

This guide walks you through the complete setup of an FTP server on AlmaLinux 9 using **vsftpd**, including user creation, custom directories, permissions, and testing.

## Step 1: Install vsftpd

```
sudo dnf update -y
sudo dnf install vsftpd -y
sudo systemctl enable vsftpd
sudo systemctl start vsftpd
sudo systemctl status vsftpd
```

## Step 2: Configure Firewall

```
sudo firewall-cmd --permanent --add-service=ftp
sudo firewall-cmd --permanent --add-port=30000-31000/tcp
sudo firewall-cmd --reload
```

## Step 3: Backup Original Config

```
sudo cp /etc/vsftpd/vsftpd.conf /etc/vsftpd/vsftpd.conf.bak
```

## Step 4: Configure vsftpd

Edit configuration:

```
sudo nano /etc/vsftpd/vsftpd.conf
```

Update settings:

```
# Allow local users to log in
local_enable=YES

# Enable upload and write permissions
write_enable=YES

# Restrict users to their home directories (important!)
chroot_local_user=YES

# Enable passive mode
pasv_enable=YES
pasv_min_port=30000
pasv_max_port=31000

# Disable anonymous login
anonymous_enable=NO

# Enable logging (optional, good for troubleshooting)
xferlog_enable=YES

# Allow user list file (optional)
userlist_enable=YES
userlist_file=/etc/vsftpd/user_list
userlist_deny=NO
```


Save and exit.

## Step 5: Create FTP User with Custom Directory

```
sudo mkdir -p /var/ftpuser
sudo useradd -d /var/ftpuser -s /sbin/nologin ftpuser
sudo passwd ftpuser
sudo chown root:root /var/ftpuser
sudo mkdir -p /var/ftpuser/upload
sudo chown ftpuser:ftpuser /var/ftpuser/upload
```

## Step 6: Create an FTP User (Optional)
Although we can use any existing user to access the FTP server, it is recommended to create a dedicated FTP user who doesn’t have any sudo or admin access to test it. So, to create a new user on Almalinux, run the following commands to create a new user and set a password:

sudo adduser ftpuser
sudo passwd ftpuser

**Add your FTP user to VSFTPD user list:**\
If you have enabled the userlist_enable and userlist_deny options in your `/etc/vsftpd/vsftpd.conf`, ensure that your ftpuser user is listed correctly. Here’s how:
To allow FTP users to log into the server, Replace “ftpuser” in the command with the user you created for the FTP server.
## Step 7: Add User to Allowed List

```
echo "ftpuser" | sudo tee -a /etc/vsftpd/user_list
```


## 8: Create a directory for the new user, and adjust permissions(OPTIONAL)

```
sudo mkdir -p /home/ftpuser/ftp/upload
sudo chmod 550 /home/ftpuser/ftp
sudo chmod 750 /home/ftpuser/ftp/upload
sudo chown -R ftpuser: /home/ftpuser0/ftp
```
This creates a /home/ftpuser directory for the new user, with a special directory for uploads. It sets permissions for uploads only to the /uploads directory.

## Step 8: Restart vsftpd

```
sudo systemctl restart vsftpd
```

## Step 9: Test FTP Access

### Local Test:

```
ftp localhost/IP
# Use 'ftpuser' and the set password
```

List files and upload:

```
ls
cd upload
put testfile.txt
```

### Remote Test:

Use an FTP client (e.g., FileZilla):

- Host: `<server-ip>`
- Username: `ftpuser`
- Password: `<your-password>`
- Port: 21

## Step 10: Check Logs

```
sudo tail -f /var/log/vsftpd.log
sudo journalctl -u vsftpd
```

## Step 11: Optional SELinux Settings (if applicable)

```
sudo setsebool -P ftpd_full_access 1
sudo chcon -R -t public_content_rw_t /var/ftpuser
```

## Summary

- ✅ Installed vsftpd
- ✅ Configured securely
- ✅ Created user `/var/ftpuser`
- ✅ Set permissions
- ✅ Tested access and uploads

---

*Maintained by: Saiful Islam*
