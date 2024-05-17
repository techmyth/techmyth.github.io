---
title: "VMware vSphere Native Key Provider"
date: 2021-03-16T08:39:31-04:00
author: 'Jonathan Colon Feliciano'
draft: false
tags:
  - "VMware"
  - "KMS"
  - "Encryption"

---

This is one of my favorite feature in vSphere 7 Update 2. VMware now provides the capability to use a new native key provider for encryption. Allowing us to use vSAN encryption, VM encryption and vTPM natively without the requirement to deploy a external Key provider. In the past this capability can only be provided by using a 3rd party solutions like Hytrust KeyControl. In this post i will explain how easy is to configure and deploy this awesome new feature.

Go to `[Configure > Key Providers]` to add the local key provider.

![Text](/img/2021-03-10_21-08-1024x402.webp#center)

Select `[ADD > Add Native Key Provider]`.

![Text](/img/2021-03-10_21-08-1024x402.webp#center)

Provide a Name and press `[ADD KEY PROVIDER]`.

![Text](/img/2021-03-10_21-10-1024x608.webp#center)

Backup the Master keys.

![Text](/img/2021-03-10_21-25-1024x449.webp#center)

Save the Native key Provider in a secure location. Optionally protect the key file with a strong password.

![Text](/img/2021-03-10_21-26-1024x318.webp#center)

Verify the ESXi Server Host Encryption Mode is [Enable].

![Text](/img/2021-03-10_21-27-1024x460.webp#center)

Test the configuration by encrypting an existing VM.

![Text](/img/2021-03-10_21-30-1024x478.webp#center)

Change the default "VM Storage Policy" to [VM Encryption Policy].

![Text](/img/2021-03-10_21-31-1024x630.webp#center)

Now the VM is encrypted with the Native Key Provider. `Really Awesome Feature`.

![Text](/img/2021-03-10_21-35-1024x470.webp#center)

[![ko-fi](https://ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/F1F8DEV80)
