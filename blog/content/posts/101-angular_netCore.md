---
title: "101 Angular_netCore"
date: 2021-12-06T22:32:17+01:00
draft: true
---

- install .net 5 https://dotnet.microsoft.com/download/dotnet
- install nodeJs using nvm https://bit.ly/2Hn8HjG to have several version
# chose desired node version
$version = "8.12.0"
# install nvm w/ cmder
choco install cmder
choco install nvm
refreshenv
# install node
nvm install $version
nvm use $version
//////////// For Basic use //////////// 
// check version
node -v || node --version
// list installed versions of node (via nvm)
nvm ls
// To list available remote versions on Windows 10 you can type
nvm list available
// install a specific version of node
nvm install 6.9.2
// switch version of node
nvm use 6.9.1



- install post man https://www.postman.com/ to test api 	
- install visual studio code https://code.visualstudio.com/ 
  - install extensions : c# | C# Extensions | material icon theme

- make a walking skeleton 
 dotnet watch run 

 dotnet tool install --global dotnet-ef --version 5.0.4

 dotnet ef migrations add InitialCreate -o Data/Migrations 
 dotnet ef database update


Install Angular : 

npm install -g @angular/cli
ng new client
ng serve 

<h1>{{title}} </h1> from class property


for vs code install those extensions : Bracket Pair Colorizer , Angular Language Service , Angular Snippets (Version 12)


add cors between  UseRouting , UseAuthorization

add bootstrap (https://www.techiediaries.com/angular-bootstrap-ui/)
npm install bootstrap --save
ng add ngx-bootstrap
npm install font-awesome


generate autosigned certificat : 
https://help.interfaceware.com/v6/how-to-create-self-certified-ssl-certificate-and-publicprivate-key-files#
1- install openssl (or use C:\Program Files\Git\usr\bin\openssl.exe)
2- C:\Program Files\Git\usr\bin genrsa -out privkey.pem 4096

---------------------------------
to do : install self signed certi 
---------------------------------

to migrate code c# to db : 
dotnet ef migrations add UserPasswordAdded
dotnet ef database update

