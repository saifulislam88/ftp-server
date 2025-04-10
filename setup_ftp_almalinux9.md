
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
local_enable=YES
write_enable=YES
chroot_local_user=YES
pasv_enable=YES
pasv_min_port=30000
pasv_max_port=31000
anonymous_enable=NO
xferlog_enable=YES
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

## Step 6: Add User to Allowed List

```
echo "ftpuser" | sudo tee -a /etc/vsftpd/user_list
```

## Step 7: Restart vsftpd

```
sudo systemctl restart vsftpd
```

## Step 8: Test FTP Access

### Local Test:

```
ftp localhost
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

## Step 9: Check Logs

```
sudo tail -f /var/log/vsftpd.log
sudo journalctl -u vsftpd
```

## Step 10: Optional SELinux Settings (if applicable)

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

## Optional Improvements

- Enable TLS encryption for secure FTP
- Limit user bandwidth and concurrent connections
- Create upload-only accounts

---

*Maintained by: [Your Name]*
