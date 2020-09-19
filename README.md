# gcloud CLI Mixin for Porter

This is a mixin for Porter that provides the gcloud CLI.

[![Build Status](https://dev.azure.com/getporter/porter/_apis/build/status/gcloud-mixin?branchName=main)](https://dev.azure.com/getporter/porter/_build/latest?definitionId=7&branchName=main)

<img src="https://porter.sh/images/mixins/google.png" align="right" width="150px"/>

## Mixin Syntax

See the [gcloud CLI Command Reference](https://cloud.google.com/sdk/gcloud/reference/) for the supported commands

```yaml
gcloud:
  description: "Description of the command"
  groups: GROUP
  command: COMMAND
  arguments:
  - arg1
  - arg2
  flags:
    FLAGNAME: FLAGVALUE
    REPEATED_FLAG:
    - FLAGVALUE1
    - FLAGVALUE2
  suppress-output: false
  outputs:
    - name: NAME
      jsonPath: JSONPATH
```

You can also specify a list of `groups`:

```yaml
gcloud:
  description: "Description of the command"
  groups:
  - GROUP 1
  - GROUP 2
  command: COMMAND
```

### Suppress Output

The `suppress-output` field controls whether output from the mixin should be
prevented from printing to the console. By default this value is false, using
Porter's default behavior of hiding known sensitive values. When 
`suppress-output: true` all output from the mixin (stderr and stdout) are hidden.

Step outputs (below) are still collected when output is suppressed. This allows
you to prevent sensitive data from being exposed while still collecting it from
a command and using it in your bundle.

### Outputs

The mixin supports `jsonpath` outputs.


#### JSON Path

The `jsonPath` output treats stdout like a json document and applies the expression, saving the result to the output.

```yaml
outputs:
- name: NAME
  jsonPath: JSONPATH
```

For example, if the `jsonPath` expression was `$[*].id` and the command sent the following to stdout: 

```json
[
  {
    "id": "1085517466897181794",
    "name": "my-vm"
  }
]
```

Then then output would have the following contents:

```json
["1085517466897181794"]
```

---

## Examples

### Authenticate

```yaml
gcloud:
  description: "Authenticate"
  groups:
    - auth
  command: activate-service-account
  flags:
    key-file: gcloud.json
```

### Provision a VM

```yaml
gcloud:
  description: "Create VM"
  groups:
    - compute
    - instances
  command: create
  arguments:
    - porter-test
  flags:
    project: porterci
    zone: us-central1-a
    machine-type: f1-micro
    image: debian-9-stretch-v20190729
    image-project: debian-cloud
    boot-disk-size: 10GB
    boot-disk-type: pd-standard
    boot-disk-device-name: porter-test
  outputs:
    - name: vms
      jsonPath: "$[*].id"
```

### Configure SSH Keys

```yaml
gcloud:
  description: "Configure SSH"
  groups: compute
  command: config-ssh
  flags:
    ssh-config-file: ./gce-ssh-config
    ssh-key-file: ./gce-ssh-key
```
