![Build](https://img.shields.io/github/workflow/status/chmouel/go-simple-uploader/Build%20and%20Test)
[![GolangCI](https://golangci.com/badges/github.com/chmouel/go-simple-uploader.svg)](https://golangci.com/r/github.com/chmouel/go-simple-uploader)
[![Codecov](https://img.shields.io/codecov/c/github/chmouel/go-simple-uploader/master.svg?style=flat-square)](https://codecov.io/gh/chmouel/go-simple-uploader) 
[![License](https://img.shields.io/github/license/chmouel/go-simple-uploader)](/LICENSE)


# GO Simple Uploader

A simple uploader in GO, meant to be deployed in a container behind a
nginx protected environment, but you can deploy it as you wish.

I mainly deploy it to OpenShift which handles for me the building of the source
the deployment with a nginx server and observability etc.. See the OpenShift
deployment section of this document.

## Install

```shell
go install github.com/chmouel/go-simple-uploader
```

### Configuration

done by environment variable with :

- **UPLOADER_HOST** -- hostname to bind to
- **UPLOADER_PORT** -- port to bind to
- **UPLOADER_DIRECTORY** -- Directory where to upload

Usually you will run this behind a HTTP server which will handle the upload
protection, acl or others. You really don't want to expose this to the internet
unprotected.

## Usage

It accepts two form field  :

- **file**: The file stream of the upload
- **path**: The path

## OpenShift Deployment

This uses S2I to generate an image against the repo and output it to an
OpenShift ImageStream and then use a Kubernetes `Deployment` to deploy it, with
a few sed for dynamic variables.

The deployment has two containers, the main one is nginx getting all requests and
passing the uploads to the uwsgi process in the other container and serves the
static file directly.

Under nginx configuration the `/private` directory is protected with the same
username password as configured in the htpasswd, which you can use to 'protect
stuff.

## Install

You need first to create a a username password with :

```
htpasswd -b -c openshift/config/osinstall.htpasswd username password
```

Then you just use the makefile target to build and deploy :

```
oc new-project uploader && make deploy
```

## Usage

Get the route of your deployment with :

```shell
oc get route uploader -o jsonpath='{.spec.host}'`
```

Test as working with :

```shell
route=http://$(oc get route uploader -o jsonpath='{.spec.host}')
echo "HELLO WORLD" > /tmp/hello.txt
curl -u username:password -F path=hello-upload.txt -X POST -F file=@/tmp/hello.txt ${route}/upload
curl ${route}/hello-upload.txt
```

## [LICENSE](LICENSE)

[Apache 2.0](LICENSE)
