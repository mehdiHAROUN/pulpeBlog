---
title: "Import Huge Db Sqlserver | SqlPackage.exe"
date: 2022-04-08T17:10:26+02:00
draft: false
---

- dacpac : schema only 
- bacpac : data + schema 

## méthode 1 utilisant SqlPackage.exe

- installer sqlpackage.exe (https://docs.microsoft.com/en-us/sql/tools/sqlpackage/sqlpackage-download?view=sql-server-ver15)
- créer une base de données vide avec une taille initiale qui correspond à la taille de la base de données à importer.
- lancer ce scrpit dans un terminal powershell en utilisant 

```powershell
[string]$myBacpac = 'C:\Users\haroun\Downloads\file.bacpac'
[string]$connectionString = 'Data Source=CGDLPT\SQL2019;Initial Catalog=testExp; Integrated Security=true;'
[string]$action = 'Import'

[string[]]$commandParameters = @(
    "/Action:`"$action`""
    "/SourceFile:`"$myBacpac`""
    "/TargetConnectionString:`"$connectionString`""
)

[string]$LatestSqlPackage = Get-Item 'C:\*\Microsoft SQL Server\*\DAC\bin\sqlpackage.exe' | %{get-command $_}| sort version -Descending | select -ExpandProperty source -First 1

if ($LatestSqlPackage) {
    Write-Verbose "Found: $LatestSqlPackage"
    & $LatestSqlPackage $commandParameters
} else {
      Write-Error "Could not find SqlPackage.exe"
}
``` 

## méthode 2 en supprimant les tables lourdes

1 - renommer le bacpac en bacpac.zip
2 - explorer les dossiers et supprimer les tables lourdes
3 - recalculer checksum du bacpac utilisant un outil de calcul de checksum (https://github.com/gertd/dac/tree/master/drop/debug) pour plus d'info (http://mathew.weaverz.net/2019/05/manually-modifying-sql-server-bacpac.html)