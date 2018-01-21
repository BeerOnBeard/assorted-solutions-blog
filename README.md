[![Travis](https://img.shields.io/travis/BeerOnBeard/assorted-solutions-blog.svg?style=flat-square)](https://travis-ci.org/BeerOnBeard/assorted-solutions-blog)
[![Docker Build Status](https://img.shields.io/docker/build/beeronbeard/assorted-solutions-blog.svg?style=flat-square)](https://hub.docker.com/r/beeronbeard/assorted-solutions-blog/)

# The Site
The site is built on top of [Hexo](https://hexo.io). I created a custom theme, so-fresh. The theme does not comply with the current standards for a stand-alone Hexo theme, but it works for this site. It's a very basic blog platform for me to write on.

# The System as a Whole
The master branch holds the code for the site and configuration files for TravisCI and Docker. The deploy branch holds the generated static site and the Dockerfile. TravisCI is used to build the site every time a commit is made to master. If the generated site or the Dockerfile changes, TravisCI will push the updates to the deploy branch. If there are no changes, no code will be committed to the deploy branch. This prevents superfluous releases from being created.

DockerHub is set up to watch the deploy branch. When a commit is made to the deploy branch, DockerHub will create a new container based on the NGINX image with the latest generated site. The container will be tagged as latest.

# NPM Commands

There are a few NPM commands for local development and the build system. These commands expose functionality provided by Hexo.io without the developer needing to install Hexo.io globally.

## npm run new [layout] "Post Name"

A helper method for adding a new article with the specified layout. For example, running the following command will create a new post "New Post":

```bash
npm run new post "New Post"
```

Running this next command will create a draft called "New Draft":

```bash
npm run new draft "New Draft"
```

## npm run pub-draft "Draft Name"

A helper method for publishing an existing draft as a post. The draft "New Draft" must already exist. It will be moved into the `_posts` folder.

## npm run dev

Generate the site and start the built-in Hexo.io development server locally.

## npm run build

Generate the site.
