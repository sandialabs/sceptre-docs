# Modifying an VM Images (.qc2)

## Modifying an Image Independently

To modify a single VM image on it's own (i.e., outside of a running experiment),
you should use [minimega](glossary.md#terminology) directly. This is also
the preferred method of modifying existing VM images if you need Internet
connectivity.

> __NOTE:__

> This section assumes you have a cluster confgiured similar to the
[Cluster Configuration](quick-start.md) outlined under the Setup section.

Directly from the [headnode](glossary.md#terminology), create a minimega
script for the VM you want to edit. An example script we'll call `modify.mm`
is below.

```bash
clear vm config                           #clear any existing configurations
vm config vcpus 2                         #number of virtual CPUs to allocate
vm config memory 4096                     #amount of memory to allocate
vm config snapshot false                  #false=changes will persist, true=changes will not persist
vm config disk /phenix/images/sceptre.qc2 #the location of the VM image
vm config net arbiter                     #network configuration settings
vm config qemu-append -vga qxl            #comment this line out if modifying Windows VMs
vm launch kvm modify                      #arbitrary name of the VM

vm launch
vm start all
```

For this particular config, minimega is going to launch a VM in non-snapshot
mode (meaning any changes made to the VM will persist) using the `sceptre.qc2`
images, and allocating 2 vCPUs, 4096MB of memory, and a single network
interface attached to the 'arbiter' [VLAN](glossary.md#acronyms). Note that
the `vm config qemu-append -vga qxl` is only required for Linux VMs. You
**MUST** comment this out for modifying Windows VMs.

To launch the VM using minimega, execute the following command. Note, that you
must pass the full path to the .mm script.

`sudo minimega -e read /home/ubuntu/modify.mm`

After the VM is launched, you should be able to see the VM in the minimega web
GUI. To access the minimega web GUI, you must forward port 9001 to your machine
then browse to the port (`localhost:9001`) in a web browser.

You might need to setup a few interfaces or add proxy settings if you need the VM to access the internet. From there, any changes you make to the VM will persist!

## Modifying an Image Deployed in an Experiment

Oftentimes you'll need to make modifications to a VM, but you need the context
of a deployed experiment to test the modifications. For example, if you are
installing new [SCADA](glossary.md#acronyms) software, you likely need to
test the configuration to ensure it's working. The phēnix GUI allows users to
easily configure VM images to boot in snapshot or nonsnapshot.

After an experiment has been created, you can modify a VM as desired and then save a new copy of the backing image. To do this, click on the name of the VM to pull up a menu for the VM. At the bottom, click on the floppy disk icon ![](img/backing_image.png). Enter the name of what you would like to call the new backing image and click "Create". This may take some time to successfully complete. Remember, this creates a new backing image, so you either need to update your topology file to use this new file, or replace the existing file once you have stopped your experiment. 
