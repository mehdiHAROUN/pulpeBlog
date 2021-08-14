---
title: "Hugo SS"
date: 2021-08-14T00:19:14+02:00
draft: false
---


#### 1 - install hugo :
 fellow the official doc : https://gohugo.io/getting-started/installing/
if you are a windows user : 
  * downlad the last relase (hugo_XXX_Windows-32bit.zip) from <https://github.com/gohugoio/hugo/eleases>
  * unzip the folder for exemple (C:\hugo).
  * add C:\hugo as variable.
  * check if you did it well , open cmd and launch 'hugo version' to get the intalled version of hugo.
#### 2 - create new project (website)
  * from cmd : hugo new site blog.
  * change them using git clone <https://github.com/olOwOlo/hugo-theme-even> themes/even , this command will copy folder even in thems directory git submodule add https://github.com/koirand/pulp.git themes/pulp
  * update config.toml with the parameters that depends from each them  
  * from cmd : hugo server to start the website, test it from <http://localhost:1313/>
  * make all modification that you want. For
 
 #### 3 - push application to github : 
  * change directory to blog and init git project : git init
  * Commit the changes git add -A  git commit -m "initial commit"
  * Create a blank GitHub repo (don't create a README) from <https://github.com/new>
  * git remote add origin <https://github.com/mehdyHD/blog>



