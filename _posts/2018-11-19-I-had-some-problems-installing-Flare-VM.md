---
layout: post
title:  "I had some problems installing Flare-VM"
date:   2018-11-19 1:28:26
author: Priyansh Singh
description: "The university uses a VPS which creates an issue following the Flare-VM installation ..."
keywords: ""
---

I recently saw a tweet from FireEye that they have updated the flare VM system and now it comes with a method for installing and uninstalling reverse engineering tools and many updates to the old tools along with numerous new ones, especially for malicious document analysis. More information about the update can be found [here](https://www.fireeye.com/blog/threat-research/2018/11/flare-vm-update.html).

I have tried installing the VM once before but quickly gave up as my university proxy was constantly swatting all the download requests down from the chocolatey and BoxStarter interfaces. Seeing [this blog post](https://www.fireeye.com/blog/threat-research/2018/11/flare-vm-update.html), I finally decided I should give this a chance and try and figure out why my machine is not responding the way I would like it to.

The problem lies in at the proxy server level. The way our sysadmin has set up the proxy, for every request or connection, there is a need for authentication form the service. Weirdly enough setting up or trying to update the connection setting already loaded in the Powershell does not work OR at least I could not find a way for it to update and reflect my credentials.

**Sidebar:** I did not realise until recently Microsoft provides Windows licenses for 90 days at no cost for testing the IE and Edge I found some references to this while I was looking for some guide of setting up my lab. You can revert the checkpoint before you registered the copy to get another 90 days. If someone knows a similar trick for MS Office, please do tell me.

Back to the topic, so after a day of googling and testing I gave up and decided, I might have to some work around this bull, and I decided to dig into PowerShell scripting. I cloned the flare VM files and made some changes to the install.ps1 file you can find these on [my GitHub](https://github.com/priyanshs/flare-proxy). Essentially I added the following code:

    $proxy = new-object System.Net.WebProxy("http://Your_ProxyServer:Your_proxyport")
    $Username = "Your_username"
    $Password = ConvertTo-SecureString "Your_password" -AsPlainText -Force
    $cred = New-Object System.Management.Automation.PSCredential $Username, $Password
    $proxy.credentials = $cred
    $WebClient = new-object System.Net.WebClient
    $WebClient.proxy = $proxy
    

    Since I failed to update the credentials for my PowerShell, I needed to make sure I could do this for every System.WebClient API calls the shell scripting makes. I have defined a few global environment variables which feed to the WebClient calls every time. Flare setup uses BoxStarter services and Chocolatey. I sincerely suggest to [look up this guide](https://github.com/chocolatey/choco/wiki/Proxy-Settings-for-Chocolatey) on Chocolatey proxy setup and define global variables in your shell before execution using the following. If you fail to install chocolatey using this method, you could try installing it offline setting up the proxy and then run the BoxStarter script.

    $env:chocolateyProxyLocation = 'https://local/proxy/server'
    $env:chocolateyProxyUser = 'username'
    $env:chocolateyProxyPassword = 'password'
    