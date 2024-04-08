# crossplane-objectstorage
Crossplane Configuration delivering CRDs to provision a AWS S3 Bucket showcasing Uptest e2e testing

> __WARNING:__ uptest is a work in progress and hardly ever used by any other than Upbound staff themselves. See this issue comment: https://github.com/upbound/official-providers-ci/issues/153#issuecomment-1756317685: "I think we have to be honest and document somewhere that currently uptest is not really usable without surrounding make targets and the build module :)"

This project was not created to show, how to provision an S3 bucket with Crossplane (this was already done here https://github.com/jonashackt/crossplane-aws-azure), but to show how to use Uptest. It build fairly simple:

```shell
├── apis
│   └── objectstorage
│       ├── composition.yaml
│       └── definition.yaml
├── examples
│   └── objectstorage
│       └── claim.yaml
```

We have a simple Composition and XRD under `apis` dir. And a corresponding example XRC under `examples` dir.

# Uptest

https://github.com/crossplane/uptest

> The end to end integration testing tool for Crossplane providers and configurations.

Uptest is based on https://kuttl.dev/ & generates a kuttl test case based on the provided input. You can even inspect the generated kuttl test case by checking the temporary test directory which is printed in the beginning of uptest e2e output.

> Uptest expects a running control-plane (a.k.a. k8s + crossplane) where required providers are running and/or required configuration were applied.

So running uptest without a working Crossplane managed cluster setup isn't possible (like [this](https://github.com/jonashackt/crossplane-aws-azure) or [this](https://github.com/jonashackt/crossplane-argocd) one). This is also stated when running the `uptest e2e` command: 

> "Run e2e tests for manifests by applying them to a control plane and waiting until a given condition is met."


### Install uptest

> "Uptest comes as a binary which can be installed from the releases section."

Strangely this currently means, the binary is available from https://github.com/upbound/official-providers-ci/releases - but not from https://github.com/crossplane/uptest/releases! 

```shell
# For Mac ARM
curl -sfLo uptest "https://github.com/upbound/official-providers-ci/releases/download/v0.11.1/uptest_darwin-arm64"

# For Linux / WSL
curl -sfLo uptest "https://github.com/upbound/official-providers-ci/releases/download/v0.11.1/uptest_linux-amd64"

# any OS
chmod +x uptest
sudo mv uptest /usr/local/bin
``` 

If this went well, the `uptest` command should work on your system:

```shell
$ uptest e2e --help
usage: uptest e2e [<flags>] [<manifest-list>]

Run e2e tests for manifests by applying them to a control plane and waiting until a given condition is met.

Flags:
  --help                         Show context-sensitive help (also try --help-long and --help-man).
  --data-source=""               File path of data source that will be used for injection some values.
  --setup-script=""              Script that will be executed before running tests.
  --teardown-script=""           Script that will be executed after running tests.
  --default-timeout=1200         Default timeout in seconds for the test. Timeout could be overridden per resource
                                 using "uptest.upbound.io/timeout" annotation.
  --default-conditions="Ready"   Comma separated list of default conditions to wait for a successful test.
                                 Conditions could be overridden per resource using "uptest.upbound.io/conditions"
                                 annotation.
  --skip-delete                  Skip the delete step of the test.
  --test-directory="/tmp/uptest-e2e"  
                                 Directory where kuttl test case will be generated and executed.
  --only-clean-uptest-resources  While deletion step, only clean resources that were created by uptest

Args:
  [<manifest-list>]  List of manifests. Value of this option will be used
                     to trigger/configure the tests.The possible usage:
                     'provider-aws/examples/s3/bucket.yaml,provider-gcp/examples/storage/bucket.yaml': The comma
                     separated resources are used as test inputs. If this option is not set, 'MANIFEST_LIST' env var
                     is used as default.

``` 

### Install kuttl kubectl plugin

Since uptest is based on kuttl, we need to install it before - see https://github.com/upbound/official-providers-ci/issues/153. Otherwise we'll get errors like: 

```shell
uptest e2e examples/objectstorage/claim.yaml --test-directory="$PWD/temp/uptest-e2e" --setup-script="test/setup.sh"
Running kuttl tests at /home/jonashackt/dev/crossplane-objectstorage/temp/uptest-e2e
bash: line 1: : command not found
uptest: error: cannot run e2e tests successfully: cannot execute tests: kuttl failed: exit status 127
```

https://kuttl.dev/docs/cli.html 

One of the best ways to install a kubectl plugin is to you the package manager [krew](https://github.com/kubernetes-sigs/krew). So first install krew via your preferred package manager (see https://krew.sigs.k8s.io/docs/user-guide/setup/install/):

```shell
# Manjaro
pamac install krew
```
(if that gives an error like `go: module cache not found: neither GOMODCACHE nor GOPATH is set`, try without sudo).

Now install kuttl via krew:

```shell
$ kubectl krew install kuttl

WARNING: To be able to run kubectl plugins, you need to add
the following to your ~/.zshrc:

    export PATH="${KREW_ROOT:-$HOME/.krew}/bin:$PATH"

and restart your shell.
```

Add the `export ...` statement to your shell configuration.


Finally define the following environment varibable:

```shell
export KUTTL='kubectl kuttl'  
```



### Use Uptest

As mentioned above you need to have a working Crossplane setup in place (like [this](https://github.com/jonashackt/crossplane-aws-azure) or [this](https://github.com/jonashackt/crossplane-argocd) one)!

> The repository https://github.com/upbound/official-providers-ci contains GitHub Action bases examples of workflows using `uptest e2e`.

> See for example this workflow: https://github.com/upbound/official-providers-ci/blob/main/.github/workflows/pr-comment-trigger.yml 

> It may be used like this: https://github.com/upbound/configuration-gitops-argocd/blob/main/.github/workflows/e2e.yaml



So here we go, let's test our simple Bucket composition:

```shell
uptest e2e examples/objectstorage/claim.yaml --test-directory="$PWD/temp/uptest-e2e" --setup-script="test/setup.sh"
```