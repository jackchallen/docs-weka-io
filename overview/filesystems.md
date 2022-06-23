---
description: >-
  This page describes the three types of entities relevant to data storage in
  the Weka system: filesystems, object stores and filesystem groups.
---

# Filesystems, object stores, and filesystem groups

## About filesystems

A Weka filesystem is similar to a regular on-disk filesystem, with the key difference that it's distributed across all the hosts in the cluster. Consequently, in the Weka system, filesystems are not associated with any physical object and are therefore nothing but a root directory with space limitations.

A total of up to 1024 filesystems are supported, all of which are equally and perfectly balanced on all SSDs and CPU cores assigned to the Weka system. This means that the allocation of a new filesystem, or the resizing of a filesystem, are instant management operations that are performed instantly, and without any constraints.

{% hint style="info" %}
**Note:** A [filesystem group](filesystems.md#about-filesystem-groups) has to be created before creating a filesystem, and every filesystem must belong to one filesystem group.
{% endhint %}

A filesystem must have a defined capacity limit. A filesystem that belongs to a tiered [filesystem group](filesystems.md#about-filesystem-groups) must have a total capacity limit and an SSD capacity limit. The total (base) SSD capacity of all filesystems cannot exceed the total SSD capacity as defined in the total SSD net capacity.&#x20;

### Thin provisioning

While a filesystem must have a minimum allocated SSD size, on top of that, filesystems can have an additional SSD capacity cap. The total can exceed the total SSD net capacity and be dynamically allocated between the different thin-provisioned filesystems.&#x20;

Thin provisioning  can help in various use cases:

* On tiered filesystems, available SSD space will be leveraged for extra performance and released to the object-store once needed by other filesystems.
* When using auto-scaling groups, using thin provisioning can help to automatically expand and shrink the filesystem's SSD capacity for extra performance.&#x20;
* Thin provisioning can be used for the logical separation of projects to filesystems when the administrator doesn't expect all to be fully utilized at the same time. Thin provisioned filesystems can replace the using the same filesystem with different directories with quotas and provide better data separation, management, and access control.

### Filesystem limits

* Up to 6.4 trillion files/directories
* Up to 6.4 billion files in a single directory
* Up to 14 EB (using and object-store) with up to 512 PB on SSD
* UP to 4 PB file size

### Encrypted filesystems

Both data at rest (residing on SSD and object store) and data in transit can be encrypted. This is achieved by enabling the filesystem encryption feature. A decision on whether a filesystem is to be encrypted is made when [creating the filesystem](../fs/managing-filesystems/#adding-a-filesystem).

For proper security, a KMS (Key Management System) must be used when creating encrypted filesystems. See [KMS Management](../fs/kms-management/) for more information about KMS support in the Weka system.

{% hint style="info" %}
**Note:** Setting data encryption (on/off) can only be done when creating a filesystem.
{% endhint %}

### Metadata limitations

In addition to the capacity limitation, each filesystem also has a limitation on the amount of metadata. The system-wide metadata limit is determined by the SSD capacity allocated to the Weka system, as well as the [RAM resources](../install/bare-metal/planning-a-weka-system-installation.md#memory-resource-planning) allocated to the Weka system processes. The Weka system will keep tracking of metadata units in RAM, and if it reaches the RAM limit, it will page these metadata tracking units to the SSD (and alert). This leaves enough time for the administrator to increase system resources, as the system keeps serving IOs with a minimal performance impact.

By default, the metadata limit associated with a filesystem is proportional to the filesystem SSD size. It is possible to override this default by defining a filesystem-specific max-files parameter. The filesystem limit is a logical limit to control the specific filesystem usage, and can always be updated by the administrator when necessary.

The total limits of the metadata for all the filesystems can exceed the total system metadata information that can fit in RAM. For minimal impact, in such a case, the least-recently-used units will be paged to disk, as necessary.

### Metadata calculations

Each metadata unit consumes 4KB of space.

Throughout this documentation, the metadata limitation per filesystem is referred to as a parameter named `max-files` , which describes the number of metadata units, and not the number of files. This parameter encapsulates both the file count and the file sizes, as follows:

* Each file requires two metadata units.
* If a file exceeds 0.5 MB, an additional metadata unit is required.
* For each additional 1 MB over the first MB, an additional metadata unit is required.

The definitions above apply to files residing on SSDs or object stores.

{% hint style="success" %}
**Examples:**

* For a filesystem with potentially 1,000,000,000 files of 64 KB in size,  2,000,000,000 metadata units are required.
* For a filesystem with potentially 1,000,000 files of 128 MB in size, 130,000,000 metadata units are required.
* For a filesystem with 1,000,000 files of 750 KB in size, 3,000,000 metadata units are required.
{% endhint %}

## About object stores

In the Weka system, object stores represent an optional external storage media, ideal for the storage of warm data. Object stores used in tiered Weka system configurations can be cloud-based, located in the same location, or at a remote location. Weka supports object stores for tiering (tiering and local snapshots) and backup (snapshots only), and both can be used for the same filesystem.

Object stores are optimally used when a cost-effective data storage tier is required at a price point that cannot be satisfied by server-based SSDs. An object store definition contains the object store DNS name, bucket identifier, and access credentials. The bucket must be dedicated to the Weka system and must not be accessible by other applications. However, a single object store bucket may serve different filesystems and multiple Weka systems.

Filesystem connectivity to object stores can be used in both the data lifecycle management and Snap to Object features. It is possible to define three object-store buckets for a filesystem, two of them for tiering (out of which only one bucket can be writable) and one for backup only. In such cases, the Weka system will search for relevant data in both the SSD and the readable and writable object stores. This allows a range of use cases, including migration to different object stores, scaling of object store capacity, and increasing the total tiering capacity of filesystems.

The Weka system supports up to three different object store buckets per filesystem. While object stores can be shared between filesystems, when possible, it is recommended to create and attach a separate object store bucket per filesystem.

## About filesystem groups

In the Weka system, filesystems are grouped into up to 8 filesystem groups.

Each filesystem group has tiering control parameters. While tiered filesystems have their own object store, the tiering policy will be the same for each tiered filesystem under the same filesystem group.



**Related topics**

****[data-storage.md](data-storage.md "mention")****

[managing-filesystem-groups](../fs/managing-filesystem-groups/ "mention")

[snap-to-obj](../fs/snap-to-obj/ "mention")

