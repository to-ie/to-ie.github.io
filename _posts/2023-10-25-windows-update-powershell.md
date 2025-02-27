---
layout: post
title: "Install Windows updates via PowerShell" 
categories: how-to
date : 2023-10-25 08:14:50
---

Keeping your Windows operating system up to date is crucial. Regular updates not only enhance system security but also improve performance and resolve various issues. Sometimes, you might find yourself in a situation where you need to force Windows updates remotely, via PowerShell

In this blog post, we will guide you through the process of installing Windows updates via PowerShell. Whether you're an IT professional or a tech-savvy user, this knowledge might come in handy someday.

## Step 1: Open PowerShell as an Administrator

Before you can start installing Windows updates, you'll need to open PowerShell with administrative privileges. To do this, follow these steps:

1.1. Click the Windows icon in the taskbar, or press the Windows key on your keyboard.

1.2. Type "PowerShell."

1.3. Right-click on "Windows PowerShell" in the search results.

1.4. Select "Run as administrator" from the context menu.

You may be prompted to confirm that you want to allow the app to make changes to your device. Click "Yes" to open PowerShell with the necessary permissions.

## Step 2: Install and import the modules required

This step might not be required if you already have the `PSWindowsUpdate` module installed. 

2.1 Allow script execution in this process.

```powershell
Set-ExecutionPolicy -Scope Process Bypass -Force 
```
Now script-based modules are allowed to run.

2.2 Install the required module

```powershell
Install-Module PSWindowsUpdate
```
2.3 Import it
```powershell  
Import-Module PSWindowsUpdate
```

## Step 3: Check for Available Updates

Now that you have PowerShell open with admin rights, it's time to check for available updates. Here's how to do it:

3.1. In the PowerShell window, type the following command and press Enter:

```powershell
Get-WindowsUpdate
```

This command will display a list of available updates, including their titles, update IDs, and more.

## Step 4: Install Windows Updates

With the list of available updates in front of you, you can now proceed to install them. Follow these steps:

4.1. To install all available updates, use the following command:

```powershell
Install-WindowsUpdate
```

4.2. If you only want to install a specific update, you can use its update ID. Use this command as a template:

```powershell
Install-WindowsUpdate -KBArticleID "KB123456"
```

Replace "KB123456" with the actual KB article ID of the update you want to install.

## Step 5: Restart Your Computer

After you've successfully installed the updates, it's essential to restart your computer to ensure they take effect. You can do this via PowerShell as well:

5.1. Type the following command in your PowerShell window and press Enter:

```powershell
Restart-Computer
```

This command will initiate a system restart, and your updates will be applied during the reboot.

## Conclusion

Keeping your Windows operating system up to date is crucial. PowerShell offers a powerful way to manage Windows updates, especially when you need to do it remotely. The steps outlined in this guide provide a simple and effective method to install updates via PowerShell, ensuring your system remains secure and performs optimally. 

Whether you're an IT professional or a Windows user looking to take control of your updates, mastering PowerShell can be a valuable skill in your tech toolbox. So, the next time you find yourself in a situation where you need to force Windows updates remotely, you'll be well-prepared to tackle the task. Happy updating!