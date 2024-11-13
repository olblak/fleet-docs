# fleet.yaml

The `fleet.yaml` file adds options to a bundle. Any directory with a
`fleet.yaml` is automatically turned into bundle.

For more information on how to use the `fleet.yaml` to customize bundles see
[Git Repository Contents](./gitrepo-content.md).

The content of the fleet.yaml corresponds to the `FleetYAML` struct at
[pkg/apis/fleet.cattle.io/v1alpha1/fleetyaml.go](https://github.com/rancher/fleet/blob/main/pkg/apis/fleet.cattle.io/v1alpha1/fleetyaml.go),
which contains the [BundleSpec](./ref-crds#bundlespec).

### Reference

```yaml title="fleet.yaml"
# The default namespace to be applied to resources. This field is not used to
# enforce or lock down the deployment to a specific namespace, but instead
# provide the default value of the namespace field if one is not specified in
# the manifests.
#
# Default: default
defaultNamespace: default

# All resources will be assigned to this namespace and if any cluster scoped
# resource exists the deployment will fail.
#
# Default: ""
namespace: default

# namespaceLabels are labels that will be appended to the namespace created by
# Fleet.
namespaceLabels:
  key: value

# namespaceAnnotations are annotations that will be appended to the namespace
# created by Fleet.
namespaceAnnotations:
  key: value

# Optional map of labels, that are set at the bundle and can be used in a
# dependsOn.selector
labels:
  key: value

kustomize:
  # Use a custom folder for kustomize resources. This folder must contain a
  # kustomization.yaml file.
  dir: ./kustomize

helm:

  # These options control how "fleet apply" downloads the chart
  #
  # Use a custom location for the Helm chart. This can refer to any go-getter
  # URL or OCI registry based helm chart URL e.g.
  # "oci://ghcr.io/fleetrepoci/guestbook".  This allows one to download charts
  # from most any location.  Also know that go-getter URL supports adding a
  # digest to validate the download. If repo is set below this field is the name
  # of the chart to lookup.
  #
  # It is possible to download the chart from a Git repository, e.g.  by using
  # `git@github.com:rancher/fleet-examples//single-cluster/helm`. If a secret
  # for the SSH key was defined in the GitRepo via `helmSecretName`, it will be
  # injected into the chart URL.
  #
  # Git repositories can be downloaded via unauthenticated http, by using for
  # example:
  #
  # `git::http://github.com/rancher/fleet-examples/single-cluster/helm`.
  chart: ./chart

  # A https URL to a Helm repo to download the chart from. It's typically easier
  # to just use `chart` field and refer to a tgz file.  If repo is used the
  # value of `chart` will be used as the chart name to lookup in the Helm
  # repository.
  repo: https://charts.rancher.io

  # The version of the chart or semver constraint of the chart to find. If a
  # constraint is specified it is evaluated each time git changes.
  #
  # The version also determines which chart to download from OCI registries.
  # Note: OCI registries don't support the '+' character, which is supported by
  # semver.  When pushing a helm chart with a tag containing the '+' character
  # helm automatically replaces '+' to '_' before uploading it.
  #
  # You should use the version with the '+' in this file, as the '_' character
  # is not supported by semver and Fleet also replaces '+' to '_' when accessing
  # the OCI registry.
  version: 0.1.0

  # By default fleet downloads any dependency found in a helm chart.  Use
  # disableDependencyUpdate: true to disable this feature.
  disableDependencyUpdate: false

  ### These options only work for helm-type bundles.
  #
  # Any values that should be placed in the `values.yaml` and passed to helm
  # during install.
  values:

    any-custom: value

    # All labels on Rancher clusters are available using
    # global.fleet.clusterLabels.LABELNAME These can now be accessed directly as
    # variables The variable's value will be an empty string if the referenced
    # cluster label does not exist on the targeted cluster.
    variableName: global.fleet.clusterLabels.LABELNAME

    # See Templating notes below for more information on templating.
    templatedLabel: "${ .ClusterLabels.LABELNAME }-foo"

    valueFromEnv:
      "${ .ClusterLabels.ENV }": ${ .ClusterValues.someValue | upper | quote }

  # Path to any values files that need to be passed to helm during install.
  valuesFiles:
    - values1.yaml
    - values2.yaml

  # Allow to use values files from configmaps or secrets defined in the
  # downstream clusters.
  valuesFrom:
    - configMapKeyRef:
        name: configmap-values
        # default to namespace of bundle
        namespace: default
        key: values.yaml
    - secretKeyRef:
        name: secret-values
        namespace: default
        key: values.yaml

  ### These options control how fleet-agent deploys the bundle, they also apply
  ### for kustomize- and manifest-style bundles.
  #
  # A custom release name to deploy the chart as. If not specified a release name
  # will be generated by combining the invoking GitRepo.name + GitRepo.path.
  releaseName: my-release
  #
  # Makes helm skip the check for its own annotations
  takeOwnership: false
  #
  # Override immutable resources. This could be dangerous.
  force: false
  #
  # Set the Helm --atomic flag when upgrading
  atomic: false
  #
  # Disable go template pre-processing on the fleet values
  disablePreProcess: false
  #
  # Disable DNS resolution in Helm's template functions
  disableDNS: false
  #
  # Skip evaluation of the values.schema.json file
  skipSchemaValidation: false
  #
  # If set and timeoutSeconds provided, will wait until all Jobs have been
  # completed before marking the GitRepo as ready.  It will wait for as long as
  # timeoutSeconds.
  waitForJobs: true

# A paused bundle will not update downstream clusters but instead mark the bundle
# as OutOfSync. One can then manually confirm that a bundle should be deployed to
# the downstream clusters.
#
# Default: false
paused: false

rolloutStrategy:

  # A number or percentage of clusters that can be unavailable during an update
  # of a bundle. This follows the same basic approach as a deployment rollout
  # strategy. Once the number of clusters meets unavailable state update will be
  # paused. Default value is 100% which doesn't take effect on update.
  #
  # default: 100%
  maxUnavailable: 15%

  # A number or percentage of cluster partitions that can be unavailable during
  # an update of a bundle.
  #
  # default: 0
  maxUnavailablePartitions: 20%

  # A number of percentage of how to automatically partition clusters if not
  # specific partitioning strategy is configured.
  #
  # default: 25%
  autoPartitionSize: 10%

  # A list of definitions of partitions.  If any target clusters do not match
  # the configuration they are added to partitions at the end following the
  # autoPartitionSize.
  partitions:

    # A user friend name given to the partition used for Display (optional).
    # default: ""
    - name: canary

      # A number or percentage of clusters that can be unavailable in this
      # partition before this partition is treated as done.
      # default: 10%
      maxUnavailable: 10%

      # Selector matching cluster labels to include in this partition
      clusterSelector:
        matchLabels:
          env: prod

      # A cluster group name to include in this partition
      clusterGroup: agroup

      # Selector matching cluster group labels to include in this partition
      clusterGroupSelector:
        clusterSelector:
          matchLabels:
            env: prod

# Target customization are used to determine how resources should be modified
# per target Targets are evaluated in order and the first one to match a cluster
# is used for that cluster.
targetCustomizations:

  # The name of target. If not specified a default name of the format
  # "target000" will be used. This value is mostly for display
  - name: prod

    # Custom namespace value overriding the value at the root.
    namespace: newvalue

    # Custom defaultNamespace value overriding the value at the root.
    defaultNamespace: newdefaultvalue

    # Custom kustomize options overriding the options at the root.
    kustomize: {}

    # Custom Helm options override the options at the root.
    helm: {}

    # If using raw YAML these are names that map to overlays/{name} that will be
    # used to replace or patch a resource. If you wish to customize the file
    # ./subdir/resource.yaml then a file
    # ./overlays/myoverlay/subdir/resource.yaml will replace the base file.  A
    # file named ./overlays/myoverlay/subdir/resource_patch.yaml will patch the
    # base file.  A patch can in JSON Patch or JSON Merge format or a strategic
    # merge patch for builtin Kubernetes types. Refer to "Raw YAML Resource
    # Customization" below for more information.
    yaml:
      overlays:
        - custom2
        - custom3

    # A selector used to match clusters.  The structure is the standard
    # metav1.LabelSelector format. If clusterGroupSelector or clusterGroup is
    # specified, clusterSelector will be used only to further refine the
    # selection after clusterGroupSelector and clusterGroup is evaluated.
    clusterSelector:
      matchLabels:
        env: prod

    # A selector used to match a specific cluster by name. When using Fleet in
    # Rancher, make sure to put the name of the clusters.fleet.cattle.io
    # resource.
    clusterName: dev-cluster

    # A selector used to match cluster groups.
    clusterGroupSelector:
      matchLabels:
        region: us-east

    # A specific clusterGroup by name that will be selected.
    clusterGroup: group1

    # Resources will not be deployed in the matched clusters if doNotDeploy is
    # true.
    doNotDeploy: false

    # Drift correction removes any external change made to resources managed by
    # Fleet.  It performs a helm rollback, which uses a three-way merge strategy
    # by default.  It will try to update all resources by doing a PUT request if
    # force is enabled.  Three-way strategic merge might fail when updating an
    # item inside of an array as it will try to add a new item instead of
    # replacing the existing one.  This can be fixed by using force.  Keep in
    # mind that resources might be recreated if force is enabled.  Failed
    # rollback will be removed from the helm history unless keepFailHistory is
    # set to true.
    correctDrift:
      enabled: false
      force: false # Warning: it might recreate resources if set to true
      keepFailHistory: false

# dependsOn allows you to configure dependencies to other bundles. The current
# bundle will only be deployed, after all dependencies are deployed and in a
# Ready state.
dependsOn:

  # Format:
  #     <GITREPO-NAME>-<BUNDLE_PATH> with all path separators replaced by "-"
  #
  # Example:
  #
  #      GitRepo name "one", Bundle path "/multi-cluster/hello-world"
  #      results in "one-multi-cluster-hello-world".
  #
  # Note:
  #
  #   Bundle names are limited to 53 characters long. If longer they will be
  #   shortened:
  #
  #     opni-fleet-examples-fleets-opni-ui-plugin-operator-crd becomes
  #     opni-fleet-examples-fleets-opni-ui-plugin-opera-021f7
  - name: one-multi-cluster-hello-world

  # Select bundles to depend on based on their label.
  - selector:
      matchLabels:
        app: weak-monkey

# Ignore fields when monitoring a Bundle. This can be used when Fleet thinks
# some conditions in Custom Resources makes the Bundle to be in an error state
# when it shouldn't.
ignore:

  # Conditions to be ignored
  conditions:

    # In this example a condition will be ignored if it contains
    # {"type": "Active", "status", "False"}
    - type: Active
      status: "False"

# Override targets defined in the GitRepo. The Bundle will not have any targets
# from the GitRepo if overrideTargets is provided.
overrideTargets:
  - clusterSelector:
      matchLabels:
        env: dev
```

### Helm Options

#### How fleet-agent deploys the bundle

These options also apply to kustomize- and manifest-style bundles.  They control
how the fleet-agent deploys the bundle. All bundles are converted into Helm
charts and deployed with the Helm SDK.  These options are often similar to the
Helm CLI options for install and update.

- releaseName
- takeOwnership
- force
- atomic
- disablePreProcess
- disableDNS
- skipSchemaValidation
- waitForJobs

#### Helm Chart Download Options

These options are for Helm-style bundles, they specify how to download the
chart.

- chart
- repo
- version

The reference to the chart can be either:

- a local path in the cloned Git repository, specified by `chart`.
- a [go-getter URL](https://github.com/hashicorp/go-getter?tab=readme-ov-file#url-format),
  specified by `chart`. This can be used to download a tarball
  of the chart. go-getter also allows to download a chart from a Git repo.
- a Helm repository, specified by `repo` and optionally `version`.
- an OCI Helm repository, specified by `repo` and optionally `version`.

#### Helm Chart Value Options

Options for the downloaded Helm chart.

- values
- valuesFiles
- valueFrom

### Values

Values are processed in different stages of the lifecycle: https://fleet.rancher.io/ref-bundle-stages

* fleet.yaml `values:` and `valuesFile:` are added to the bundle's values when it is created.
* helm values templating, e.g. with `${ }` happens when the bundle is targeted at a cluster, cluster labels filled in, etc.
* When the agent installs the chart, values from `valuesFrom` are read. Then Helm templating `{{ }}` is processed.

### Templating

It is possible to specify the keys and values as go template strings for
advanced templating needs.  Most of the functions from the [sprig templating
library](https://masterminds.github.io/sprig/) are available.

Note that if the functions output changes with every call, e.g. `uuidv4`, the
bundle will get redeployed.

The template context has the following keys:

* `.ClusterValues` are retrieved from target cluster's `spec.templateValues`
* `.ClusterLabels` and `.ClusterAnnotations` are the labels and annotations in
  the cluster resource.
* `.ClusterName` as the fleet's cluster resource name.
* `.ClusterNamespace` as the namespace in which the cluster resource exists.

To access Labels or Annotations by their key name:

```
${ get .ClusterLabels "management.cattle.io/cluster-display-name" }
```

Note: The fleet.yaml must be valid yaml. Templating uses `${ }` as delims,
unlike Helm which uses `{{ }}`.  These fleet.yaml template delimiters can be
escaped using backticks, eg.:

```
foo-bar-${`${PWD}`}
```

will result in the following text:

```
foo-bar-${PWD}
```