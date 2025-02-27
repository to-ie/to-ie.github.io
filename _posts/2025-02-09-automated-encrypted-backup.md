---
layout: post
title: "Automated Encrypted Backups to S3 with Duplicity on Ubuntu" 
categories: self-hosted
date : 2025-01-18 09:14:50
---

Backing up your data securely is crucial, especially when storing it in the cloud. This guide explains how to set up automatic, encrypted backups of your hard drive to AWS S3 using Duplicity on Ubuntu.

## Installing the Required Tools

First, install the necessary packages:

```
sudo apt update && sudo apt install duplicity gnupg python3-boto3 -y 
```
- `duplicity` - handles the backup
- `gnupg` - encrypts the backup
- `boto3` - allows S3 communication

## Creating a GPG Key for Encryption

Run the following to create a 4096-bit RSA key:

```
gpg --full-generate-key
```

List your key:
```
gpg --list-keys
```

- Note your Key ID, as you’ll need it for encryption.

## Configuring AWS S3

Install AWS CLI manually (if needed):

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Configure AWS credentials:
```
aws configure
```

Create an S3 bucket:
```
aws s3 mb s3://my-secure-backup-bucket
```

## Automating Backups

Create a backup script (`backup.sh`):
```
#!/bin/bash
GPG_KEY="ABCDEFG1234567890"
SOURCE_DIR="/mnt/my-hard-drive"
S3_BUCKET="s3://my-secure-backup-bucket"

export PASSPHRASE="your-strong-passphrase"

duplicity --encrypt-key "$GPG_KEY" "$SOURCE_DIR" "$S3_BUCKET"

unset PASSPHRASE
```

Make it executable:
```
chmod +x backup.sh
```

Automate with Cron:
Run the backup weekly at 2 AM:
```
crontab -e
```
Add:
```
0 2 * * 0 /home/user/backup.sh >> /var/log/backup.log 2>&1
```

## Extra Security Measures

- Store your GPG private key somewhere safe:
```
gpg --export-secret-keys -a "your-email@example.com" > my-private-key.asc
```
- Use AWS IAM roles with minimal permissions.
- Enable S3 versioning in case of accidental deletions.

## Restoring Backups

Check If Your GPG Key Exists:
```
gpg --list-keys
```
If your key is not listed, restore it from your backup:
```
gpg --import my-private-key.asc
```

To restore everything:
```
duplicity restore --decrypt-key "ABCDEFG1234567890" s3://my-secure-backup-bucket /mnt/restore-location
```

To restore a specific file:
```
duplicity restore --file-to-restore "path/to/file" s3://my-secure-backup-bucket /mnt/restore-location/file
```

To restore a previous version from 3 days ago:
```
duplicity restore --time "3D" s3://my-secure-backup-bucket /mnt/restore-location
```


