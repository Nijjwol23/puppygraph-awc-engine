# PuppyGraph on Cloudera AWC

Schema 2 ACE bundle for PuppyGraph 1.1.0. This is the bundle that deployed on AWC 2.0.0-b.70 with strict ACE RBAC enabled.

`role.yaml` is a namespaced Role for the engine ServiceAccount. The AWC hub creates the RoleBinding. The engine installs the PuppyGraph Helm chart in its own namespace.

| File | What it is |
| --- | --- |
| `engine.yaml` | Engine `puppygraph-plugin` 1.1.0 |
| `blueprint.yaml` | Marketplace blueprint `puppygraph` |
| `role.yaml` | Namespaced Role used under strict ACE RBAC |
| `imagecatalog.yaml` | PuppyGraph 1.1.0 image |
| `chartcatalog.yaml` | PuppyGraph Helm chart 1.0.0 |

The image and chart references point at the lab registry used for this deploy (`docker-sandbox.infra.cloudera.com/nijjwol-awc`).
