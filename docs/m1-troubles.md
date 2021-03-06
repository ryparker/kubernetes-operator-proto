# M1 Troubles

Issues I ran into while setting up on an M1 CPU:

When running

```sh
make docker-build docker-push IMG="example.com/memcached-operator:v0.0.1"
```

throws

```sh
unable to start control plane itself: failed to start the controlplane. retried 5 times: exec: \"etcd\": executable file not found in $PATH
```

[Github issue](https://github.com/operator-framework/operator-sdk/issues/5090)

[fix](https://github.com/kubernetes-sigs/controller-runtime/issues/1657#issuecomment-988484517)

Just modify Makefile's `test` target generated by kubebuilder:
```sh
test: manifests generate fmt vet envtest ## Run tests.
-	KUBEBUILDER_ASSETS="$(shell $(ENVTEST) use $(ENVTEST_K8S_VERSION) -p path)" go test ./... -coverprofile cover.out
+	KUBEBUILDER_ASSETS="$(shell $(ENVTEST) --arch=amd64 use $(ENVTEST_K8S_VERSION) -p path)" go test ./... -coverprofile cover.out
```


This will get you further however docker will eventually throw

```sh
798afb9dcee7: Preparing
error parsing HTTP 404 response body: invalid character '<' looking for beginning of value: "<?xml version=\"1.0\" encoding=\"iso-8859-1\"?>\n<!DOCTYPE html PUBLIC \"-//W3C//DTD XHTML 1.0 Transitional//EN\"\n         \"http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd\">\n<html xmlns=\"http://www.w3.org/1999/xhtml\" xml:lang=\"en\" lang=\"en\">\n\t<head>\n\t\t<title>404 - Not Found</title>\n\t</head>\n\t<body>\n\t\t<h1>404 - Not Found</h1>\n\t\t<script type=\"text/javascript\" src=\"//wpc.75674.betacdn.net/0075674/www/ec_tpm_bcon.js\"></script>\n\t</body>\n</html>\n"
make: *** [docker-push] Error 1
```


This is an error from `docker push` , they don't tell you this very explicitly in the docs but you have to update the command to use a real image registry. For example my docker hub username is `ryparker`:

```sh
make docker-build docker-push IMG="ryparker/memcached-operator:v0.0.1"
```


now that we're past that we run
```sh
make bundle IMG="ryparker/memcached-operator:v0.0.1"
make bundle-build bundle-push BUNDLE_IMG="ryparker/memcached-operator-bundle:v0.0.1"
```

Then we get another error:

```sh
Version v3.8.7 does not exist.
```

[github issue](https://github.com/operator-framework/operator-sdk/issues/5785)

Fix this by modifying the Makerfile with

```sh
KUSTOMIZE_VERSION ?= v4.4.0
```

Boom! We're moving on

it's going to ask a bunch of questions here so get creative. i.g.

```sh
Display name for the operator (required):
> my-prototype-operator

Description for the operator (required):
> i've made a huge mistake

Provider's name for the operator (required):
> Ryan

Any relevant URL for the provider name (optional):
>

Comma-separated list of keywords for your operator (required):
> prototype,not-useful,junk

Comma-separated list of maintainers and their emails (e.g. 'name1:email1, name2:email2') (required):
> ryan:parkerzr@amazon.com
```


then run the bundle (yes you have to add the docker.io this time 🙄)

```sh
operator-sdk run bundle docker.io/ryparker/memcached-operator-bundle:v0.0.1
```
