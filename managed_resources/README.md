<!-- This guide shows how to install and use a new kind of custom resource called Bucket. When a user calls the custom resource API to create a Bucket, Crossplane creates a bucket in AWS S3. -->

# Install support for the managed resource

# Follow these steps to install support for the Bucket managed resource:

    Install the provider
    Save the provider’s credentials as a secret
    Configure the provider to use the secret

After you complete these steps you can use the Bucket managed resource.

# Install the provider
A Crossplane provider installs support for a set of related managed resources. The AWS S3 provider installs support for all the AWS S3 managed resources.
```bash
kubectl apply -f maanged_resources/provider.yaml
```

Check that Crossplane installed the provider:
```bash
✗ kg provider
NAME                                     INSTALLED   HEALTHY   PACKAGE                                                            AGE
crossplane-contrib-provider-aws-s3       True        False     xpkg.crossplane.io/crossplane-contrib/provider-aws-s3:v2.0.0       10s
crossplane-contrib-provider-family-aws   True        True      xpkg.crossplane.io/crossplane-contrib/provider-family-aws:v2.3.0   4m56s
```

Crossplane installed the AWS S3 provider. The provider needs credentials to connect to AWS. Before you can use managed resources, you have to save the provider’s credentials and configure the provider to use them.


## Save the provider’s credentials
The provider needs credentials to create and manage AWS resources. Providers use a Kubernetes secret to connect the credentials to the provider.
I have used the OIDC provider for credentials with a separate role
https://docs.upbound.io/manuals/packages/providers/authentication#create-an-iam-oidc-provider
https://docs.upbound.io/manuals/packages/providers/authentication#aws-irsa 
# Create IAM role
Create an iam role as mentioned in `limited_s3_access.json` and use trust policy to use SA of S3 provider not aws. 

## Create a DeploymentRuntimeConfig
IRSA relies on an annotation on the service account attached to a pod to identify the IAM role to use.

Crossplane uses a DeploymentRuntimeConfig to apply settings to the provider, including the provider service account.

Create a DeploymentRuntimeConfig object to apply a custom annotation to the provider service account:
```bash
✗ k apply -f managed_resources/deployment_runtime_config.yaml
deploymentruntimeconfig.pkg.crossplane.io/irsa-runtimeconfig created
```
`This didn't create SA or updated existing SA. So manually edited SA of S3.`

## update provider
```bash
✗ k apply -f managed_resources/s3_provider.yaml
provider.pkg.crossplane.io/provider-aws-s3 created
```
## Create a ProviderConfig
```bash
$ kubectl apply -f managed_resources/provider_config.yaml
```

# Use the managed resource 
```bash
✗ k create -f managed_resources/bucket.yaml
bucket.s3.aws.m.upbound.io/crossplane-bucket-2bptz created
```

```bash
✗ kg bucket
NAME                      SYNCED   READY   EXTERNAL-NAME             AGE
crossplane-bucket-dcmkf   True     False   crossplane-bucket-dcmkf   9m57s
```