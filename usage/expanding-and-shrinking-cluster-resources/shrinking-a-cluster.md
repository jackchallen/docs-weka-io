---
description: >-
  This page describes the procedures involved in the shrinking of a cluster,
  which may be required when it is necessary to reallocate cluster hardware.
---

# Shrink a Cluster

## Shrink cluster options

Cluster shrinking can involve either the removal of some of the assigned SSDs or the removal of hosts from the system. The following operations are available:

1. List all the drives and their states, to receive a view of currently-allocated resources and their status.
2. Deactivate drives as the first step before removing a host.
3. Remove a subset of SSD drives allocated for the cluster.
4. Deactivate hosts, which can be used after deactivating drives in preparation for the removal of the host.
5. Remove hosts to complete the cluster shrinking.

## List drives and their sees

**Command:** `weka cluster drive`

Use this command to display a list of all the drives in the cluster and their status.

## Deactivate a drive

**Command:** `weka cluster drive deactivate`

Running this deactivation command will redistribute the stored data on the remaining drives and can be performed on multiple drives.

{% hint style="info" %}
**Note:** After running this command, the deactivated drives will still appear in the list.

It is not possible to deactivate a drive if it will lead to an unstable state, i.e., if the [system capacity](../../overview/ssd-capacity-management.md) after drive deactivation is insufficient for the SSD capacity of currently-provisioned filesystems.
{% endhint %}

Drive deactivation starts an asynchronous process known as phasing out, which is a gradual redistribution of the data between the remaining drives in the system. On completion, the phased-out drive is in an inactive state, i.e., not in use by the Weka system, but still appearing in the list of drives.

{% hint style="info" %}
**Note:** Running the `weka cluster drive` command will display whether the redistribution is still being performed.
{% endhint %}

To deactivate a drive, run the following command:

`weka cluster drive deactivate <uuids>`

**Parameters**

| **Name** | **Type**                | **Value**                         | **Limitations** | **Mandatory** | **Default** |
| -------- | ----------------------- | --------------------------------- | --------------- | ------------- | ----------- |
| `uuids`  | Comma-separated strings | Comma-separated drive identifiers |                 | Yes           |             |

## Remove a drive

**Command:** `weka cluster drive remove`

This command is used to completely remove a drive from the cluster. After removal, the drive will not be recoverable.

To remove a drive, run the following command:

`weka cluster drive remove <uuids>`

**Parameters**

| **Name** | **Type**                | **Value**                         | **Limitations** | **Mandatory** | **Default** |
| -------- | ----------------------- | --------------------------------- | --------------- | ------------- | ----------- |
| `uuids`  | Comma-separated strings | Comma-separated drive identifiers |                 | Yes           |             |

## Deactivate an entire host

**Command:** `weka cluster host deactivate`

This command is used as the first step when seeking to shrink a cluster. Running this command will automatically [deactivate all the host's drives](shrinking-a-cluster.md#deactivating-a-drive).

To deactivate an entire host, run the following command:

`weka cluster host deactivate <host-ids> [--allow-unavailable]`

**Parameters**

| **Name**            | **Type**                 | **Value**                                 | **Limitations**                                                  | **Mandatory** | **Default** |
| ------------------- | ------------------------ | ----------------------------------------- | ---------------------------------------------------------------- | ------------- | ----------- |
| `host-ids`          | Space-separated integers | Space-separated host identifiers          |                                                                  | Yes           |             |
| `allow-unavailable` | Boolean                  | Allow deactivation of an unavailable host | If the host returns, it will join the cluster in an active state | No            | No          |

## Remove a host

**Command:** `weka cluster host remove`

Running this command will eliminate the host from the cluster, i.e., the host will switch to the [stem mode](../../overview/glossary.md#stem-mode) after the removal, at which point it can be reallocated either to another cluster or purpose.

To remove the host from the cluster, run the following command:

`weka cluster host remove <host-id>`

**Parameters**

| **Name**  | **Type**                | **Value**                        | **Limitations** | **Mandatory** | **Default** |
| --------- | ----------------------- | -------------------------------- | --------------- | ------------- | ----------- |
| `host-id` | Comma-separated strings | Comma-separated host identifiers |                 | Yes           |             |
