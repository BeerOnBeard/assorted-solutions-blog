---
title: Running Golang Tests in Alpine Docker Container
date: 2019-10-11 13:16:36
---


As of Go 1.10, `vet` is run when `go test` is run. Unfortunately, `vet` requires `gcc` which is not installed in Alpine docker images. In order to run tests using the Alpine linux base image, `golang:1.13-alpine`, the developer must explicitly turn `vet` off when running tests.

Change the following line

```dockerfile
RUN go test -v ./...
```

to

```dockerfile
RUN go test -v -vet=off ./...
```

to disable `vet` when running `go test`.

Another option would be to use the Debian-based image as an initial stage to run the tests with `vet` enabled and then use a second stage based on the Alpine image to reduce the image size.
