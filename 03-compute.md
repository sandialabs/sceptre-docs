# Adding/Removing a Compute Node

This page describes the steps to add or remove a compute node from an existing SCEPTRE cluster.

__Assumptions:__

> - A SCEPTRE cluster has already been [installed](quick-start.md#installation).
> - For adding: Compute node has the necessary [connections](02-networking.md) and [system packages](quick-start.md#installation#compute-node)
> - For removing: Compute node is still connected to the minimega [mesh](04-minimega.md#mesh) or is accessible via SSH.

# Adding a compute node

- Verify the compute node is successfully connected to the [CTL](02-networking.md#control-network-ctl) and [TDN](02-networking.md#trunked-data-network-tdn) networks.

- Verify the necessary [packages](quick-start.md#installation#compute-node) are installed on the compute node.

- Verify the user on the compute node has `sudo` privileges __*WITHOUT*__ using a password (See NOTE below).

- From the [headnode](01-cluster.md#headnode), connect the new compute node to the minimega [mesh](04-minimega.md#mesh) (in the command below, replace `NEW_COMPUTE` with the hostname of the new compute node and replace `USERNAME` with the username of the new compute node):

    ```bash
sudo minimega -e deploy launch NEW_COMPUTE USERNAME sudo
    ```

- Verify the new compute node is connected to minimega mesh:

    ```bash
sudo minimega -e mesh list
    ```

> __NOTE__: To allow USERNAME to use `sudo` without a password (replace USERNAME with the username of the new compute node), first open the sudoers file using: `sudo visudo`. Next add the following entry for USERNAME to the bottom of the file: `USERNAME ALL=(ALL) NOPASSWD:ALL`. Finally, save and close the file.

# Removing a compute node

- Verify there are no VMs running on the node.

- From the [headnode](01-cluster.md#headnode), remove the compute node from the minimega [mesh](04-minimega.md#mesh) (in the command below, replace `COMPUTE` with the hostname of the compute node to be removed):

    ```bash
sudo minimega -e mesh send COMPUTE quit
    ```
