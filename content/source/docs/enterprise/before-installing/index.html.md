---
layout: "enterprise"
page_title: "Pre-Install Checklist - Before Installing - Terraform Enterprise"
description: |-
  You need to make key architecture decisions and prepare infrastructure and data files before installing Terraform Enterprise.
---

# Terraform Enterprise Pre-Install Checklist

Before installing Terraform Enterprise, you'll need to make several key architecture decisions and prepare some infrastructure and data files. Please make careful note of these requirements, as the installation may not be successful if these requirements are not met.

## Summary

Prepare all of the following before installing:

2. **Choose an operational mode:** Decide how Terraform Enterprise should store its data. This is affected by your choice of deployment method.
3. **Credentials:** Ensure you have a Terraform Enterprise license and a TLS certificate for Terraform Enterprise to use.
4. **Data storage:** Depending on your operational mode, prepare data storage services or a block storage device.
5. **Linux instance:** Prepare a running Linux instance for Terraform Enterprise. This might require additional configuration or software installation, depending on the OS and your operational requirements. [See below](#linux-instance) for further details.

## Operational Mode Decision

Terraform Enterprise can store its state in a few different ways, and you'll
need to decide which works best for your installation. Each option has a
different approach to
[reliability and availability](../system-overview/reliability-availability.html)
and should be selected based on your organization's preferences.

~> **Important:** Make this decision before you begin installation, because some modes have additional preflight requirements.
The operational mode is selected at install time and cannot be changed once Terraform Enterprise is running.

1. _External Services_ - This mode stores the majority of the
   stateful data used by the instance in an external PostgreSQL database and
   an external S3-compatible endpoint, GCP Cloud Storage bucket or Azure blob storage. There is still critical data stored on the instance that must be managed with snapshots. Be sure to
   check the [PostgreSQL Requirements](./postgres-requirements.html) for information that
   needs to be present for Terraform Enterprise to work. This option is best
   for users with expertise managing PostgreSQL or users that have access
   to managed PostgreSQL offerings like the offers from [Amazon](https://aws.amazon.com/rds/), [Azure](https://azure.microsoft.com/en-us/services/postgresql/), or [Google Cloud](https://cloud.google.com/sql/docs/postgres/).

1. *Active/Active* - This mode has the same requirements as External Services, with the additions of an external Redis server, and a fully automated install method. Refer to the [Active/Active](/docs/enterprise/install/active-active.html) page for more information about how to perform this type of install.

1. *Mounted Disk* - This mode stores data in a separate
   directory on the host, with the intention that the directory is
   configured to store its data on an external disk, such as EBS, iSCSI,
   etc. This option is best for users with experience mounting performant
   block storage.

1. *Demo* - This mode stores all data on the instance. The data can be
   backed up with the snapshot mechanism for restore later. This option is best for initial
   installation and testing, and is not recommended or supported for true production use.

The decision you make will be entered during setup.

## Credentials

Ensure you have all of the following credentials.

### License File

To deploy Terraform Enterprise, you must obtain a license file from HashiCorp.

### TLS Certificate and Private Key

Terraform Enterprise requires a TLS certificate and private key in order to operate. This certificate must match Terraform Enterprise's hostname, either by being issued for the FQDN or being a wildcard certificate.

The certificate can be signed by a public or private CA, but it _must_ be trusted by all of the services that Terraform Enterprise is expected to interface with; this includes your VCS provider, any CI systems or other tools that call Terraform Enterprise's API, and any services that Terraform Enterprise workspaces might send notifications to (for example: Slack). Due to these wide-ranging interactions, we recommend using a certificate signed by a public CA.

The key and X.509 certificate should both be PEM (base64) encoded, and should be provided to the installer as text.

~> **Important:** If you use a certificate issued by a private Certificate
   Authority, you must provide the certificate for that CA in the
   `Certificate Authority (CA) Bundle` section of the installation. This allows services
   running within Terraform Enterprise to access each other properly.
   See [Installation: Certificate Authority (CA) Bundle](../install/installer.html#certificate-authority-ca-bundle)
   for more on this.

## Data Storage

Make sure your data storage services or device meet Terraform Enterprise's requirements. These requirements differ based on operational mode:

- _External Services_
    - [PostgreSQL Requirements](./postgres-requirements.html)
    - Any S3-compatible object storage service, GCP Cloud Storage or Azure blob storage meets Terraform Enterprise's object storage requirements. You must create a bucket for Terraform Enterprise to use, and specify that bucket during installation. Depending on your infrastructure provider, you might need to ensure the bucket is in the same region as the Terraform Enterprise instance.
        - In environments without their own storage service, it may be possible to use [Minio](https://minio.io) for object storage. See the [Minio Setup Guide](./minio-setup-guide.html) for details.
    - Optionally: if you already run your own [Vault](https://www.vaultproject.io/) cluster in production, you can configure Terraform Enterprise to use that instead of running its own internal Vault instance. Before installing Terraform Enterprise, follow the instructions in [Externally Managed Vault Configuration](./vault.html).

- *Active/Active*
    - Redis server v5 or v6
    - Redis Cluster is *not* supported.

- *Mounted Disk*
    - [Mounted Disk Requirements](./disk-requirements.html)

-> **Note:** If you are following one of the [reference architectures](./reference-architecture/index.html), refer to it while preparing your data storage services.

## Linux Instance

Terraform Enterprise runs on Linux instances, and you must prepare a running Linux instance for Terraform Enterprise before running the installer. You will start and manage this instance like any other server.

### Operating System Requirements

Terraform Enterprise currently supports running under the following operating systems:

- **Standalone deployment:**

    - Debian 9 - 10
    - Ubuntu 14.04.5 / 16.04 / 18.04 / 20.04
    - Red Hat Enterprise Linux 7.4 - 7.9 / 8.4
    - CentOS 7.4 - 7.9 / 8.4
    - Amazon Linux 2014.03 / 2014.09 / 2015.03 / 2015.09 / 2016.03 / 2016.09 / 2017.03 / 2017.09 / 2018.03 / 2.0
    - Oracle Linux 7.4 - 7.9 / 8.4

### Hardware Requirements

These requirements provide the instance with enough resources to run the
Terraform Enterprise application as well as the Terraform plans and applies.

* At least 10GB of disk space on the root volume
* At least 40GB of disk space for the Docker data directory (defaults to `/var/lib/docker`)
* At least 8GB of system memory
* At least 4 CPU cores

### Network Requirements

Terraform Enterprise is a networked application. Its Linux instance(s) needs to allow several kinds of incoming and outgoing network traffic.

See [Network Requirements](./network-requirements.html) for details.

<a id="software-requirements"></a>

### Software Requirements (Standalone Deployment)

Some operating systems have specific configuration requirements:

- [RedHat Enterprise Linux (RHEL) Requirements](./rhel-requirements.html)
- [CentOS Requirements](./centos-requirements.html)

For other Linux distributions, check Docker compatibility:

* The instance should run a supported version of Docker engine (1.7.1 or later, minimum 17.06.2-ce, maximum 19.03.11). This also requires a 64-bit distribution with a minimum Linux Kernel version of 3.10.
    * Replicated 2.32.0 and above required when running Docker 18+.
    * In Online mode, the installer will install Docker automatically.
    * In Airgapped mode, Docker should be installed before you begin.
* For _RedHat Enterprise_ and _Oracle Linux_, you **must** pre-install Docker as these distributions are [not officially supported by Docker Community Edition](https://docs.docker.com/engine/install/#server).

~> **Important:** We do not recommend running Docker under a 2.x kernel.

### IAM Policies

If you have chosen the [external services operational mode](https://www.terraform.io/docs/enterprise/before-installing/index.html#operational-mode-decision), Terraform Enterprise will need access to an S3-compliant endpoint for object storage. You can grant access to the object storage endpoint by either assigning an AWS instance profile or an equivalent IAM system in non-AWS environments.

#### S3 Policy

At a minimum, Terraform Enterprise requires the following S3 permissions:

```
{
    "Effect": "Allow",
    "Action": [
        "s3:PutObject",
        "s3:ListBucket",
        "s3:GetObject",
        "s3:DeleteObject",
        "s3:GetBucketLocation"
    ],
    "Resource": [
        "<BUCKET_ARN>",
        "<BUCKET_ARN>/*"
    ]
}
```
-> **Note:** The `s3:ListAllMyBuckets` permission is necessary when testing authentication via the Replicated web console. However, the permission is not required for Terraform Enterprise to function and can be removed once the authentication is successfully tested.


#### KMS Policy

At a minimum, Terraform Enterprise will require the following permissions if the objects in the bucket are to be encrypted via resources in AWS's KMS:

```
{
    "Effect": "Allow",
    "Action": [
        "kms:Decrypt",
        "kms:Encrypt",
        "kms:DescribeKey",
        "kms:ReEncrypt*",
        "kms:GenerateDataKey*"
    ],
    "Resource": [
        "<KMS_KEY_ARN>"
    ]
}
```

#### Instance Profile as Default Credentials
You can use Terraform Enterprise's instance profile to provide default credentials to workspaces. Terraform will attempt to use the instance profile to provision resources when you do not set credentials as environment variables. However, this approach presents a few security risks:

1. All workspaces will have the same permissions because they have access to the same instance profile. You cannot selectively allow or deny access to the instance profile for each workspace.
2. Workspaces will share the instance profile with the Terraform Enterprise application. All workspaces within the application will have access to any resources that Terraform Enterprise depends on, such as its S3 bucket, KMS keys, etc.

~> **Important:** If you choose not to use the instance profile for default credentials, we highly recommend that you [restrict build worker metadata access](../system-overview/security-model.html#restrict-terraform-build-worker-metadata-access) to prevent workspaces from accessing the instance profile.

### SELinux

Terraform Enterprise supports SELinux running in enforcing mode when certain requirements are met. These requirements vary depending on the type of Terraform Enterprise installation.

For _External Services_ and _Demo_ installations, you must install the latest version of the `container-selinux` package.

For _Mounted Disk_ installations, you must:

- Install the latest version of the `container-selinux` package.
- Add the `container_file_t` type to the SELinux context for the mounted disk path and its subdirectories. The commands below update the mounted disk path `/opt/tfe` and its subdirectories to use the correct SELinux context.

    ```
    semanage fcontext -a -t container_file_t "/opt/tfe(/.*)?"
    restorecon -R /opt/tfe
    ```
