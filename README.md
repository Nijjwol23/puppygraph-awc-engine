# PuppyGraph on Cloudera AWC

Schema 2 ACE bundle for PuppyGraph 1.1.0. This is the bundle that deployed on AWC 2.0.0-b.70 with strict ACE RBAC enabled.

`role.yaml` is a namespaced Role for the engine ServiceAccount. The AWC hub creates the RoleBinding. The engine installs the PuppyGraph Helm chart in its own namespace. Keep this Role. A ClusterRole is rejected under strict ACE RBAC. A `role.yaml` change does not require a new PuppyGraph image. Republish the engine metadata.

| File | What it is |
| --- | --- |
| `engine.yaml` | Engine `puppygraph-plugin` 1.1.0 |
| `blueprint.yaml` | Marketplace blueprint `puppygraph` |
| `role.yaml` | Namespaced Role used under strict ACE RBAC |
| `imagecatalog.yaml` | PuppyGraph 1.1.0 image |
| `chartcatalog.yaml` | PuppyGraph Helm chart 1.0.0 |

The catalog files in this repo point at the lab registry used for the first deploy (`docker-sandbox.infra.cloudera.com/nijjwol-awc`). Point them at your registry before you publish.

## What you publish

You own this bundle, the PuppyGraph image, and the Helm chart. Publish them to your OCI registry with the `awc-marketplace` CLI from Cloudera. Use schema version 2. This console does not list schema 1.

```bash
docker login your-registry.example.com

docker tag puppygraph:1.1.0 \
  your-registry.example.com/your-namespace/puppygraph/puppygraph:1.1.0
docker push your-registry.example.com/your-namespace/puppygraph/puppygraph:1.1.0

helm push puppygraph-1.0.0.tgz oci://your-registry.example.com/your-namespace/charts
```

Set `imagecatalog.yaml` and `chartcatalog.yaml` to that registry, repository, and tag. The image `id` and the chart `id` stay `puppygraph`. Then, from this directory:

```bash
awc-marketplace publish --push --version 1.1.0 \
  --destination-registry your-registry.example.com \
  --namespace your-namespace \
  --schema-version 2 \
  --skip-images --skip-charts \
  .

awc-marketplace publish --push --version 1.1.0 \
  --destination-registry your-registry.example.com \
  --namespace your-namespace \
  --schema-version 2 \
  --blueprint blueprint.yaml
```

`--skip-images` and `--skip-charts` keep the copies you already pushed. The CLI still resolves their digests, so those tags must exist first. Publish the next version instead of reusing a tag that is already in the registry.

## What you send Cloudera

Send three things:

- `registryHost`, the hostname only, such as `your-registry.example.com`
- `marketplacePrefix`, the namespace under that host, such as `your-namespace`
- Pull credentials for that registry (Harbor or Docker basic auth: username and a token file)

Cloudera registers that registry on the AWC control plane with [awc-oci-register](https://github.com/agupta-git/awc-oci-register). That adds the pull secret and appends `registryHost` plus `marketplacePrefix` to the marketplace list, then restarts the console when the list changes. The marketplace polls about every 5 minutes and picks up the catalog at `your-registry.example.com/your-namespace/catalog`.

A new version in the same registry does not need a new registration. Send a new host or prefix only when those change.

This is the Helm path PuppyGraph is on today. Operator-based engines are a separate path and are not covered here.
