# PuppyGraph ACE engine

Schema 2 bundle for PuppyGraph 1.1.0. Deploy this tree as it is on a Cloudera Anywhere Cloud build newer than 2.0.0-b.70.

Last live test: 2.0.0-b.70, strict ACE RBAC on. The partner confirmed that deploy. The ace-hub pods on that console have been running since 10 November 2026. The ClusterRole rule that rejects namespace-scoped Flux kinds, AWC-6150, merged on 12 November 2026, so 2.0.0-b.70 does not contain it. This `role.yaml` was checked on 19 November 2026 with `validate-engine-rbac` from ace-hub `b784beef` and passed: 0 ClusterRole, 1 Role. A newer 2.0.0.x hub is the build that enforces that gate at deploy time.

Do not rebuild the PuppyGraph image or the Helm chart for this test. `role.yaml` has not changed since the b.70 deploy.

## Files

| File | What it is |
| --- | --- |
| `engine.yaml` | Engine `puppygraph-plugin` 1.1.0. Helm install in the engine namespace. No operator |
| `blueprint.yaml` | Blueprint `puppygraph`, instance minimum version 1.1.0 |
| `role.yaml` | Namespaced Role `puppygraph-plugin-role`. ACE Hub creates the RoleBinding |
| `imagecatalog.yaml` | Image `puppygraph` tag 1.1.0 |
| `chartcatalog.yaml` | Chart `puppygraph` version 1.0.0 |

`role.yaml` grants `helmrepositories` and `helmreleases` on the Role. There is no ClusterRole. On a hub that contains AWC-6150, a ClusterRole that names those kinds is rejected. Leave this Role as it is.

The password field in `engine.yaml` defaults to `changeme`. Set it on the deploy. It is not a credential for the registry.

## Registry already published

The catalogs point at the lab registry used for the b.70 deploy. Leave them pointed there for this test.

| Object | Reference |
| --- | --- |
| Image | `docker-sandbox.infra.cloudera.com/nijjwol-awc/puppygraph/puppygraph:1.1.0` |
| Chart | `docker-sandbox.infra.cloudera.com/nijjwol-awc/charts/puppygraph` version 1.0.0 |
| Catalog | `docker-sandbox.infra.cloudera.com/nijjwol-awc/catalog` |

Register that registry on the 2.0.0.x control plane if it is not already listed. Host and prefix only:

- `registryHost`: `docker-sandbox.infra.cloudera.com`
- `marketplacePrefix`: `nijjwol-awc`

[awc-oci-register](https://github.com/agupta-git/awc-oci-register) adds the pull secret and appends that host and prefix to the marketplace list. The marketplace polls about every 5 minutes.

Publish again only if that console does not list the blueprint. The CLI defaults to schema version 1. This bundle is schema 2.

```bash
awc-marketplace publish --push --version 1.1.0 \
  --destination-registry docker-sandbox.infra.cloudera.com \
  --namespace nijjwol-awc \
  --schema-version 2 \
  --skip-images --skip-charts \
  .

awc-marketplace publish --push --version 1.1.0 \
  --destination-registry docker-sandbox.infra.cloudera.com \
  --namespace nijjwol-awc \
  --schema-version 2 \
  --blueprint blueprint.yaml
```

`--skip-images` and `--skip-charts` reuse the image and chart already in the lab registry. Those tags have to exist before publish.
