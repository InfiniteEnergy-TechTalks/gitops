# [GitOps]

A complete cluster flux

[GitOps]: https://www.weave.works/technologies/gitops/

---

## What is it?

Originally coined by [Weaveworks], GitOps is:

***"Operations by pull request"*** <!-- .element: class="fragment" -->

[Weaveworks]: https://weave.works

--

### Operations

Making *declarative* <!-- .element: class="fragment highlight-blue" --> changes to infrastructure and applications

Say **what** you want, rather than **how** to get there

<!-- .element: class="fragment" -->

```shell
kubectl apply -f $THING # one and done

vs

apt-get ... # install $THING
scp ... # copy files to setup/configure $THING
vi ... # maybe manually configure $THING
systemd/systemctl ... # make sure $THING starts/runs as intended
```

<!-- .element: class="fragment" -->

--

### By...pull request?

*"What's the current state of the world?"*

Version control should be the truth <!-- .element: class="fragment" -->

Git is today's de facto version control tool <!-- .element: class="fragment" -->

--

![GitOps Pipeline](https://lh3.googleusercontent.com/Wyh241CIsJA3zGzzlx8M7Np92b22KJIRde0TvnFGuA2G4opetTOcdmQHOEja1tNEyzlWq9gQJzBSLtWGr8D18hR2zm2Y2nzeyVbVkUh3IV2dLSMmGCSOwc5pympw8Eqp6eiq4tMd)

---

## Why use it?

Because it's new and cool

Isn't that reason enough? <!-- .element: class="fragment" -->

--

### The state of the world

How many places would we need to look to figure it out?

* Git? <!-- .element: class="fragment" -->
* Individual VMs? <!-- .element: class="fragment" -->
* Docker Swarm? <!-- .element: class="fragment" -->
* Azure? <!-- .element: class="fragment" -->
* Databases? <!-- .element: class="fragment" -->
* That sticky note on your monitor? <!-- .element: class="fragment" -->

--

### What if it changes?

How do we know?

![Flux Slack Alert](https://images.contentstack.io/v3/assets/blt300387d93dabf50e/blt6691c5ffa9b24751/59888079727eb46e09477548/download) <!-- .element: class="fragment fade-up" -->

--

### Powerful tools

And a lot to learn

`kubectl` and `helm` are awesome

<!-- .element: class="fragment" -->

`terraform` is awesome

<!-- .element: class="fragment" -->

...but not everyone knows them, nor should they need to <!-- .element: class="fragment" -->

--

### Leverage what we know

* Git
* GitLab
* Your favorite editor (it's OG Notepad, right?)

---

## How does it work?

![Peewee Machine](https://i.imgur.com/wmBkvIo.gif)

--

Our use case is centered around [Kubernetes]

[Kubernetes]: https://kubernetes.io

--

![GitOps Pipeline](https://lh3.googleusercontent.com/Wyh241CIsJA3zGzzlx8M7Np92b22KJIRde0TvnFGuA2G4opetTOcdmQHOEja1tNEyzlWq9gQJzBSLtWGr8D18hR2zm2Y2nzeyVbVkUh3IV2dLSMmGCSOwc5pympw8Eqp6eiq4tMd)

--

### Workflow

1. Commit changes to application repo <!-- .element: class="fragment" -->
2. CI pipeline builds and tests app <!-- .element: class="fragment" -->
3. If successful, publishes new container image <!-- .element: class="fragment" -->
4. Update config repo to use new image <!-- .element: class="fragment" -->
5. Operator sees change <!-- .element: class="fragment" -->
6. Operator deploys updated manifest to cluster <!-- .element: class="fragment" -->

--

### Manifests

Define what resources to create in cluster

```yaml
...
spec:
  replicas: 1
  template:
    spec:
      containers:
        - name: gitops-demo
          image: "my-image:master"
          imagePullPolicy: Always
          ports:
            - name: http
              containerPort: 8080
              protocol: TCP
...
```

--

### [Operators]

An operator is a Kubernetes native application

Extends the Kubernetes API

Provides domain-specific functionality

Commonly used to manage stateful apps, like [etcd]

[Operators]: https://coreos.com/blog/introducing-operators.html
[etcd]: https://github.com/coreos/etcd-operator

---

## [Weave Flux]

The GitOps Operator

[Weave Flux]: https://github.com/weaveworks/flux

--

Monitors config and image repos for changes

Triggers a deployment when a diff is found

Config repo tells flux *what* <!-- .element: class="fragment highlight-blue" --> we want deployed

--

### Some other features

* Automate releases when new images are pushed
* Apply policies for image tags
* Mark resources as ignored to avoid deployment
* ...

--

### Annotations!

```yaml
apiVersion: helm.integrations.flux.weave.works/v1alpha2
kind: FluxHelmRelease
metadata:
  name: prometheus
  namespace: monitoring
  annotations:
    flux.weave.works/automated: 'true'
    flux.weave.works/tag.chart-image: semver:*
```

--

### Demo

---

## [Sealed Secrets]

DevOps Tested, Security Engineer Approved

[Sealed Secrets]: https://github.com/bitnami-labs/sealed-secrets

--

### The problem with secrets

Kubernetes secrets are encrypted at rest

Their manifests are not <!-- .element: class="fragment" -->

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: gitops-demo-secret
type: Opaque
data:
  # These values are just Base64 encoded!
  username: YWJ3aGl0bGVy
  password: c3VwZXJzZWNyZXRwYXNzd29yZA==
```

<!-- .element: class="fragment" -->

--

<!-- .slide: data-background="https://i.imgur.com/u9GqzdB.jpg" data-background-opacity="0.5" -->

```shell
$ echo -n YWJ3aGl0bGVy | base64 -D
abwhitler
```

<!-- .element: class="fragment" -->

```shell
$ echo -n c3VwZXJzZWNyZXRwYXNzd29yZA== | base64 -D
supersecretpassword
```

<!-- .element: class="fragment" -->

--

### Secrets + Git

`kubeseal` provides encrypted manifests

```shell
$ kubeseal < secret.yaml > secret-sealed.yaml
```

```yaml
apiVersion: bitnami.com/v1alpha1
kind: SealedSecret
metadata:
  creationTimestamp: null
  name: gitops-demo-secret
  namespace: default
spec:
  encryptedData:
    password: AgAv/689SC2wNr8snTar7RIEoEe7Xhm1nHpq+Qr8xQx13fLZa4t7NncZSgfXnffoNSpGmSZvLv7JWl59McxB048bYk2dn+2cGEnOcuHvuUp93u3/OQ/S9/XDZYyGNL4CV33IFL+2wL3pUeND6BEmxzZoshr+1pY09GWH3oy4OnDRmo2FohPysC5lwSp5Ho4DDSdo+X4rxjEaIwHv7v6rm4YR90n+uv/Poc8kq6sf9kEt1PWv6J9YqKnKa/59x+HPXXWqKzaiWbVsQOO4JVnpYp/Wi//T9E4Si+7j7WMSz+umzAfL4YeiTicLGKXiLO2hWCYi7Pn8QyL4xxwI+726ZPBIcjvHkQJQz8DEAJu28NTnhZfIah5z62nd6orMWck10TN/Juqk4O3dD3ZJ5kL7NKw99ljJ+bysQzjISa8cY6SMCulOTKMiDW5dRdkOrkXITjzdpCrBXDsaNBUi0KH58v2YwivbMsbDHLqHAAVutt8I2wM6mHCWcRPlp/zfKQgpR0VcZ1bnkqeKcLTKMJ/etp+kiy3RL0ogU6p5OzLUlcQgLXirP4LL2s1SEh7sgzNQWe1ol/r3ozrgaqWrGvK97Y3yUHStQks3LAITp+57hJbVdsGUTds6UKs4QvY/7bVBzDIAFIfJ/B/zzunOgOO2tyYZveOtZPHnPwaAW2bpaQqK+xLNhKjbN+cCJf2u/ewh50P9IlYz+75k
    username: AgBhqjQjcfhnpv5Pu1gn+a1kUwUy7NP0dFqIrZJCxC0cl7HQ/KN08q4Sf8CZxhtzobGtVg/XQARClD+Fe9KGYZBjV4sNPsB3m7b6+xEMGd1xq4HzVxopVRrhW9IhvTysrQojPYG/7IkAuqmKFQMqbpRTXYT/E3ybeJQWThErZt1DpW59Z+TaoUQN8XwgyLDUfWycYIHOSopVGAldXCII+ieqaCgoYkHGk6zrQ7lFhDwZMle1O2X5+V16jD4dbf2V8lHBvOywNEwve86wfxLaI7sIr8wsltowRtnWYyIJIN2a5TLZXUQUsLCZgvggIexQ0w6sA9jv62kI5JQEQxI3ngr7L3AhFF9E+KEumXHmMln4/yaNoSlG79KHiyuOPfIu0K/K4a6HfAkY1HayNfyMTALHtNWp2DLU3uH1Mcb24nXEzxx+5SOoRrre7tX+OC/KV4r5o5LJGf3PfY1CKpatUnRfTAHLYh25aSWuAq3XoFVaQ/+9S3bHMUca/VAAJD/PR8bdcEjSGJPqA0QlquPjhj4zbzJ2RQOE8uI69Kp+ZeBq0+ghS2bC8VBowBwBLhxRkQvKjvyzI1e4bUpEuG4IFGxtkN/EGL+r114RG6Ps8BytGc2mYgk+3/uJvyQpCXfAVv47ijoOqCkoSVQgJd4dTjqLslYcAPqj5yxG6BNtUiJe4yd4Y//VZd+q1sWeDDLLhgtZ+g3W927kxq8=
```

---

## Questions?
