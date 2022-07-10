---
{"dg-publish":true,"permalink":"/obsidian-git-home/","tags":"gardenEntry","dgHomeLink":true,"dgPassFrontmatter":false}
---

[![Netlify Status](https://api.netlify.com/api/v1/badges/73cab77a-2258-4791-b877-151289fe70ed/deploy-status)](https://app.netlify.com/sites/ns-dhaliwal/deploys)
[[obsidian-git-pdf|obsidian-git-pdf]]

# Configure SAS Web Server Manually for HTTPS

[Use of TLS with SAS Web Applications](https://documentation.sas.com/doc/en/bicdc/9.4/bimtag/n0nakjyj6hlqmvn11p9p04l25j9n.htm#n09d8bbr6uq45xn1nfqt1q0c7fie)

[Reconfigure to Use HTTPS](https://documentation.sas.com/doc/en/bicdc/9.4/bimtag/n0nakjyj6hlqmvn11p9p04l25j9n.htm#n10aqkb89lg6c0n1ksyagv6g0ydn)

## Use of TLS with SAS Web Applications

Transport Layer Security (TLS) is a successor protocol to SSL. It is used to provide network security and privacy. In addition to providing encryption services, TLS uses trusted certificates to perform client and server

, and it uses message authentication codes to ensure data integrity.

This documentation assumes that you have a basic understanding of TLS and that you know how to obtain and use trusted certificates.

The best practice is to acquire CA-signed certificates before you install and configure SAS software. You can specify the location of the certificate to the SAS Deployment Wizard and it can configure SAS Web Server to use it. For more information, see [Use HTTPS](https://documentation.sas.com/doc/en/bicdc/9.4/bimtag/n1j5pntraygvvcn1wk9r3svhqo8k.htm#p0sp0ef2jxyeuon1cntg7xzz9npg).

Note: Starting at SAS 9.4M5, it is required that you configure SAS Environment Manager for HTTPS. For more information, see [Configure SAS Environment Manager for HTTPS Starting at SAS 9.4M5](https://documentation.sas.com/doc/en/bicdc/9.4/bimtag/p1fpnnm9hxkhlzn1x5tkqs1caeg5.htm#n1q6bzohmbc5a2n1mchl7lfziqo8).

## Reconfigure to Use HTTPS

The following table shows when you should revert manual TLS changes to SAS Web Server if you are upgrading.

When to Revert Manual TLS Changes to SAS Web Server during an Upgrade

SAS Release

Upgrade Information

SAS 9.4M4 and later

You do not need to revert manual TLS changes to SAS Web Server if you are upgrading.

If you perform an update in place, you might be required to manually reset the TLS settings in the SAS environment file. For more information, see [Step 14](https://documentation.sas.com/doc/en/bicdc/9.4/bimtag/n0nakjyj6hlqmvn11p9p04l25j9n.htm#n1fdubtqv5rvfmn1g6ig92mz06ad).

You must determine which solutions, if any, need to have their manual TLS settings for WebDAV and connection properties reset. This is because some solutions do not preserve manual TLS settings for the WebDAV repository and connection properties. To do this review, complete [Step 8](https://documentation.sas.com/doc/en/bicdc/9.4/bimtag/n0nakjyj6hlqmvn11p9p04l25j9n.htm#n1rzr3sp57j1djn17lhs3d83wvfv) and [Step 9](https://documentation.sas.com/doc/en/bicdc/9.4/bimtag/n0nakjyj6hlqmvn11p9p04l25j9n.htm#n1brrv6x40x4krn1cxbkb8eohfi9).

SAS 9.4M3 and earlier

You must revert the manual TLS configuration changes to the original non-TLS values before applying any maintenance releases or upgrades to the system.

If you did not follow the steps for configuring manual HTTPS as documented in this guide, you must revert those changes as well. For more information, see [Revert Manual HTTPS Changes to SAS Web Server](https://documentation.sas.com/doc/en/bicdc/9.4/bimtag/n0haeml26dr4fmn1it9ojxhzf57x.htm). Once maintenance or upgrades have been applied, the manual TLS configuration steps can be reapplied to the upgraded system.

If you did not choose to configure with secure sockets during the initial installation and configuration with the SAS Deployment Wizard, you can manually configure SAS Web Server to use HTTPS. Follow these steps:

1.  Create a private key, generate a certificate signing request, get a signed certificate, and create a certificate chain. For more information, see [Encryption in SAS](https://documentation.sas.com/doc/en/bicdc/9.4/secref/titlepage.htm).
    
2.  Stop SAS Web Server and all SAS Web Application Server instances.
    
3.  If the directory `SAS-configuration-directory\Levn\Web\WebServer\ssl` does not exist, then create it.
    
    Put the certificate file and key file in this directory. Be sure to change the default permissions of the .key file to be read-only for the user that SAS Web Server is configured to run as.
    
4.  Edit the `SAS-configuration-directory\Levn\Web\WebServer\conf\httpd.conf` file and make the following changes:
    
    1.  Remove the # from the following line:
        
        #Include C:\SAS\Config\Levn\Web\WebServer\conf\extra\httpd-ssl.conf
        
    2.  To use SAS Environment Manager to monitor SAS Web Server, locate the following line on Windows systems:
        
        Listen 80
        
        Replace the previous line with the following line:
        
        Listen localhost:80
        
        On UNIX systems, the port number is 7980.
        
    3.  Starting at SAS 9.4M6, remove the # from the following line:
        
        #LoadModule ssl_module  "SASHOME\SASWebServer\9.4\httpd-2.4\modules\mod_ssl.so"
        
    
5.  Edit the `SAS-configuration-directory\Levn\Web\WebServer\conf\extra\httpd-ssl.conf` file and make the following changes:
    
    1.  Locate the following line and make sure it refers to the HTTPS port that you want the server to listen on:
        
        Listen 443 https
        
        Note: Be aware that on UNIX platforms, you must start SAS Web Server as root in order to listen on ports below 1024.
        
    2.  Locate the following line and make sure it refers to the same HTTPS port:
        
        <VirtualHost _default_:443>
        
    3.  Locate the following lines for the certificate file and key file and enter the correct file names:
        
        SSLCertificateFile "ssl/myhost.crt"
        SSLCertificateKeyFile "ssl/myhost.key"
        SSLCertificateChainFile "ssl/myhost.crt
        
    
6.  (Optional) To verify that security has been configured correctly, start SAS Web Server. Then, access the secure SAS Web Server from your web browser.
    
7.  For each instance of SAS Web Application Server, edit the `SAS-configuration-directory\Levn\Web\WebAppServer\SASServern_m\conf\server.xml` file and follow these steps:
    
    1.  Make the following changes to the **Connector** element:
        
        -   Change the **proxyPort** attribute to specify the HTTPS listen port.
            
        -   Change the **scheme** to `https`.
            
        
    2.  In the **RemoteIpValve** element, if the httpServerPort, httpsServerPort, protocolHeader, and internalProxies options are present, make sure they are correct.
        
    
8.  Use
    

to update the protocol and port number for each web application. For more information, see [Specify Connection Properties](https://documentation.sas.com/doc/en/bicdc/9.4/bimtag/n1s9ty2ykoxyr5n1ikt28hcjwuuz.htm).

CAUTION

Do not modify the connection properties for the DP-SAS-Environment-Manager node. However, the connection properties for the Environment Mgr Mid-Tier node should be changed.

Note the following items:

-   When you log on to SAS Management Console to update the connection properties (by navigating to the Plug-ins tab and selecting Application Management![then select](https://documentation.sas.com/api/docsets/bimtag/9.4/content/images/arrow.gif?locale=en)Configuration Manager), view the Properties of each web application that is listed to determine whether connection information needs to be updated.
    
-   The connection properties for all Environment Mgr Mid-Tier applications should also be changed. Under Application Management![then select](https://documentation.sas.com/api/docsets/bimtag/9.4/content/images/arrow.gif?locale=en) Configuration Manager, expand the Environment Mgr Mid-Tier node and make connection changes for each application in this node.
    
-   The SAS
    

-   must be running for you to use SAS Management Console to modify the Themes Connection information.
    
-   To change the connection properties for SAS Visual Analytics, expand the sub-trees and apply the change to each SAS Visual Analytics application and
    
-   -   . Also, for Visual Analytics, change the connection properties for Search Interface to SAS Content.
        
    -   Some SAS users prefer to update these values using a SAS DATA step. This approach is beyond the scope of this task. If you do choose to modify these connections using a SAS script rather than SAS Management Console, the SAS_THEME table in SharedServices DB will not be modified. It is possible to manually update this database entry. However, the simplest solution is to use SAS Management Console to modify the Themes Connection, even if you use a script to configure the rest of these values.
        
    -   Change the port and protocol for SASTheme_default. From SAS Management Console, navigate to the Plug-ins tab and select Application Management![then select](https://documentation.sas.com/api/docsets/bimtag/9.4/content/images/arrow.gif?locale=en)Configuration Manager![then select](https://documentation.sas.com/api/docsets/bimtag/9.4/content/images/arrow.gif?locale=en)SAS Themes![then select](https://documentation.sas.com/api/docsets/bimtag/9.4/content/images/arrow.gif?locale=en)SASTheme_default. View the Properties to determine whether there is connection information that needs to be updated.
        
    
-   Use SAS Management Console to update the
    
-   connection information. For more information, see [Manual Configuration Tasks](https://documentation.sas.com/doc/en/bicdc/9.4/bimtag/n1uh9f02c7lsnzn1cu57z20h95m8.htm).
    
-   For SAS 9.4M2 and previous releases, if the certificate that you use is not signed by a certificate authority (CA) that would be located in the JRE default truststore (for example, VeriSign), then add all the CA certificates in the chain to the SAS Private JRE truststore (the cacerts file). Do this for all middle-tier machines before starting any servers.
    
    You must also import the certificate chain for server-tier machines to support any Java clients such as PROC SOAP. Also do this for client tier products.
    
    To add certificates into the SAS Private JRE truststore, follow these steps:
    
    1.  Know the location of your self-signed or site-signed certificate.
        
    2.  Import your self-signed or site-signed certificate into the SAS Private JRE default truststore (cacerts).
        
        Importing a certificate into a Java keystore or truststore is accomplished with the Java `keytool - importcert` command. The location of cacerts in the SAS Private JRE is as follows: `SAS-installation-directory/SASHome/ SASPrivateJavaRuntimeEnvironment/9.4/jre/lib/security`
        
        For example, on Windows, the command that you run is similar to this:
        
        cd C:\Program Files\SASHome\SASPrivateJavaRuntimeEnvironment\9.4\jre\lib\security
        ..\..\bin\keytool -importcert -keystore cacerts -file mycert.crt
        
        TipThe default password for cacerts is `changeit`. It is publicly documented on the Oracle website.
        
        For more information, see your Java documentation at [http://docs.oracle.com/javase/7/docs/technotes/tools/solaris/keytool.html](http://docs.oracle.com/javase/7/docs/technotes/tools/solaris/keytool.html "opens in a new window or tab").
        
    
    For SAS 9.4M3 and later, use the SAS Deployment Manager to import your CA certificates into the trusted CA bundle. You need to specify the location of your self-signed or site-signed CA certificate to the SAS Deployment Manager, and it updates the SAS Private JRE for you. For more information, see “Manage Certificates in the Trusted CA Bundle Using the SAS Deployment Manager” in [Encryption in SAS](https://documentation.sas.com/doc/en/bicdc/9.4/secref/titlepage.htm).
    
-   Configure the server tier and client tier.
    
    UNIX Specifics: For SAS 9.4M2 and earlier, for the server tier, you can create a base64 encoded certificate chain file that contains all trust certificates in the chain and use the file in the SSLCALISTLOC= SAS system option. Create the chain file by concatenating the individual CA base64 files. For more information, see Encryption in SAS.
    
    For SAS 9.4M3 and later, this step is no longer needed.
    
    Windows Specifics: For server and client tiers machines, add any required CA certificates to the Windows truststore.
    
-   Start each SAS Web Application Server instance.
    
-   For SAS Visual Analytics deployments, perform the following steps with SAS Management Console to confirm that the SAS LASR Authorization Service URI is updated:
    
    1.  Select Environment Management![then select](https://documentation.sas.com/api/docsets/bimtag/9.4/content/images/arrow.gif?locale=en)Server Manager.
        
    2.  For each SAS LASR Analytic Server, select the server to display the connection information in the right panel. Right-click the connection and select Properties.
        
    3.  Select the Options tab. Make sure the Use LASR authorization service check box is selected and that the URI includes the HTTPS protocol and port number. Click OK.
        
        Note: You must perform these steps so that the HTTPS connection information is saved in metadata.
        
        If the URI does not include the HTTPS protocol and port number, confirm that the LASRAuthorizationService REST and LASRKeyRegistrationService REST Connection protocol and port properties have been changed to HTTPS.
        
        1.  Select Application Management![then select](https://documentation.sas.com/api/docsets/bimtag/9.4/content/images/arrow.gif?locale=en)Configuration Manager![then select](https://documentation.sas.com/api/docsets/bimtag/9.4/content/images/arrow.gif?locale=en)SAS Application Infrastructure![then select](https://documentation.sas.com/api/docsets/bimtag/9.4/content/images/arrow.gif?locale=en)Visual Analytics version![then select](https://documentation.sas.com/api/docsets/bimtag/9.4/content/images/arrow.gif?locale=en)Visual Analytics Services version![then select](https://documentation.sas.com/api/docsets/bimtag/9.4/content/images/arrow.gif?locale=en)LASR Authenticaion Service version.
            
        2.  Right-click LASRAuthenticationService REST and select Properties.
            
        3.  Select the Connection tab. Change the Communication Protocol and Port Number to the appropriate HTTPS values. Click OK.
            
        4.  Repeat steps 2 and 3 for the LASRKeyRegistrationService REST component.
            
        5.  If you made changes to these components, repeat the preceding verification steps for each SAS LASR Analytic Server, to confirm that the URI is set correctly and is stored in metadata.
            
        
    
-   Depending on which products you have installed, you might have to update the `SAS-configuration-directory\Levn\Web\WebServer\htdocs\sas\sas-environment.xml` file. For more information, see [Customize the SAS Environment File](https://documentation.sas.com/doc/en/bicdc/9.4/bimtag/n0s325xuzaji9tn16zvgq128dvb5.htm#n1kvvcgkaw2r1an1oy64eha8sg3m).
    
-   To access SAS Environment Manager console with TLS enabled on SAS Web Server, edit the following files and locate the instances of the URLs listed below that begin with http://server:port. Modify them to point to https://server:ssl-port. The default port for HTTPS on Windows systems is 443. On UNIX systems, the default port is 8343.
    
    1.  Edit the `SAS-configuration-directory\Levn\Web\SASEnvironmentManager\server-version-EE\hq-engine\hq-server\webapps\ROOT\WEB-INF\spring\security-web-context.xml` file. Locate the following lines and enter the correct information for your environment in place of the highlighted text:
        
        <constructor-arg 
        value="https://server:ssl-port/SASLogon/
        logout?
        sas_svcs_logon_LogonUrl=http%3A%2F%2Fserver%3Aev-server-port%2F"/>
        <property name="loginUrl" 
        value="https://server:ssl-port/SASLogon"/>
        <property name="casUrl" 
        value="https://server:ssl-port/SASLogon"/>
        <bean id="casTicketValidator"
        class="com.sas.hyperic.security.CasIdentityRetrievingTicketValidator">
                    <constructor-arg
        value="https://server:ssl-port/SASLogon"/>
        
    2.  Edit the `SAS-configuration-directory\Levn\Web\SASEnvironmentManager\server-version-EE\hq-engine\hq-server\webapps\ROOT\WEB-INF\web.xml` file. Locate the following lines and enter the correct information for your environment:
        
        <param-value>https://server:ssl-port/SASWebDoc</param-value>
        <param-value>https://server:ssl-port/SASEnvironmentMgrMidTier</param-value>
        <param-value>https://server:ssl-port/SASLogon/TimedOut.do
        ?sas_svcs_logon_LogonUrl=http://server:ev-server-port/</param-value>
        
        Note: Depending on the version of SAS Environment Manager that is deployed in your environment, SASWebDoc might not be defined in the web.xml file.
        
    3.  For SAS 9.4M4 and previous releases, edit the `SAS-configuration-directory\Levn\Web\SASEnvironmentManager\server-version-EE\hq-engine\hq-server\webapps\ROOT\WEB-INF\classes\identity-service.properties` file. Locate the following line and enter the correct information for your environment:
        
        url.base=https\://server/
        
    4.  Restart SAS Environment Manager.
        
    
-   Update the SAS Content Server
    

1.  options with the new HTTPS URI values that were specified in [Step 15](https://documentation.sas.com/doc/en/bicdc/9.4/bimtag/n0nakjyj6hlqmvn11p9p04l25j9n.htm#n1rfh2tvdusl79n1v3mwjhr06tyx). Otherwise, users cannot access the SAS Content Server administration console.
    
    The following JVM options must be updated:
    ```bash
    -   -Dsas.scs.cas.host
        
    -   -Dsas.scs.cas.port
        
    -   -Dsas.scs.cas.scheme
        
    -   -Dsas.scs.svc.host
        
    -   -Dsas.scs.svc.port
        
    -   -Dsas.scs.svc.scheme
	```
    
    For a description of each JVM option and more information, see [SAS Content Server JVM Options](https://documentation.sas.com/doc/en/bicdc/9.4/bimtag/n1c7byu3scvmpan14wykb97lpy3i.htm#p16s0xss6clb0tn1eb8blbfnnwvj).
    
    For more information about how to update the JVM options, see [Specify JVM Options](https://documentation.sas.com/doc/en/bicdc/9.4/bimtag/p1qwfe2p0832wjn1d98p8f5y9sbx.htm#n1anib9gixijtun1uzrr9wimqzd4).
    

### See Also

[Encryption in SAS](https://documentation.sas.com/doc/en/bicdc/9.4/secref/titlepage.htm)

[SAS Intelligence Platform: Installation and Configuration Guide](https://documentation.sas.com/doc/en/bicdc/9.4/biig/titlepage.htm)

Last updated: April 28, 2022

---
created: 2022-07-07T20:11:13 (UTC +01:00)
tags: [flexible canvas app,Microsoft loop,Microsoft loop app,project collaboration,project organization,project organization app]
source: https://www.microsoft.com/en-us/microsoft-loop?ms.url=microsoftcommicrosoft-loop
author: 
---

# Microsoft Loop: Flexible Canvas App | Microsoft 365

> ## Excerpt
> Introducing Microsoft Loop, the new app that combines a flexible canvas with components that stay in sync and move freely across apps, allowing for better project collaboration and organization.

---
## Introducing Microsoft Loop

## Think, plan, and create - together

Microsoft Loop is a new app that combines a powerful and flexible canvas with portable components that stay in sync and move freely across Microsoft 365 apps.

![](https://img-prod-cms-rt-microsoft-com.akamaized.net/cms/api/am/imageFileData/RWOfkq?ver=e6f2&q=90&m=2&h=2147483647&w=2147483647&b=%23FFFFFFFF&aim=true)

### Work together, even when you’re apart

Cocreate, build on each other’s ideas, and add a little personality. Loop is an entirely new way to work together—designed for the hybrid world.

![](https://img-prod-cms-rt-microsoft-com.akamaized.net/cms/api/am/imageFileData/RWO6Hm?ver=41da&q=90&m=2&h=2147483647&w=2147483647&b=%23FFFFFFFF&aim=true)

### See everything in one place

Organize what you need for your project, such as files, links, and data from other apps, in a single workspace. Work in chat, meetings, or documents using portable components that are always up to date.

![](https://img-prod-cms-rt-microsoft-com.akamaized.net/cms/api/am/imageFileData/RWO6Hr?ver=98c0&q=90&m=2&h=2147483647&w=2147483647&b=%23FFFFFFFF&aim=true)

### Stay in sync and on track

Jump in, catch up, and get going with Loop. Connect naturally with emojis and expressions. Easily collaborate and track progress with notifications, highlighted changes, status labels, and task lists.

## Microsoft Loop meets you where you are

### **Loop pages**

Organize your Loop components and important elements in your project such as links, files, and data in Loop pages—flexible canvases that can grow to match the size of your ideas.

Loop page sneak peek thoughts and high five

  

### **Loop pages**

Organize your Loop components and important elements in your project such as links, files, and data in Loop pages—flexible canvases that can grow to match the size of your ideas.

## Check out the Microsoft Loop announcement made at Ignite

## What are news outlets saying?

### Microsoft adds Loop project-management app to Office | Protocol

### Microsoft launches Loop Office app for real-time collaboration | CNBC

### Microsoft Loop is a new Office app built for collaborative work | Engadget


# Step One: (Optional) Provision System Requirements

After you have a list of all the SAS products that you plan to roll out across your Windows enterprise, provision the necessary system requirements on which they depend.

[Determine System Requirements](http://go.documentation.sas.com/doc/en/bicdc/9.4/biig/n15003intelplatform00install.htm#n15008intelplatform00install)

[Add the System Requirements Commands to Your Provisioning System](http://go.documentation.sas.com/doc/en/bicdc/9.4/biig/n15003intelplatform00install.htm#n15009intelplatform00install)

[Provision the System Requirements](http://go.documentation.sas.com/doc/en/bicdc/9.4/biig/n15003intelplatform00install.htm#n15010intelplatform00install)

## Determine System Requirements

Review the [SAS 9.4 system requirements](http://support.sas.com/resources/sysreq/ "opens in a new window or tab").

However, if you alrea[[Getting Started fd9b1757bb8a4bd8b641012b6210345f|Getting Started fd9b1757bb8a4bd8b641012b6210345f]]dy have the SAS 9.4 software that you plan to provision on-site, you can run the

to analyze your system and identify which system requirements you must install. When you run the deployment wizard with the `-srwonly` option, the wizard creates a batch file that contains a list of the commands required to install the necessary system requirements. You then add these commands to your provisioning system in order to roll out the system requirements for SAS across your Windows enterprise.

For more information about the `-srwonly` option, see the [SAS Deployment Wizard and SAS Deployment Manager User’s Guide](http://support.sas.com/documentation/installcenter/en/ikdeploywizug/66034/PDF/default/user.pdf "opens in a new window or tab").

TipRemember that you might need to create multiple system requirements batch files when there are target groups that have different needs for specific deployments. For example, you might have three groups with different product needs: Group A needs SAS Enterprise Guide only. Group B requires SAS Enterprise Guide and SAS Add-In for Microsoft Office. Group C requires only SAS Foundation. Depending on the SAS products involved, different groups can have different system requirements.

To run the deployment wizard to create a batch file of system requirements commands, follow these steps:

1.  Make sure that the Windows operating systems that you are using support SAS 9.4. For more information, see [Supported Operating Systems](http://support.sas.com/supportos/list "opens in a new window or tab").
    
2.  Make sure that the machine that you choose for analyzing system requirements meets the following criteria:
    
    -   It is a clean machine on which no SAS products have been installed.
        
        If a version of SAS is already installed on the machine on which you are recording, you might see different results in your response file than you would see on a clean machine. Consequently, your results are different when you attempt the installation.
        
    -   It is a machine that is typical of machines that your end users use (for example, in its operating system, bitness, third-party software, drives, group policy, and so on.)
        
    
3.  Log on to the machine with the SAS Installer account (or with an account that is a member of the Windows Administrators group).
    
4.  Open a Windows command prompt and start the SAS Deployment Wizard from the highest-level directory in your
    

1.  using the following command: `setup.exe -srwonly`
    
    You should see a welcome page similar to the following:
    
    ![SAS Deployment Wizard splash screen](http://go.documentation.sas.com/api/docsets/biig/9.4/content/images/sdwwelcome.png?locale=en)
    
2.  On the Select System Requirements Task page, select List System Requirements to Install Later and click Next.
    
    The wizard does not install any of the system requirements when run with `–srwonly` and List System Requirements to Install Later. The wizard generates only a batch file that contains a list of commands to install system requirements.
    
    ![Select System Requirements Task page](http://go.documentation.sas.com/api/docsets/biig/9.4/content/images/sdw_srwonly.png?locale=en)
    
3.  Provide the information that the deployment wizard prompts you for.
    
4.  On the Select Products to Install page, select the SAS products that you plan to roll out across your Windows enterprise.
    
    ![Select Products to Install page](http://go.documentation.sas.com/api/docsets/biig/9.4/content/images/sdw_select_prods_install.png?locale=en)
    
5.  On the pages that follow, make selections for such prompts as language and regional settings support. Provide any additional information that the deployment wizard prompts you for.
    
    Note: Language and region settings can affect whether certain system requirements are needed on a particular machine.
    
6.  At the Deployment Summary page, click Start.
    
    ![Deployment Summary page](http://go.documentation.sas.com/api/docsets/biig/9.4/content/images/sdw_deployment_summary.png?locale=en)
    
    When the deployment wizard is finished, the Deployment Complete page appears.
    
7.  Review the Deployment Complete page.
    
    The icons on this page indicate the following:
    
    -   A green icon indicates that the particular system requirement was added to the batch file successfully.
        
    -   A yellow icon indicates that the particular system requirement was not needed on this machine, based on the Windows version and the language and encoding selections made.
        
    -   A red icon indicates that an error occurred when adding the particular system requirement to the batch file.
        
        ![Deployment Complete page](http://go.documentation.sas.com/api/docsets/biig/9.4/content/images/sdwdeploycomplete_srwonly.png?locale=en)
        
    
8.  When you are finished, click Next.
    
    The Additional Resources page appears and lists the location of the generated batch file under SAS System Requisite Batch File.
    
    ![Additional Resources page](http://go.documentation.sas.com/api/docsets/biig/9.4/content/images/sdw_add_resource_srwonly.png?locale=en)
    
    The wizard-generated batch file contains the following information:
    
    -   order number for which the system requirements check was performed
        
    -   Windows operating system for the machine
        
    -   list of SAS products on which the requirements check was performed
        
    
    ![a sample wizard-generated batch file of system requirements commands](http://go.documentation.sas.com/api/docsets/biig/9.4/content/images/srwonly_batch_file.png?locale=en)
    
9.  Click Finish to close the SAS Deployment Wizard.
    
10.  Proceed to the next section.
    

## Add the System Requirements Commands to Your Provisioning System

To add the system requirements commands to your provisioning system, follow these steps:

1.  Using a text editor, open the batch file that the deployment wizard generated in [Determine System Requirements](http://go.documentation.sas.com/doc/en/bicdc/9.4/biig/n15003intelplatform00install.htm#n15008intelplatform00install).
    
    The batch file is named SAS-order-number_sysreq_YYYY-MM-DD-HH-MM-SS.bat, and resides in `[drive]:\Users\SAS Installer\AppData\Local\SAS\SASDeploymentWizard`.
    
2.  Verify that the SAS Software Depot paths are accurate for each installation command, and that all machines on which you are provisioning SAS have access to these paths.
    
    Note: The commands contained in the batch file can be used on other machines than the one where the check was performed, but they should be used only with the same SAS software order, the same selected products, and on the same Windows operating system.
    
    [Sample System Requirements Batch File](http://go.documentation.sas.com/doc/en/bicdc/9.4/biig/n15003intelplatform00install.htm#n15013intelplatform00install) illustrates a sample batch file to install system requirements generated by the deployment wizard.
    
    Sample System Requirements Batch File
    
    ![Sample System Requirements Batch File](http://go.documentation.sas.com/api/docsets/biig/9.4/content/images/srwonly_batch_file.png?locale=en)
    
3.  Make sure the systems requirements commands work on your Windows machines in your enterprise environment.
    
4.  Add these system requirements commands to your provisioning system.
    
    For more information, consult your provisioning system documentation.
    
5.  Proceed to the next section.
    

## Provision the System Requirements

To provision the system requirements throughout your Windows enterprise, follow these steps:

1.  Run your provisioning system and
    

-   the SAS system requirements.
    
    Operating system reboots are not always required. However, reboots might be pending either from previous installations of products other than SAS or because one of the system requirements that you just installed requires a reboot.
    
    If you want to suppress a reboot, remove the Windows registry entries that cause reboots, such as `FileRenameOperations` or `PendingFileRenameOperations`. These entries are located here: `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SessionManager`.
    
-   After you provision the system requirements, rerun the deployment wizard without the `-srwonly` option. (For more information, revisit [Determine System Requirements](http://go.documentation.sas.com/doc/en/bicdc/9.4/biig/n15003intelplatform00install.htm#n15008intelplatform00install).)
    
    The Deployme<sup>nt Summary page sho</sup>uld no long`er list a sy`stems requirements stage. Stage 1 should now read Installation. This<sub> verifies that your systems requirements commands worked properly and tha</sub>t no more system requirements are needed.
    
    ![Deployment Summary page](http://go.documentation.sas.com/api/docsets/biig/9.4/content/images/sdw_summary_web_parts.png?locale=en)
    
-   Proceed to the next section.

<iframe src="https://google.com" width="800" height="600"
		</iframe>
		