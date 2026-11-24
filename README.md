# PuppyGraph on Cloudera Anywhere Cloud

Schema 2 bundle for PuppyGraph 1.1.0. The engine installs the PuppyGraph Helm chart in its own namespace. There is no operator.

`role.yaml` is a namespaced Role, `puppygraph-plugin-role`, for the engine ServiceAccount. The platform creates the RoleBinding. `helmrepositories` and `helmreleases` stay on this Role. Do not add a ClusterRole. A change to `role.yaml` does not require a new image. Republish the engine metadata.

Set the password in the engine config at deploy time. The default in `engine.yaml` is `changeme`.

## Files

| File | What it is |
| --- | --- |
| `engine.yaml` | Engine `puppygraph-plugin` 1.1.0 |
| `blueprint.yaml` | Blueprint `puppygraph`, instance minimum version 1.1.0 |
| `role.yaml` | Namespaced Role `puppygraph-plugin-role` |
| `imagecatalog.yaml` | Image `puppygraph`, tag 1.1.0 |
| `chartcatalog.yaml` | Chart `puppygraph`, version 1.0.0 |

## Publish to your registry

Set `imagecatalog.yaml` and `chartcatalog.yaml` to your registry before you publish. The image `id` and the chart `id` stay `puppygraph`. The image tag stays `1.1.0`. The chart version stays `1.0.0`.

`<host>` is the registry hostname only. `<prefix>` is the path under that host. A project name is one segment. An Artifact Registry path can be several segments.

```bash
docker login registry.example.com

docker tag puppygraph:1.1.0 \
  registry.example.com/your-prefix/puppygraph/puppygraph:1.1.0
docker push registry.example.com/your-prefix/puppygraph/puppygraph:1.1.0

helm push puppygraph-1.0.0.tgz oci://registry.example.com/your-prefix/charts
```

`imagecatalog.yaml`:

```yaml
registry: registry.example.com
repository: your-prefix/puppygraph/puppygraph
tag: "1.1.0"
```

`chartcatalog.yaml`:

```yaml
registry: registry.example.com
repository: your-prefix/charts/puppygraph
version: "1.0.0"
```

Publish from this directory. The CLI defaults to schema version 1. Pass schema version 2. `--namespace` is the same string as `<prefix>`.

```bash
awc-marketplace publish --push --version 1.1.0 \
  --destination-registry registry.example.com \
  --namespace your-prefix \
  --schema-version 2 \
  --skip-images --skip-charts \
  .

awc-marketplace publish --push --version 1.1.0 \
  --destination-registry registry.example.com \
  --namespace your-prefix \
  --schema-version 2 \
  --blueprint blueprint.yaml
```

`--skip-images` and `--skip-charts` keep the image and chart you already pushed. Those tags must exist before publish. The catalog is then at `registry.example.com/your-prefix/catalog`.

Publish the next version instead of reusing a tag that is already in the registry.

## What to send

Send the registry host, the prefix, and a pull credential. A later version on the same host and prefix does not need a new registration.

`registryHost` is the hostname only, with no scheme and no path. `marketplacePrefix` is the path under that host, with no hostname.

Artifact Registry or another registry that uses a JSON key:

```yaml
registryHost: registry.example.com
marketplacePrefix: your-prefix

auth:
  kind: gcr
  keyFile: /path/to/pull.json
```

A registry that uses a username and a token:

```yaml
registryHost: registry.example.com
marketplacePrefix: your-prefix

auth:
  kind: basic
  username: pull-user
  passwordFile: /path/to/token
```

Amazon ECR. Pull access is granted to the principal that will run `aws ecr get-login-password`:

```yaml
registryHost: 123456789012.dkr.ecr.region.amazonaws.com
marketplacePrefix: your-prefix

auth:
  kind: ecr
  region: region
```

Azure Container Registry:

```yaml
registryHost: yourregistry.azurecr.io
marketplacePrefix: your-prefix

auth:
  kind: acr
  registryName: yourregistry
```

Send the key or token on a secure channel. Do not commit it to this repo.
