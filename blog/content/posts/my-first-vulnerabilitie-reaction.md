---
title: "My First Vulnerabilitie Reaction"
date: 2021-12-13T10:32:27+01:00
draft: false
---

Today I have ti fix a vulnerability in our project, you already heard about Cve-2021-44228 ?
https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-44228
https://www.cert.ssi.gouv.fr/alerte/CERTFR-2021-ALE-022/
https://www.nolimitsecu.fr/log4shell/


0- understand the vulnerability.
1- list projects by scaning code source.
2- list dependencies using :
    for .net : 'Get-Package | Format-Table -AutoSize'
    for android : 'adb shell pm list packages' 
3- and yes , 0 reference detected
