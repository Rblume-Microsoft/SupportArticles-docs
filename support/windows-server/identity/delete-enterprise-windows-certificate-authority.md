---
title: How to remove manually Enterprise Windows Certificate Authority from a Windows 2000/2003 domain
description: Describes steps to delete manually Enterprise Windows Certificate Authority from a Windows 2000/2003 domain.
ms.date: 09/21/2020
author: Deland-Han
ms.author: delhan
manager: dscontentpm
audience: itpro
ms.topic: troubleshooting
ms.prod: windows-server
localization_priority: medium
ms.reviewer: kaushika
ms.prod-support-area-path: Active Directory Certificate Services
ms.technology: ActiveDirectory
---
# How to remove manually Enterprise Windows Certificate Authority from a Windows 2000/2003 domain

This article describes how to delete manually objects and data that belong to the original Enterprise Windows and reside in the Windows Active Directory.

_Original product version:_ &nbsp; Windows Server 2003  
_Original KB number:_ &nbsp; 555151

This article was written by [Yuval Sinay](https://mvp.microsoft.com/en-us/PublicProfile/7674?fullName=Yuval%20Sinay), Microsoft MVP.

## Symptoms

In some organizations, there are regular backup procedures for Enterprise Windows Certificate Authority. In case of server problem (software/hardware), there may need to reinstall the Enterprise Windows Certificate Authority. Before you can reinstall the Enterprise Windows Certificate Authority, you may need to delete manually objects and data that belong to the original Enterprise Windows and reside in the Windows Active Directory.

## Cause

Enterprise Windows Certificate Authority saves the configurations settings and data in the Windows Active Directory.

## Resolution

**A. Backup:**  

It's recommended that you back up all the nodes that contain active directory-related data: Windows Domain Controllers, Exchange Servers, Active Directory Connector, Windows Server with Services for Unix, ISA Server Enterprise, Enterprise Windows Certificate Authority - **before** and **after** you following this procedure.

The following procedure should be use as last resort and may impact on your production environment and may require to reboot some nodes/services.

**B. Active Directory Clean:**  

**Note:** Log on into the system with an account that has the permissions bellow:

1. Enterprise Administrator
2. Domain Administrator
3. Certificate Authority Administrator
4. Schema Administrator (The server that function as Schema Master FSMO should be online during the process).

To remove all Certification Services objects from Active Directory:

1. Start "**Active Directory Sites and Services**".
2. Click the "**View**" menu option, and select "**Show Services**" Node.
3. Expand the "**Services**", and then expand "**Public Key Services**".
4. Select the "**AIA**" node.
5. In the right-hand pane, locate the "**certificateAuthority**" object for your Certification Authority. Delete the object.
6. Select the "**CDP**" node.
7. In the right-hand pane, locate the Container object for the server where Certification Services is installed. Delete the container and the objects it contains.
8. Select the "**Certification Authorities**" node.
9. In the right-hand pane, locate the "**certificateAuthority**" object for your Certification Authority. Delete the object.
10. Select the "**Enrollment Services**" node.
11. In the right-hand pane, verify that the "**pKIEnrollmentService**" object for your Certification Authority, delete it.
12. Select the "**Certificate Templates**" node.
13. In the right-hand pane, delete all the Certificate Templates.
        
    **Note:** Delete all the Certificate Templates only if no other Enterprise CAs are installed in the forest. If the templates are inadvertently deleted, restore the
    templates from backup.
14. Click the "**Public key Services**" node and locate the "**NTAuthCertificates**" object.
15. If there are no other Enterprise or Stand-alone CAs installed in the forest, delete the object, otherwise leave it alone.
16. Use "**Active Directory Sites and Services**" or "**Repadmin**" command from the Windows resource kit to force replication to the other domain controllers in the domain/forest.

**Domain Controller Cleanup**  

Once the CA has been taken down, the certificates that have been issued to all the domain controllers need to be removed. This can be done easily using DSSTORE.EXE from the Resource Kit:

You can also remove old domain controller certificates by using "**certutil**" command:

1. At the command prompt on a domain controller, type: "**certutil -dcinfo deleteBad**".

2.Certutil.exe will attempt to validate all the DC certificates issued to the domain controllers. Certificates that fail to validate will be removed.
 At this point, you can reinstall Certificate Services. After the installation is finished, the new root certificate will be published to Active Directory. When the domain 
 clients refresh their security policy, they'll automatically download the new root certificate into their trusted root stores.
 To force application of the security policy.

3. At the command prompt, type "**gpupdate /target: computer**".

**Note:** If the Enterprise Windows Certificate Authority published computer/user certificate or other types of certificates (Web Server Certificate, and so on), it's recommended that you remove the old certificates before you reinstall the Enterprise Windows Certificate.

## More information

[!INCLUDE [Community Solutions Content Disclaimer](../../includes/community-solutions-content-disclaimer.md)]