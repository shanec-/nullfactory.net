---
layout: post
title: Creating a Self-Signed Wild Card SSL Certificate for Your Development Environment
category: Security
---

Secure Socket Layer (SSL) is a security standard used to ensure secure communication between a web server and browser and used in most modern web application. As a developer it is prudent to setup your development environment to closely resemble production as much as possible, including security concerns. However, getting a full fledged CA SSL certificate for you development environment might not be the most cost-effective solution. Therefore post summarizes the steps I take to create a self signed wild card certificate to be used in the internal environments. My guide is based on this [excellent post](https://www.macaw.nl/weblog/2013/6/configuring-an-asp-net-project-for-development-with-ssl).

## Create the Certificate

In order to create the certificate we would be using the `MakeCert.exe` tool which can be found at `C:\Program Files (x86)\Windows Kits\8.1\bin\x64\`. This command creates the certificate and adds it to the logged in user's personal certificate store:

    makecert -r -pe -e 01/01/2099 -eku 1.3.6.1.5.5.7.3.1 -ss My -n CN="*.wunder.local" -sky exchange -sp "Microsoft RSA SChannel Cryptographic Provider" -sy 12 -len 2048 

Some of the notable flags:

- **-r** - Indicates that we're creating a self-signed certificate.
- **-pe** - Includes the private key in the certificate and makes it exportable.
- **-e** - The validity period of the certificate.
- **-n** - The subject's certificate name - specify the wildcard url. 

	![Create Certificate](/images/posts/SSWildCardSSL/1_CreateCertificate.png)

<!--excerpt-->

### Verify Creation

1. Open a new `Microsoft Management Console (mmc.exe)`.
2. Add the `Certificates` Snap-In for `My Account`.
3. Under the `Personal` node, ensure that the newly created certificate exists.  

	![Verify Certificate](/images/posts/SSWildCardSSL/2_CreateCertificate.png)

### Export Certificate

Next we export the certificate, one with the private key (pfx) and one without (cer). 

1. Within the same snap-in, right click on the certificate and select export. 

	![Export Certificate Menu](/images/posts/SSWildCardSSL/3_ExportCertificate.png)

2. Select the option to export the private key. 
	
	![Export Pfx Wizard 1](/images/posts/SSWildCardSSL/4_ExportPfxWizard.png)

	![Export Pfx Wizard 2](/images/posts/SSWildCardSSL/5_ExportPfxWizard.png)

3. Provide a password to protect the private key. 
	
	![Export Pfx Wizard 3](/images/posts/SSWildCardSSL/6_ExportPfxWizard.png)

	![Export Pfx Wizard 4](/images/posts/SSWildCardSSL/7_ExportPfxWizard.png)

4. Provide a location to export the file.
	
	![Export Pfx Wizard 5](/images/posts/SSWildCardSSL/8_ExportPfxWizard.png)

	![Export Pfx Wizard 6](/images/posts/SSWildCardSSL/9_ExportPfxWizard.png)

5. Next let export the (.cer) file. Let's repeat the steps 1-4 but this time opting out of exporting the private key.

	![Export Cer Wizard 1](/images/posts/SSWildCardSSL/10_ExportCerWizard.png)

	![Export Cer Wizard 2](/images/posts/SSWildCardSSL/11_ExportCerWizard.png)

	![Export Cer Wizard 3](/images/posts/SSWildCardSSL/12_ExportCerWizard.png)

	![Export Cer Wizard 4](/images/posts/SSWildCardSSL/13_ExportCerWizard.png)

	![Export Cer Wizard 5](/images/posts/SSWildCardSSL/14_ExportCerWizard.png)
 
## Installation

Now, we install the pfx on the web server and each of the client machines consuming the web application.

#### Web Server

1. The easiest way to install the certificate is to right-click the certificate within explorer and select `Install PFX`.
	
	![Install Pfx Wizard 1](/images/posts/SSWildCardSSL/15_InstallPfxWizard.png)

2. This would launch the Certificate Import Wizard. Select the `Local Machine` as the certificate store.

	![Install Pfx Wizard 2](/images/posts/SSWildCardSSL/16_InstallPfxWizard.png)

	![Install Pfx Wizard 3](/images/posts/SSWildCardSSL/17_InstallPfxWizard.png)

3. Provide the password for the private key. 

	![Install Pfx Wizard 4](/images/posts/SSWildCardSSL/18_InstallPfxWizard.png)

4. Ensure that the certificate is installed under the `Personal` store.
	
	![Install Pfx Wizard 5](/images/posts/SSWildCardSSL/19_InstallPfxWizard.png)

	![Install Pfx Wizard 6](/images/posts/SSWildCardSSL/20_InstallPfxWizard.png)

	![Install Pfx Wizard 7](/images/posts/SSWildCardSSL/21_InstallPfxWizard.png)


Navigate to Internet Information Services (IIS) Manager and make sure that the certificate is visible.

![Verify Certificate in IIS](/images/posts/SSWildCardSSL/22_VerifyIIS.png)

Bind the certificate to the SSL Port and configure the web application as necessary.

#### Client Machines

Each of the client machines accessing the web application would have to trust the new certificate. This is done by adding the certificate to the the `Trusted Root Certification Authorities` store.  

1. On the client machine, right-click the certificate and selecting `Install` from the menu. On the wizard select the `Local Machine` as the store.

	![Install as Trusted Authority 1](/images/posts/SSWildCardSSL/23_InstallCertificate.png)

2. Add the certificate to the `Trusted Root Certification Authorities` store.

	![Install as Trusted Authority 2](/images/posts/SSWildCardSSL/24_InstallCertificate.png)

	![Install as Trusted Authority 3](/images/posts/SSWildCardSSL/25_InstallCertificate.png)

	![Install as Trusted Authority 4](/images/posts/SSWildCardSSL/26_InstallCertificate.png)

 
## Final Thoughts

- As suggested in the [original article](https://www.macaw.nl/weblog/2013/6/configuring-an-asp-net-project-for-development-with-ssl), this would be a good time check-in the certificates into source control so that the entire development team has access to the same files.
- While `Makecert.exe` was used to create the certificate, there are other options such as OpenSSL that would work just the same. 

## References
- [Configuring an ASP.NET project for development with SSL](https://www.macaw.nl/weblog/2013/6/configuring-an-asp-net-project-for-development-with-ssl)
- [Technet - Faisal (Sal) Bawany’s TechNet Blog - How to create a self-signed Wildcard SSL Certificate](http://blogs.technet.com/b/salbawany/archive/2014/05/24/how-to-create-a-self-signed-wild-card-ssl-certificate.aspx)
- [MSDN - Makecert.exe (Certificate Creation Tool)](https://msdn.microsoft.com/en-us/library/bfsktky3(v=vs.110).aspx)
- [Difference between MakeCert and OpenSSL wrt C# SslStream](http://stackoverflow.com/questions/26186780/difference-between-makecert-and-openssl-wrt-c-sharp-sslstream)
- [StackOverFlow - What's the difference between the Personal and Web Hosting certificate store?](http://stackoverflow.com/questions/26681192/whats-the-difference-between-the-personal-and-web-hosting-certificate-store)







