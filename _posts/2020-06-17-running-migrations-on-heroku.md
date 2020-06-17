---
layout: post
title: Running Alpas app Migrations on a free tier Heroku Server
author: Gideon Brimleaf
postHero: /assets/images/alpas.png
description: Steps to run alpas database migrations on Heroku free tier
---

So you have built an Alpas app and [followed these steps to deploy to Heroku](https://gideonbrimleaf.github.io/2020/06/15/deploying-alpas-app-to-heroku.html) but you are having issues running migrations. These additional steps should help you run the migrations as well as any other Alpas task you need on Heroku, especially the free tier.

### Initial Fixes

When running <span class="code-snippet">heroku run ./alpas db:migrate</span> you may find that the migration exits too early because the dyno capacity on the free tier has been maxed out. If this happens try making a trivial change to your project to force a new deploy (with the Heroku web process set to 0), navigate to <span class="code-snippet">App > More > Restart All Dynos</span> to reset the box.  Then try to run the migration command.

Alternatively try <span class="code-snippet">App > More > Restart All Dynos</span> followed by <span class="code-snippet">heroku ps:scale web=0</span> to ensure that the app is not running when trying to run a migration.

### Migrating with the Compiled Project

If the problem persists, you will need to run the <span class="code-snippet">db:migrate</span> command on the compiled jar file.  The normal <span class="code-snippet">alpas</span> script recompiles the entire project before running the migration which likely requires too much RAM for the lower/free tier dynos on Heroku.

1. Create a new file called <span class="code-snippet">alpas_prod.sh</span> in the root of your project. This can be called anything but if you name it differently just apply that consistently to the next steps.
2. Copy and paste in the contents of [this sample file](https://gist.github.com/GideonBrimleaf/fb57c60f5b10c547d0f88468d4aaa9ad) into your <span class="code-snippet">alpas_prod.sh</span> file.  This is very similar to the original <span class="code-snippet">alpas</span> script but runs against the already compiled jar file rather than using gradle commands which recompile the project. Be sure to rename the jar file reference in the script to your project's jar file after copying over. 
3. As per the original <span class="code-snippet">alpas</span> script, make sure this new file has executable rights with <span class="code-snippet">chmod +x ./alpas_prod.sh</span>
4. Commit your changes then step through the migration process in the [previous post](https://gideonbrimleaf.github.io/2020/06/15/deploying-alpas-app-to-heroku.html), substituting <span class="code-snippet">heroku run ./alpas_prod.sh db:migrate</span> in for the migration command.

You should see now that the migration happens without the need to recompile and successfully executes!