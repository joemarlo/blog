# Source code for my blog @ joemarlo.com

To build site make sure joemarlo.github.io repo is public

Execute deploy.sh via terminal: ./deploy.sh. This builds the site to the "public" folder by:
  1. Deletes all content in the public folder
  2. Builds site
  3. Places that content in the joemarlo.github.io repo

The site can be run locally via shell:
 - `hugo server`  
 - `hugo server -D` to include all the draft posts

