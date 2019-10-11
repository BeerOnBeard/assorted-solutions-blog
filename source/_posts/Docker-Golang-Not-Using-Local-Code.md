---
title: Docker Golang Not Using Local Code
date: 2019-10-11 13:08:01
---


I just started building an app in Golang that scrapes some mountain bike outlet sites and sends me an email with any updates. Since my home server runs everything in Docker, I thought I would wrap the new app in a Docker container to make deployment easy.

My dockerfile looked like this (with unrelated lines removed)

```dockerfile
FROM golang:1.13-alpine

WORKDIR /go/src/go-watch-that-site
COPY . .

RUN go get -v ./...
RUN go install -v ./...

CMD [ "go-watch-that-site" ]
```

After making some changes and trying to run it again, I found that my changes were not reflected when I ran them in the container. After a while scratching my head, I found out that go was installing the code from my GitHub remote repository and completely ignoring my local code. It turns out that I didn't set my `WORKDIR` correctly. I needed to set the path to the fully-qualified path in order for the system to function correctly.

I changed

```dockerfile
WORKDIR /go/src/go-watch-that-site
```

to

```dockerfile
WORKDIR /go/src/github.com/beeronbeard/go-watch-that-site
```

and my local changes were finally reflected in the running container.

Hooray!
