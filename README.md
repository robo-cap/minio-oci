# About

The only difference from [main repository](https://github.com/minio/minio) is the ability to set `MINIO_GW_REGION` environment variable. 

Using [OCI S3 Compatibility API](https://docs.oracle.com/en-us/iaas/Content/Object/Tasks/s3compatibleapi.htm) we can use MINIO as a storage gateway for the buckets deployed in OCI.

By default, MINIO running in S3 gateway mode is able to access only buckets in home region.

When `MINIO_GW_REGION` environment variable is configured, users can access buckets created in other OCI regions. (The region within OCI S3 compatible endpoint URL should match region defined using MINIO_GW_REGION environmnet variable)

## Prerequisite

- [go](https://go.dev/doc/install) >= 1.16.

## Installation

Run the following command to run the latest stable image of MinIO as a container using an ephemeral data volume:

```sh
git clone https://github.com/robo-cap/minio-oci
go install -v
ls ~/go/bin/minio
```

> Note: MinIO strongly recommends *against* using compiled-from-source MinIO servers for production environments.

## Generate Access and Secret Keys in OCI console

Navigate to User Profile settings using the icon on top right in OCI console.

[[/images/user_profile.png]]

Under *Customer Secret Keys* section, click on *Generate Secret Key*. Save the Secret key from the displayed popup and Access key from the table with user secret keys (second column).

[[/images/secret_keys.png]]

## Deployment

```
export MINIO_ROOT_USER=<access_key>
export MINIO_ROOT_PASSWORD=<secret_key>
export MINIO_GW_REGION=<oci_region> (e.g. eu-frankfurt-1)
minio gateway s3 https://<tenancy_namespace>.compat.objectstorage.<oci_region>.oraclecloud.com --console-address ":9001"
```

> Note: In this configuration, one MINIO S3 gateway deployment can access only buckets within one region.

## Deployment Recommendations

### IAM

Note that user credentials and policies are ephemeral. For persistent IAM you need to setup (etcd)[https://github.com/minio/minio/blob/master/docs/sts/etcd.md]

### User policy example

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:*"
            ],
            "Resource": [
                "arn:aws:s3:::default-bucket"
            ]
        },
        {
            "Effect": "Allow",
            "Action": [
                "s3:DeleteObject",
                "s3:GetObject",
                "s3:ListBucket",
                "s3:PutObject"
            ],
            "Resource": [
                "arn:aws:s3:::default-bucket",
                "arn:aws:s3:::default-bucket/*"
            ]
        }
    ]
}
```

### Allow port access for Firewalls

By default MinIO uses the port 9000 to listen for incoming connections. If your platform blocks the port by default, you may need to enable access to the port.

### ufw

For hosts with ufw enabled (Debian based distros), you can use `ufw` command to allow traffic to specific ports. Use below command to allow access to port 9000

```sh
ufw allow 9000
```

Below command enables all incoming traffic to ports ranging from 9000 to 9010.

```sh
ufw allow 9000:9010/tcp
```

### firewall-cmd

For hosts with firewall-cmd enabled (CentOS), you can use `firewall-cmd` command to allow traffic to specific ports. Use below commands to allow access to port 9000

```sh
firewall-cmd --get-active-zones
```

This command gets the active zone(s). Now, apply port rules to the relevant zones returned above. For example if the zone is `public`, use

```sh
firewall-cmd --zone=public --add-port=9000/tcp --permanent
```

Note that `permanent` makes sure the rules are persistent across firewall start, restart or reload. Finally reload the firewall for changes to take effect.

```sh
firewall-cmd --reload
```

### iptables

For hosts with iptables enabled (RHEL, CentOS, etc), you can use `iptables` command to enable all traffic coming to specific ports. Use below command to allow
access to port 9000

```sh
iptables -A INPUT -p tcp --dport 9000 -j ACCEPT
service iptables restart
```

Below command enables all incoming traffic to ports ranging from 9000 to 9010.

```sh
iptables -A INPUT -p tcp --dport 9000:9010 -j ACCEPT
service iptables restart
```

## Test MinIO Connectivity

### Test using MinIO Console

MinIO Server comes with an embedded web based object browser. Point your web browser to <http://127.0.0.1:9000> to ensure your server has started successfully.

> NOTE: MinIO runs console on random port by default if you wish choose a specific port use `--console-address` to pick a specific interface and port.

## Contribute to MinIO Project

Please follow MinIO [Contributor's Guide](https://github.com/minio/minio/blob/master/CONTRIBUTING.md)

## License

- MinIO source is licensed under the GNU AGPLv3 license that can be found in the [LICENSE](https://github.com/minio/minio/blob/master/LICENSE) file.
- MinIO [Documentation](https://github.com/minio/minio/tree/master/docs) © 2021 by MinIO, Inc is licensed under [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/).
- [License Compliance](https://github.com/minio/minio/blob/master/COMPLIANCE.md)
