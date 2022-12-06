---
layout: post
title: "Applying an SSL Certificate to Qlik Sense Entreprise " 
categories: tech
date : 2022-12-06 10:13:50
---

I spent too much time scouting the web for a solution to this so I decided to write a little tutorial about it, as most posts out there are incomplete. 

## Generate a private key and CSR
RDP into the EC2 instance your Qlik Enterprise is rolled out onto and open Powershell. 

```
openssl req -new -newkey rsa:2048 -nodes -keyout server.key -out server.csr
```

This command creates a private key file named server.key and a CSR named server.csr. You can change these filenames to anything you want.

You will have to fill in a few details about the CSR. 

Note: At the Common Name prompt, type the domain name that you want to secure with the SSL certificate, and then press Enter. The common name is often simply your domain name, such as example.com. Or, if you are going to install an SSL certificate for a subdomain, subdomain.example.com. 

Make sure you know where these files are saved, you will need them later on.

## Purchase a SSL Certificate

I used a PositiveSSL DV cert from [Positive SSL](https://store.positivessl.com).

Go through the steps to validate your SSL Cert:
- Submit the CSR you just created
- Validate the domain 
- Receive your certificate by email. 

## Create .pfx file from certificate and private key

This is the part I kept missing. [This](https://community.qlik.com/t5/Official-Support-Articles/How-to-change-the-certificate-used-by-the-Qlik-Sense-Proxy-to-a/ta-p/1712773) post outlines the requirements of the cert. 

You now have a certificate (.crt file) and a key (.key file). Before importing the cert onto the instance, you need to combine the two into a .pfx file.

```
openssl pkcs12 -export -out domain.name.pfx -inkey server.key -in server.crt
```

## Import the certificate and the key (.pfx)

1. Launch the MMC (in Powershell, type 'mmc')
2. When the MMC opens go to File|Add/Remove Snap-in. 
3. Click on the Certificates snap-in on the left side list box and click the add button. 
4. Choose Computer account and click Next. 
5. Leave Local computer selected and click Finish. 
6. Click OK to go back to the MMC. 
7. The Local computer certificate store opens Right click on the Personal folder and choose Import.
8. Because we have specified the location to import a certificate, the store location choice is greyed out. Click Next.
9. Browse to the location of the certificate containing private key (the pfx file) and click Next. 
10. Enter the password set during certificate merge process. Make sure to un-tick the box `Mark this key as exportable`
11. Prompted with the Certificate store to place the certificate, make sure the radio button is highlighted for Place all cert... and the store is Personal. Click Next.
12. Click Finish and the certificate will be imported. 
13. Double click on the imported certificate and choose details.
14. Navigate down the list and click on Thumbprint.
15. Copy the thumbprint with spaces from the textbox. 
16. Browse to the Qlik Sense QMC. 
17. Click on the Proxies menu item.
18. Choose the appropriate proxy to input the thumbprint to use the 3rd party certificate. Click edit to open the proxy settings. 
19. Click on the security option and enter the thumbprint (with spaces) in the textbox. Click Apply. The QMC will ask to restart. Please click the Restart QMC button. You may be required to refresh the browser as well.

(this last part is taken from [Aginic](https://support.aginic.com/support/solutions/articles/14000031148-applying-an-ssl-certificate-to-qlik-sense) )



