# ISLE Installation Option Comparison

## Make Local
* Creates a development environment
* Codebase folder is bind-mounted
* Uses the Islandora Sandbox repo to set up the Drupal site
* Using git clone to create codebase folder

## Make Demo
* Runs make local, then also populates the site with some demo content

## Make Starter
* Creates a development environment
* Codebase folder is bind-mounted
* Uses the Islandora Starter Site repo to set up the Drupal site
* Using composer create project to create codebase folder
* Can also be used with a custom codebase folder to spin up a production site from your already existing site instead of the starter site
* If you place your codebase folder in the isle-dc directory it will use that, otherwise it downloads the starter site

## Make Starter-dev
* Creates a development environment
* Codebase folder is bind-mounted
* Uses the Islandora starter site repo to set up the drupal site
* Using git clone to create codebase folder

##  Make Production
* Creates a production environment
* Codebase folder is NOT bind-mounted
* Uses a custom Drupal image to set up the drupal site
* Using composer install on the composer.json file that is included in the custom Drupal container

make local and make starter-dev both leave you with a git repo in your codebase folder, so you could easily contribute back to the sandbox or starter site. Running make starter uses composer instead so the codebase folder is not set up as a git repo. I believe the only reason to use starter-dev is if you are contributing to the starter site. Otherwise make starter will create a new site or spin up an existing one for you.
