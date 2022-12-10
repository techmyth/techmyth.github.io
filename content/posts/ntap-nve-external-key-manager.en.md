---
title: "NetApp Volume Encryption Setup with External Key Manager"
date: 2021-05-16T08:39:31-04:00
author: 'Jonathan Colon Feliciano'
draft: false
tags:
  - "NetApp"
  - "KMS"
  - "Encryption"
  - "External Manager"
---

Using NetApp documentation as a reference:

> NetApp Volume Encryption (NVE) is a software-based technology for encrypting data at rest one volume at a time. An encryption key that can only be accessed by the storage system ensures that the data on the volume cannot be read if the underlying device is reused, returned, lost or stolen.

##### [NetApp Documentation](https://docs.netapp.com/ontap-9/topic/com.netapp.doc.pow-nve/GUID-EAD13D8E-0219-45B6-A2C6-B25B76C9CA1A.html)

In this tutorial I explain how easy it is to configure and manage this impressive security feature.

Before starting to configure this feature it is necessary to have an existing “Key Management Service”. For the purpose of this tutorial I will use the “HyTrust KeyControl” KMS which I previously explained in the blog. If you want to know more about it, read the following [post](http://192.168.7.40/2021/05/16/hytrust-keycontrol-key-management-server-setup/).

**Step 1:** Create a client certificate for authentication purposes:

Go to **[KMIP > Client Certificate]** and select Create Certificate in the Actions menu.

![Text](/img/2021-05-16_15-25-1024x563.webp#center)

Select a name for the certificate and press **Create**.

![Text](/img/2021-05-16_15-27-1024x565.webp#center)

Once the certificate has been created, it should be stored in a safe place.

![Text](/img/2021-05-16_15-28-1024x737.webp#center)

The downloaded file contains the **Client** and **Root CA** certificate needed to configure the KMS option in Ontap.

![Text](/img/2021-05-16_15-31-1024x493.webp#center)

**Step 2:** Validation of the Ontap cluster

The important things to consider before you can configure this security feature are the cluster status and the necessary license to support the encryption feature.

The **cluster show** command displays the overall status of the Ontap cluster.

```bash
OnPrem-HQ::> cluster show 
Node                  Health  Eligibility
--------------------- ------- ------------
OnPrem-HQ-01         true    true
OnPrem-HQ-02         true    true
2 entries were displayed.
```

The **system node show** command displays the node health and the system model.

```bash
OnPrem-HQ::> system node show
Node      Health Eligibility Uptime        Model       Owner    Location  
--------- ------ ----------- ------------- ----------- -------- ---------------
OnPrem-HQ-01 true true           00:18:10 SIMBOX
OnPrem-HQ-02 true true           00:18:08 SIMBOX
2 entries were displayed.
```
With the **system license show** command you can validate the license installed on the cluster. Here you can see that the volume encryption license is installed on both nodes.

```bash
OnPrem-HQ::> system license show -package VE

Serial Number: X-XX-XXXXXXXXXXXXXXXXXXXXXXXXXX
Owner: OnPrem-HQ-01
Installed License: Legacy Key
Capacity: -
Package           Type     Description           Expiration
----------------- -------- --------------------- -------------------
VE                license  Volume Encryption License 
                                                 -

Serial Number: X-XX-XXXXXXXXXXXXXXXXXXXXXXXXXX
Owner: OnPrem-HQ-02
Installed License: Legacy Key
Capacity: -
Package           Type     Description           Expiration
----------------- -------- --------------------- -------------------
VE                license  Volume Encryption License 
                                                 -
2 entries were displayed.

OnPrem-HQ::> 
```

**Step 3:** Configuration of the KMS certificate in Ontap:

#### As stated in the NetApp documentation:

> The cluster and the KMIP server use KMIP SSL certificates to verify each other’s identity and establish an SSL connection. Before setting up the SSL connection to the KMIP server, you must install the KMIP client SSL certificates for the cluster and the public SSL certificate for the KMIP server’s root Certificate Authority (CA).

##### [NetApp KMIP Documentation](https://docs.netapp.com/ontap-9/topic/com.netapp.doc.pow-nve/GUID-D1593ED6-AAF8-4DEE-A2A7-6AEF239C6874.html)

To install the KMIP client certificate on the NetApp cluster, run the following commands:

##### Note: The contents of the certificates are extracted from the previously downloaded files. The ONTAPEncryption.pem and cacert.pem files contain the necessary information.

```text
OnPrem-HQ::> security certificate install –vserver OnPrem-HQ -type client –subtype kmip-cert
Please enter Certificate: Press <Enter> when done
-----BEGIN CERTIFICATE----- 
Certificate Content
-----END CERTIFICATE-----
Please enter Private Key: Press <Enter> when done
-----BEGIN PRIVATE KEY-----  
Certificate Private Key Content
-----END PRIVATE KEY-----
You should keep a copy of the private key and the CA-signed digital certificate for future
reference.

The installed certificate's CA and serial number for reference:
CA: HyTrust KeyControl Certificate Authority
serial: C9A148B9

The certificate's generated name for reference: ONTAPEncryption
```

```text
OnPrem-HQ::> security certificate install -vserver OnPrem-HQ -type server-ca -subtype kmip-cert 

Please enter Certificate: Press <Enter> when done
-----BEGIN CERTIFICATE-----  
Certificate Content
-----END CERTIFICATE-----
You should keep a copy of the CA-signed digital certificate for future reference.

The installed certificate's CA and serial number for reference:
CA: HyTrust KeyControl Certificate Authority
serial: 60A148B6

The certificate's generated name for reference: HyTrustKeyControlCertificateAuthority
```

**Step 4:** Configure the NetApp Volume Encryption solution:

For this tutorial it is necessary to configure an external “KMS” key management server so that the storage system can securely store and retrieve authentication keys for the NetApp Volume Encryption (NVE) solution.

##### Note: NetApp recommends a minimum of two server for redundancy and disaster recovery.

The following command allows you to add the KMS server to the Ontap system using the IP address 192.168.7.201, port TCP/5696 and using the previously configured certificates.

```text
OnPrem-HQ::> security key-manager external enable -vserver OnPrem-HQ -key-servers 192.168.7.201:5696 -client-cert ONTAPEncryption -server-ca-certs HyTrustKeyControlCertificateAuthority 

OnPrem-HQ::> security key-manager external show                                                                                                                                           

                  Vserver: OnPrem-HQ
       Client Certificate: ONTAPEncryption
   Server CA Certificates: HyTrustKeyControlCertificateAuthority
          Security Policy: -

Key Server
------------------------------------------------
192.168.7.201:5696
```

It is important to validate that the KMS service is **“available”** before proceeding to create encrypted volumes. The **security key-manager external show-status** command does allow you to validate the status of the service.

```text
OnPrem-HQ::> security key-manager external show-status

Node  Vserver  Key Server                                   Status
----  -------  -------------------------------------------  ---------------
OnPrem-HQ-01
      OnPrem-HQ
               192.168.7.201:5696                           available
OnPrem-HQ-02
      OnPrem-HQ
               192.168.7.201:5696                           available
2 entries were displayed.

OnPrem-HQ::> 
```

**Step 5:** Create an encrypted volume (NVE)

In this part of the tutorial we must validate that the configuration is correct by creating an encrypted volume inside Ontap. For this step I will use the vol create command using the encrypt true option.

```text
OnPrem-HQ::> vol create TEST_Encryption -vserver SAN -size 10G -aggregate OnPrem_HQ_01_SSD_1 -encrypt true 
  
[Job 763] Job succeeded: Successful 
```

With the **vol show** command i can verify that the volume has been created with the encryption option.

```text
OnPrem-HQ::> vol show -encryption -vserver SAN -encryption-state full 
Vserver   Volume       Aggregate    State      Encryption State
--------- ------------ ------------ ---------- ----------------
SAN       TEST_Encryption OnPrem_HQ_01_SSD_1 online full

OnPrem-HQ::> 
```

**Step 6:** Validate the Encryption information on the KMS server.

In the last step we must enter the administration portal of the **“HyTrust KeyControl”** application to validate that the encryption keys are stored in the platform. To validate this information go to the menu **[KMIP > Objects]** where you can validate that the keys were created after the creation of the volume within Ontap.

![Text](/img/HytrustDashBoard.webp#center)

### Summary

In this tutorial I show you how to configure the KMS service within Ontap that allows us to create encrypted volumes. This simple solution allows us to increase or improve the security posture in our organization.

### Hasta Luego Amigos!
