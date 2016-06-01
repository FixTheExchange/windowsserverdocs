---
title: Planning Server Isolation Zones
ms.custom: na
ms.prod: windows-server-2012
ms.reviewer: na
ms.suite: na
ms.technology: 
  - techgroup-networking
ms.tgt_pltfrm: na
ms.topic: article
ms.assetid: 7290f8b4-508d-44e3-96d8-51d798e33647
---
# Planning Server Isolation Zones
Sometimes a server hosts data that is sensitive. If your servers host data that must not be compromised, you have several options to help protect that data. One was already addressed: adding the server to the encryption zone. Membership in that zone prevents the server from being accessed by any computers that are outside the isolated domain, and encrypts all network connections to server.

The second option is to additionally restrict access to the server, not just to members of the isolated domain, but to only those users or computers who have business reasons to access the resources on the server. You can specify only approved users, or you can additionally specify that the approved users can only access the server from approved computers.

To grant access, you add the approved user and computer accounts to network access groups \(NAGs\) that are referenced in a firewall rule on this server. When the user sends a request to the server, the standard domain isolation rules are invoked. This causes IKE to use Kerberos V5 to exchange credentials with the server. The additional firewall rule on the server causes Windows to check the provided computer and user accounts for group membership in the NAGs. If either the user or computer is not a member of a required NAG then the network connection is refused.

## Isolated domains and isolated servers
If you are using an isolated domain, the client computers already have the IPsec rules to enable them to authenticate traffic when the server requires it. If you add an isolated server, it must have a GPO applied to its group with the appropriate connection security and firewall rules. The rules enforce authentication and restrict access to only connections that are authenticated as coming from an authorized computer or user.

If you are not using an isolated domain, but still want to isolate a server that uses IPsec, you must configure the client computers that you want to access the server to use the appropriate IPsec rules. If the client computers are members of an Active Directory domain, you can still use Group Policy to configure the clients. Instead of applying the GPO to the whole domain, you apply the GPO to only members of the NAG.

## Creating multiple isolated server zones
Each set of servers that must be accessed by different sets of users should be set up in its own isolated server zone. After one set of GPOs for one isolated server zone has been successfully created and verified, you can copy the GPOs to a new set. You must change the GPO names to reflect the new zone, the name and membership of the isolated server zone group to which the GPOs are applied, and the names and membership of the NAG groups that determine which clients can access the servers in the isolated server zone.

## Creating the GPOs
Creation of the groups and how to link them to the GPOs that apply the rules to members of the groups are discussed in the [Planning Group Policy Deployment for Your Isolation Zones](Planning-Group-Policy-Deployment-for-Your-Isolation-Zones.md) section.

An isolated server is often a member of the encryption zone. Therefore, copying that GPO set serves as a good starting point. You then modify the rules to additionally restrict access to only NAG members.

### GPO settings for isolated servers running [!INCLUDE[win8_server_2](includes/win8_server_2_md.md)], [!INCLUDE[nextref_server_7](includes/nextref_server_7_md.md)] or [!INCLUDE[nextref_longhorn](includes/nextref_longhorn_md.md)]
GPOs for computers running [!INCLUDE[win8_server_2](includes/win8_server_2_md.md)], [!INCLUDE[nextref_server_7](includes/nextref_server_7_md.md)] or [!INCLUDE[nextref_longhorn](includes/nextref_longhorn_md.md)] should include the following:

> [!NOTE]
> The connection security rules described here are identical to the ones for the encryption zone. If you do not want to encrypt access and also restrict access to NAG members, you can use connection security rules identical to the main isolated domain. You must still add the firewall rule described at the end of this list to change it into an isolated server zone.

-   IPsec default settings that specify the following options:

    1.  Exempt all ICMP traffic from IPsec.

    2.  Key exchange \(main mode\) security methods and algorithm. We recommend that you do not include Diffie\-Hellman Group 1, DES, or MD5 in any setting. They are included only for compatibility with previous versions of Windows. Use the strongest algorithm combinations that are common to all your supported operating systems.

    3.  Data protection \(quick mode\) algorithm combinations. Check **Require encryption for all connection security rules that use these settings**, and then specify one or more integrity and encryption combinations. We recommend that you do not include DES or MD5 in any setting. They are included only for compatibility with previous versions of Windows. Use the strongest algorithm combinations that are common to all your supported operating systems.

        If any NAT devices are present on your networks, do not use AH because it cannot traverse NAT devices. If isolated servers must communicate with hosts in the encryption zone, include an algorithm that is compatible with the requirements of the encryption zone GPOs.

    4.  Authentication methods. Include at least computer\-based Kerberos V5 authentication for compatibility with the rest of the isolated domain. If you want to restrict access to specific user accounts, also include user\-based Kerberos V5 authentication as an optional authentication method. Do not make the user\-based authentication method mandatory, or else computers that cannot use AuthIP instead of IKE, including Windows XP and Windows Server 2003, cannot communicate. Likewise, if any of your domain isolation members cannot use Kerberos V5, include certificate\-based authentication as an optional authentication method.

-   The following connection security and firewall rules:

    -   A connection security rule that exempts all computers on the exemption list from authentication. Be sure to include all your Active Directory domain controllers on this list. Enter subnet addresses, if applicable in your environment.

    -   A connection security rule, from **Any IP address** to **Any IP address**, that requires inbound and requests outbound authentication by using Kerberos V5 authentication.

        > [!IMPORTANT]
        > Be sure to begin operations by using request in and request out behavior until you are sure that all the computers in your IPsec environment are communicating successfully by using IPsec. After confirming that IPsec is operating as expected, you can change the GPO to require in, request out.

    -   A firewall rule that specifies **Allow only secure connections**, **Require encryption**, and on the **Users and Computers** tab includes references to both computer and user network access groups.

-   A registry policy that includes the following values:

    1.  Enable PMTU discovery. Enabling this setting allows TCP\/IP to dynamically determine the largest packet size supported across a connection. The value is found at HKLM\\System\\CurrentControlSet\\Services\\TCPIP\\Parameters\\EnablePMTUDiscovery \(dword\). The sample GPO preferences XML file in [Appendix A: Sample GPO Template Files for Settings Used in this Guide](Appendix-A--Sample-GPO-Template-Files-for-Settings-Used-in-this-Guide.md) sets the value to **1**.

    > [!NOTE]
    > For a sample template for these registry settings, see [Appendix A: Sample GPO Template Files for Settings Used in this Guide](Appendix-A--Sample-GPO-Template-Files-for-Settings-Used-in-this-Guide.md).

**Next:**[Planning Certificate-based Authentication](Planning-Certificate-based-Authentication.md)

