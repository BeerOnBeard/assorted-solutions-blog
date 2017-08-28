[![Build Status](https://travis-ci.org/BeerOnBeard/assorted-solutions-blog.svg?branch=master)](https://travis-ci.org/BeerOnBeard/assorted-solutions-blog)

# The Site
The site is built on top of [Hexo](https://hexo.io). I created a custom theme, so-fresh. The theme does not comply with the current standards for a stand-alone Hexo theme, but it works for this site. It's a very basic blog platform for me to write on.

# The System as a Whole
The master branch holds the code for the site and configuration files for TravisCI and Docker. The deploy branch holds the generated static site and the Dockerfile. TravisCI is used to build the site every time a commit is made to master. If the generated site or the Dockerfile changes, TravisCI will push the updates to the deploy branch. If there are no changes, no code will be committed to the deploy branch. This prevents superfluous releases from being created.

DockerHub is set up to watch the deploy branch. When a commit is made to the deploy branch, DockerHub will create a new container based on the NGINX image with the latest generated site. The container will be tagged as latest.
