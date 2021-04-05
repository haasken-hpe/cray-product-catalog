# Cray Product Catalog

This repository contains the Docker image definition for the cray-product-catalog
image. This image provides a script that uploads the contents of a yaml file to
a product catalog entry, which serves as a kubernetes config map.

At minimum, the `catalog_update.py` script takes a `PRODUCT` and
`PRODUCT_VERSION` and applies the content of a file denoted by `YAML_CONTENT`
file as follows:

```yaml
{PRODUCT}:
  {PRODUCT_VERSION}:
    {content of yaml file (in YAML_CONTENT)}
```

The product catalog is a software inventory of sorts, and allows for system
users to view a product and its associated versions and version metadata that
have been _installed_ on the system.

The cray-product-catalog image is assumed to be running in the Shasta
Kubernetes cluster by an actor that has permissions to read and update config
maps in the namespace that is configured.

## Getting Started

The main use case for cray-product-catalog is for a product to add install-time
information and metadata to the cray-product-catalog config map located in the
services namespace via a Kubernetes job as part of a Helm chart. The image
could also be used via podman on an NCN, but this has not been tested.

## Example Usage

### Helm Chart

Two seminal examples of using cray-product-catalog are the `cray-import-config`
and `cray-import-kiwi-recipe-image` base charts. Review the values files and
Kubernetes job template to see cray-product-catalog in action.

* [cray-import-config](https://stash.us.cray.com/projects/SCMS/repos/cray-product-install-charts/browse/charts/cray-import-config)
* [cray-import-kiwi-recipe-image](https://stash.us.cray.com/projects/SCMS/repos/cray-product-install-charts/browse/charts/cray-import-kiwi-recipe-image)

### Podman on NCN

To create an entry in the config map for an "example" product with version
1.2.3, you can use podman on a Kubernetes worker/master node. Be sure to mount
the Kubernetes config file into the running container as well as the
`YAML_CONTENT`.

```bash
ncn-w001:~/ # podman run --rm --name example-cpc --network podman-cni-config \
    -e PRODUCT=example \
    -e PRODUCT_VERSION=1.2.3 \
    -e YAML_CONTENT=/results/example.yaml \
    -e KUBECONFIG=/.kube/admin.conf \
    -v /etc/kubernetes:/.kube:ro \
    -v ${PWD}:/results:ro \
    dtr.dev.cray.com/cray/cray-product-catalog-update:0.1.0-20201215000547_74d64e3
Updating config_map=cray-product-catalog in namespace=services for product/version=example/1.2.3
Retrieving content from /results/example.yaml
Resting 3s before reading ConfigMap
Product=example does not exist; will update
ConfigMap update attempt=1
Resting 2s before reading ConfigMap
ConfigMap data updates exist; Exiting.
```

View the results in a nice format:

```bash
ncn-w001:~/ # kubectl get cm -n services cray-product-catalog -o json | jq .data.example | ./yq r -
1.2.3:
  this:
    is: some
    yaml: stuff
```

## Configuration

All configuration options are provided as environment variables.

### Required Environment Variables

* `PRODUCT` = (no default)

> The name of the Cray/Shasta product that is being cataloged.

* `PRODUCT_VERSION` = (no default)

> The SemVer version of the Cray/Shasta product that is being imported, e.g.
  `1.2.3`.

* `YAML_CONTENT`=  (no default)

> The filesystem location of the YAML that will be added to the config map.

### Optional Environment Variables

 * `CONFIG_MAP` = `cray-product-catalog`

 > The name of the config map to add the `YAML_CONTENT` to.

 * `CONFIG_MAP_NAMESPACE` = `services`

 > The Kubernetes namespace of the `CONFIG_MAP`.

## Versioning
We use [SemVer](http://semver.org/) for versioning. See the `.version` file in
the repository root for the current version. Please update that version when
making changes.

## Contributing

CMS folks, make a branch. Others, make a fork.

## Built With

* Alpine Linux
* Python 3
* Python Requests
* Kubernetes Python Client
* Docker
* Good intentions

## Blamelog

* _0.1.0_ - initial release for Shasta 1.4, addition to CSM product stream - David Laine (david.laine@hpe.com)
* _0.0.1_ - initial implementation - Randy Kleinman (randy.kleinman@hpe.com)

## License
Copyright 2020 Hewlett Packard Enterprise Development LP