# Kube Secrets

* [Overview](#overview)
* [Install](#install)
* [Debugging](#debugging)

## Overview

This project aims to improve developement agility by providing one configuration file to hold all kubernetes secrets in the form of simple key/value pairs. The file is broken up into sections that are parsed into ordered dictionaries which get iterated upon to generate and deploy Kubernetes secrets based on the type of secret being used.

There are 3 secret types that this project can create:

- **literals**: A data object containing key-value information.
- **files**: A file secret that is referenced by path.
- **docker**: A `kubernetes.io/dockerconfigjson` secret type for creating secrets from docker config files.

### How it Works

Let's say we have a `secrets.ini` file with the following secret names in the pod spec below. The `secret.ini` allows for values file will be parsed into 3 separate separate ordered dictionaries where they are then iterated upon to apply the appropriate Kubernetes manifest block for each secret in the dictionaries. [pystache](https://pypi.org/project/pystache/) is the module used for templating the input values into Kubernetes configuration.

```ini
[literals]
trivy-username=
trivy-password=
github-secret=
github-token=
sonar-token=

[docker]
docker-config-path=/Users/gregrobinson/.docker/config.json

[ssh]
ssh-key-path=/Users/gregrobinson/.ssh/id_rsa
```

These secrets can be referenced in a pod spec like below...

```diff
apiVersion: v1
kind: Pod
metadata:
  name: secret-env-pod
spec:
  containers:
  - name: mycontainer
    image: redis
    env:
      - name: TRIVY_USERNAME
        valueFrom:
          secretKeyRef:
+           name: trivy-username
+           key: trivy-username
      - name: TRIVY_PASSWORD
        valueFrom:
          secretKeyRef:
+           name: trivy-password
+           key: trivy-password
  restartPolicy: Never
```

The name secret `key` in `secrets.ini` is both the name and key value for Opaque literals...

## Install

1. Clone the repository. (If you want to make changes, fork the repository)

   ```bash
   git clone https://github.com/gregnrobinson/kube-secrets.git &&\
   cd kube-secrets
   ```

2. Create a file called `secrets.ini` using the snippets below, and populate the file with any secrets you want deployed to Kubernetes.

    **secrets.ini**
    Creates secrets for all secret types. The `key` refers to the secret name, and the `value` is the secret contents.

   ```bash
   cat <<EOF >./secrets.ini
   [literals]
   trivy-username=
   trivy-password=
   github-secret=
   github-token=
   sonar-token=

   [docker]
   docker-config-path=

   [ssh]
   ssh-key-path=
   EOF
   ```

3. Set the context you wish to deploy to. If left unset, resources will deploy against the currently active context.

   ```bash
   # kubectl config get-contexts
   export CONTEXT="<TARGET_CONTEXT>"
   ```

4. Deploy the secrets to Kubernetes. Rerun this command whenever changes are made to `secrets.ini`

   ```bash
   python main.py <TARGET_NAMESPACE>
   ```

### Expected Output

```diff
(.venv) gregrobinson:repos/kube-secrets$ python main.py default
+Generating secrets with Kustomize...
+Applying secrets...
secret/docker-config-path configured
secret/github-secret unchanged
secret/github-token unchanged
secret/sonar-token unchanged
secret/ssh-key-path configured
secret/trivy-password unchanged
secret/trivy-username unchanged
+Completed...
```

## Debugging

By default, all resources containing sensitive informaton are purged after script execution. If you want to preserve the generated files for troubleshooting a possible misconfiguration, set `DEBUG = True` in `main.py`. You will receive a warning reminding you that resources were not deleted.

[Back to top](#kube-secrets)
