---
title: "Azure Devops"
date: 2022-01-03T20:23:03+01:00
draft: false
---

$RessourcesGroup = 'azure-poc'
$VMName = 'vmone'

write-host "Start VM " $VMName " in Ressources Group " $RessourcesGroup
az vm start -g "$RessourcesGroup" -n $VmName --no-wait
write-host "VM started "

https://techgenix.com/azure-vm-storage-account-azcopy/

azcopy sync "https://[sourceaccount].file.core.windows.net/[Share]?[SAS]" "https://[targetaccount].file.core.windows.net/[Share]?[SAS]" --recursive=true