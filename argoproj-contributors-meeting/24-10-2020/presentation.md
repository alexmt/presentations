---
marp: true
theme: default
class:
---

# Argo Contributor Experience

* Feedback
* Argo CD E2E tests

![bg right contain](https://raw.githubusercontent.com/argoproj/argoproj/master/docs/assets/argo.png)

---

## Feedback

### Github Stats August 24, 2020 – September 24, 2020

* 🎉 51 authors have pushed 108 commits
* ✅ 94 Closed Issues, 101 New Issues
* ✅ v1.8 milestone 38% complete

---

# Argo CD E2E tests

## Automated

```
make start-e2e
make test-e2e
```

## Manual

```
kubectl create ns argocd-e2e
kubectl config set-context --current --namespace=argocd-e2e
kustomize build test/manifests/base | kubectl apply -f -
goreman start
```

```
go test ./test/e2e
```

---

# Test Tools

## git-server

```
docker run --name e2e-git --rm -i \
    -p 2222:2222 -p 9080:9080 -p 9443:9443 -p 9444:9444 -p 9445:9445 \
    -w /go/src/github.com/argoproj/argo-cd -v $(pwd):/go/src/github.com/argoproj/argo-cd -v /tmp:/tmp argoproj/argo-cd-ci-builder:v1.0.0 \
    bash -c "goreman -f ./test/fixture/testrepos/Procfile start"
```

```
cat ./test/fixture/testrepos/Procfile
sshd: mkdir -p /var/run/sshd && mkdir -p ~/.ssh && cat ./test/fixture/testrepos/id_rsa.pub > ~/.ssh/authorized_keys && /usr/sbin/sshd -p 2222 -D -e
fcgiwrap: fcgiwrap -s unix:/var/run/fcgiwrap.socket & sleep 1 && chmod 777 /var/run/fcgiwrap.socket && wait
nginx: nginx -prefix=$(pwd) -g 'daemon off;' -c $(pwd)/test/fixture/testrepos/nginx.conf
```

## dev-mounter

```
go run hack/dev-mounter/main.go \
  --configmap argocd-ssh-known-hosts-cm=/tmp/argocd-local/ssh \
  --configmap argocd-tls-certs-cm=/tmp/argocd-local/tls \
  --configmap argocd-gpg-keys-cm=/tmp/argocd-local/gpg/source
```

---

# Test Framework

```golang
func TestJsonnetTlaEnv(t *testing.T) {
  /* Context */
  Given(t).                 // cleanup argocd-e2e namespace, copy test data to 
    Path("jsonnet-tla-cm"). // set app path to /tmp/argo-e2e/testdata.git/jsonnet-tla-cm
    When().

  /* Test scenario */
    Create("--jsonnet-tla-str", "foo=$ARGOCD_APP_NAME", "--jsonnet-tla-code", "bar='$ARGOCD_APP_NAME'"). // run "argocd app create ..."
    Sync().                 // run "argocd app sync"
    Then().

  /* Verification */
    Expect(OperationPhaseIs(OperationSucceeded)).
    Expect(SyncStatusIs(SyncStatusCodeSynced)).
    And(func(app *Application) {
      assert.Equal(t, Name(), FailOnErr(Run(".", "kubectl", "-n", DeploymentNamespace(), "get", "cm", "my-map", "-o", "jsonpath={.data.foo}")).(string))
      assert.Equal(t, Name(), FailOnErr(Run(".", "kubectl", "-n", DeploymentNamespace(), "get", "cm", "my-map", "-o", "jsonpath={.data.bar}")).(string))
    })
}
```

