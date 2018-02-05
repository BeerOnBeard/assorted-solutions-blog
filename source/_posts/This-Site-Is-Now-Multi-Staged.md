---
title: This Site Is Now Multi-Staged
subtitle: Goodbye, TravisCI. Thank you for being awesome.
date: 2018-02-04 18:59:45
---

I just pulled the trigger on migrating this site to use a multi-stage Docker build instead of TravisCI pushing into a deploy branch that then triggered a build on Docker Hub. It feels awesome.

Thank you, [TravisCI](https://travis-ci.org/), for providing an awesome and easy to use build service. Although my CI/CD pipeline is one step easier, it was nice of you to be there when I needed a helping hand.

Multi-stage builds for static sites like this are super simple. To keep everything light, I used the Alpine-based versions of Node Carbon and Nginx. It was as simple as copying my files into the container, running `npm install`, `npm build`, and shuttling all the files over to an Nginx container that exposes port 80. Technology is amazing. Here's the file.

```dockerfile
FROM node:carbon-alpine as generator

WORKDIR /usr/generator

COPY . .

RUN npm install && \
    npm run build

###################################
# Host image
###################################
FROM nginx:alpine

ARG BUILD_DATE
ARG VCS_REF

LABEL org.label-schema.build-date=$BUILD_DATE \
      org.label-schema.vcs-url="https://github.com/BeerOnBeard/assorted-solutions-blog.git" \
      org.label-schema.vcs-ref=$VCS_REF \
      org.label-schema.schema-version="1.0.0-rc1"

COPY --from=generator /usr/generator/public /usr/share/nginx/html
```

I also set up a `.dockerignore` file so I don't ship as much stuff when building the container.

```file
.git
.gitignore
db.json
hooks
node_modules
npm-debug.log
README.md
```

Now my build environment is the same on my local machine and my production build system!

You also might have noticed the extra `ARG`s I'm passing in and using for labels. `BUILD_DATE` and `VCF_REF` are passed in by Docker Hub thanks to a build hook I set up in the project. The details are outlined in the [Docker Docs page on Advanced options for Autobuild and Autotest](https://docs.docker.com/docker-cloud/builds/advanced/). Basically, you're allowed to override the build phase to pass in some custom variables. Here's my `build` file:

```bash
#!/bin/bash

docker build \
  --build-arg VCS_REF=`git rev-parse --short HEAD` \
  --build-arg BUILD_DATE=`date -u +”%Y-%m-%dT%H:%M:%SZ”` \
  -t $IMAGE_NAME .
```
