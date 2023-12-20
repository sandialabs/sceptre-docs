# minimega

minimega is a tool that launches and manages virtual machines across one or more compute nodes.

- Sandia-developed and [open-source](https://www.sandia.gov/minimega/).
- SCEPTRE uses minimega as the deployment service to orchestrate virtual machines for an experiment.
- Under the hood, minimega wraps the [QEMU](glossary.md#acronyms) hypervisor to boot virtual machines on a compute node.

# Mesh
After following the [installation](quick-start.md#installation) guide, minimega should be installed and running on the headnode. Using the `deploy` api, minimega can copy itself to the compute nodes in the cluster, launch itself, and discover the other cluster members to form a mesh.

- The `deploy` api requires password-less root SSH logins for each node.

```bash
sudo minimega -e deploy launch node[1-3]
```

The `deploy` command copies the current minimega binary to the compute nodes you specify using `scp`, then launches them with `ssh` using the same set of command line flags as the minimega instance that's running on the headnode. After a minute or so, the other instances of minimega should have located each other and created a communications mesh. You can check the status like this:

```
root$ minimega -e mesh status
host     | mesh size | degree | peers | context | port
headnode | 4         | 10     | 3     | sceptre | 11235

root$ minimega -e mesh list
headnode: headnode
 |--node1
 |--node2
 |--node3
node1
 |--headnode
 |--node2
 |--node3
node2
 |--headnode
 |--node1
 |--node3
node3
 |--headnode
 |--node1
 |--node2
```
The `mesh status` command shows general information about the communications mesh, including &#34;mesh size&#34;, the number of nodes in the mesh. For this example, because it shows a mesh size of 4, we know our entire 4-node cluster is in the mesh.

The `mesh list` command lists each mesh node and the nodes to which it is connected.

# Additional Documentation
minimega is an open-source and well-documented project. For additional documentation, please see the official minimega documentation:

- [https://www.sandia.gov/minimega/](https://www.sandia.gov/minimega/)
