---
title: "VMware vSphere Native Key Provider"
date: 2021-03-23T08:39:31-04:00
draft: false
tags:
  - "KMS"
  - "Encryption"

---

HyTrust KeyControl enables encryption users to easily manage their encryption keys at scale. HyTrust is the only KMS vendor that VMware invested in. It is available as an OVA, for fast installation and configuration in VMware vCenter. In this post i show you how to easily install and configure this KMS service in a vSphere environment.

**Step 1** – Deploying the OVA Package

![Text](/img/2021-03-13_15-51-1024x727.webp#center)

Browse to the location where the OVA file located.

![Text](/img/2021-03-13_15-52-1024x445.webp#center)

Type the name for the new VM.

![Text](/img/2021-03-13_15-53-1024x574.webp#center)

Select where to run the new VM.

![Text](/img/2021-03-13_15-54-1024x582.webp#center)

Review the hardware requirement.

![Text](/img/2021-03-13_15-55-1024x576.webp#center)

Accept the license agreement

![Text](/img/2021-03-13_15-56-1024x578.webp#center)

Select the VM configuration. In my case because is a lab setup the demo configuration is selected.

#### Note: Not recommended for production environment

![Text](/img/2021-03-13_15-56_1-1024x575.webp#center)

Select the Storage and Network where the new VM will be running.

![Text](/img/2021-03-13_15-57-1024x575.webp#center)

Specify the VM Network parameters.

![Text](/img/2021-03-13_16-01-1024x575.webp#center)

**Step 2** – Configuring the newly deployed KMS appliance.

Power on the newly deployed VM server. It will ask you to specify a password for the htadmin account. Enter a new password for **htadmin** and press **OK**.

![Text](/img/2021-03-13_16-17-1024x757.webp#center)

Wait for the configuration process to complete

![Text](/img/2021-03-13_16-17_1-1024x764.webp#center)

Go to the KMS management console by acceding https://**kms-ip-address** then provide the default credentials.

![Text](/img/2021-03-13_16-35.webp#center)

Complete the configuration wizard by selecting the instance type and specify a new password.

![Text](/img/2021-03-13_16-37_1-1024x281.webp#center)

Optional - Configure Email notification.

![Text](/img/2021-03-13_16-39-1-1024x982.webp#center)

Download and save the Admin Key to a secure location.

#### Note: This key is primarily used for recovery purpose

**Step 3** – Enable the KMIP Service

![Text](/img/2021-03-13_16-39_1-1024x379.webp#center)

The Key Management Interoperability Protocol (KMIP) enables communication between key management systems and cryptographically-enabled applications, including email, databases, and storage devices. Select KMIP in the top banner bar. Go to State and put it on **Enabled**. Then open Protocol and select **Version 1.1** from the drop-down list. As a final step go to **Restrict TLS** and select **Enabled** to make sure traffic is on the TLS 1.2 protocol. Click the **Apply** button now to apply the new settings.

![Text](/img/2021-03-13_20-34-1024x403.webp#center)

### Summary

We have now added and configure the KMS server which gives us some extra security possibilities for our infrastructure or cryptographically-enabled applications.

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
