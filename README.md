# Deploy Bitwarden in Kubernetes

This projects contains Kubernetes manifest files for deploying a production-ready self-hosted Bitwarden solution in Kubernetes. It utilizes the Rust-based implementation from [https://github.com/dani-garcia/bitwarden_rs](https://github.com/dani-garcia/bitwarden_rs).

## Configuration

* Objects are created in the `bitwarden` namespace.
* This project specifically deploys to Amazon AWS and uses an [nginx ingress controller](https://kubernetes.github.io/ingress-nginx/), and thus provisions an NLB instance. Refer to [this](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/) guide on Ingress Controllers for more options.
* Application image: the statefulset deploys the default Debian-based image, which in turn uses SQLite for the database. In most use-cases this should be enough. Otherwise, please refer to [this](https://github.com/dani-garcia/bitwarden_rs/wiki/Which-container-image-to-use) guide for choosing the right application image.

secrets.yaml:

* SMTP credentials: The base64-encoded SMTP username and password.
* Admin token: The admin token used to login to the admin page (`/admin`). Hint: generate one using `openssl rand -base64 48`.

configmap.yaml:

* Set the application URL with the variable **DOMAIN**.
* Set the domain with the variable **SIGNUPS_DOMAINS_WHITELIST** to restrict email sign-ups for users with that domain. Please note that the administrator would still be able to invite users outside of this domain using the admin page (`/admin`).
* For more configuration parameters, such as adding U2F and YubiKey support, please refer to the [bitwarden-rs wiki](https://github.com/dani-garcia/bitwarden_rs/wiki).

## Prerequisites

### Storage class

For simplicity, the statefulset makes use of a [storage class](https://kubernetes.io/docs/concepts/storage/storage-classes/#aws-ebs). Here is a working example for a storage class manifest which provisions Elastic Block storage.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: sc-ebs-gp2-zone1
  annotations:
    storageclass.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/aws-ebs
reclaimPolicy: Retain
allowVolumeExpansion: true
volumeBindingMode: WaitForFirstConsumer
parameters:
  type: gp2
allowedTopologies:
  - matchLabelExpressions:
    - key: failure-domain.beta.kubernetes.io/zone
      values:
        - ap-southeast-1a
```

### SSL certificate

This project expects a secret named `foo-io-wildcardssl` in the **bitwarden** namespace. I find [this](https://medium.com/faun/mount-ssl-certificates-in-kubernetes-pod-with-secret-8aca220896e6) guide useful for adding SSL certificates as Kubernetes secrets.

## Usage

Finally, run the following command to apply the manifests:

`kubectl apply -f .`

## Credits

- [icicimov's](https://github.com/icicimov/kubernetes-bitwarden_rs) repo, although it appears to have been outdated, for the inspiration.

## License

This work is licensed under [GNU GPL v3](./LICENSE).
