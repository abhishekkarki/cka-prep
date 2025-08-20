# Volume Driver Plugins in Docker

## Introduction to Volume Driver Plugins in Docker
In the [storage_in_docker.md](./1_storage_in_docker.md), we discussed storage drivers. Storage drivers help manage storage on images and containers. We also briefly touched upon volumes. We learned that if you want to persist storage, you must create volumes. However, volumes are not handled by storage drivers; instead, they are managed by volume driver plugins.

The default volume driver plugin in Docker is the local plugin. The local volume plugin helps create a volume on the Docker host and stores its data under the `/var/lib/docker/volumes `directory.

There are many other volume driver plugins that allow you to create volumes on third-party solutions such as:
- Azure File Storage
- Convoy
- DigitalOcean Block Storage
- Flocker
- Google Compute Persistent Disks
- Cluster FS
- NetApp
- REX-Ray
- Portworx
- VMware vSphere Storage

These are just a few examples among many available volume drivers.

Some volume drivers support multiple storage providers. For instance, the REX-Ray storage driver can be used to provision storage on:
- AWS Elastic Block Store (EBS)
- Amazon S3
- EMC storage arrays such as Isilon and ScaleIO
- Google Persistent Disk
- OpenStack Cinder

When you run a Docker container, you can choose to use a specific volume driver, such as the REX-Ray EBS driver, to provision a volume from Amazon EBS. This process creates a container and attaches a volume from the AWS Cloud. When the container exits, your data remains safe and persistent in the cloud.