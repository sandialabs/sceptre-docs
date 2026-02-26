# Quick Start Guide

## Installation

The SCEPTRE platform is a combination of [COTS](about/glossary.md#cots) hardware, software, and Sandia-developed tools. Installation can be local (one computer) or distributed (multiple computers).

*For the best performance, install SCEPTRE using the [distributed](#distributed-installation-guide-recommended) installation guide.*

### Prerequisites

- Computer has Ubuntu 20.04 LTS or 22.04 LTS [installed](https://help.ubuntu.com/20.04/installation-guide/amd64/index.html) as the operating system
- Computer has internet access to install packages (or an [apt-mirror](01-cluster.md#apt-mirror) has been configured). If behind a web interception proxy, ensure `apt` is configured to use the proxy.
- All commands should be ran as `root`. Switch to the root user using `sudo su`.
- Install [ORAS](https://oras.land/docs/installation/) to download pre-build VM images.


### *Local Installation Guide (Quickest)*

For a local SCEPTRE installation, a single computer will act as both [headnode](#headnode) and [compute node](#compute-node).

1. Check [Prerequisites](#prerequisites) and ensure you are running as `root` user (`sudo su`).
1. Install required packages. Alternatively, follow [Docker's installation instructions for Ubuntu](https://docs.docker.com/engine/install/ubuntu/).

    ```bash
    apt update
    apt install -y git docker.io docker-compose-plugin
    ```

1. **Optional** If behind a proxy server, configure Docker to use the proxy

    ```bash
    mkdir /etc/systemd/system/docker.service.d/
    cat <<EOF | sudo tee /etc/systemd/system/docker.service.d/http-proxy.conf
    [Service]
    Environment="NO_PROXY=*.example.com"
    Environment="HTTP_PROXY=http://proxy.example.com:8080/"
    Environment="HTTPS_PROXY=https://proxy.example.com:8080/"
    Environment="no_proxy=*.example.com"
    Environment="http_proxy=http://proxy.example.com:8080/"
    Environment="https_proxy=https://proxy.example.com:8080/"
    EOF
    systemctl daemon-reload
    systemctl restart docker
    ```

1. Install topologies and base images

	```bash
    mkdir -p /phenix
    cd /phenix
    git clone https://github.com/sandialabs/sceptre-phenix-topologies.git topologies
    git clone https://github.com/sandialabs/sceptre-phenix-images.git vmdb2
	```

1. Install phēnix source files

	```bash
    mkdir -p /opt
    cd /opt
    git clone https://github.com/sandialabs/sceptre-phenix.git phenix
	```

1. **Optional** Download pre-built VM images. Not all images are required.For a basic deployment of SCEPTRE, `bennu.qc2` is recommended. **NOTE**: This requires [ORAS](https://oras.land/docs/installation/) to be installed and the `oras` command to be available.

    ```bash
    mkdir -p /phenix/images
    cd /phenix/images

    # Download essential images from open-source
    oras pull ghcr.io/sandialabs/sceptre-phenix-images/bennu.qc2:latest
    oras pull ghcr.io/sandialabs/sceptre-phenix-images/vyos.qc2:latest

    # Optional: download other open-source images as-needed
    # Available images are documented in the sceptre-phenix-images README
    # https://github.com/sandialabs/sceptre-phenix-images
    oras pull ghcr.io/sandialabs/sceptre-phenix-images/ntp.qc2:latest
    oras pull ghcr.io/sandialabs/sceptre-phenix-images/kali.qc2:latest
    oras pull ghcr.io/sandialabs/sceptre-phenix-images/bookworm.qc2:latest
    oras pull ghcr.io/sandialabs/sceptre-phenix-images/jammy.qc2:latest
    oras pull ghcr.io/sandialabs/sceptre-phenix-images/noble.qc2:latest
    oras pull ghcr.io/sandialabs/sceptre-phenix-images/docker-hello-world.qc2:latest
    oras pull ghcr.io/sandialabs/sceptre-phenix-images/ubuntu.qc2:latest
    oras pull ghcr.io/sandialabs/sceptre-phenix-images/ubuntu-soaptools.qc2:latest
    oras pull ghcr.io/sandialabs/sceptre-phenix-images/minirouter.qc2:latest
    ```

1. Install docker images

	- Pull pre-built docker containers. Useful for *users* of SCEPTRE.

        ```bash
        docker pull ghcr.io/sandialabs/sceptre-phenix/phenix:main
        docker pull ghcr.io/sandia-minimega/minimega:master
	    ```

	- Alternatively, build the docker containers from source. Useful for *developers* of SCEPTRE.

		```bash
        cd /opt/phenix/docker
        docker compose build
		```

    - **Tip** - If behind a web proxy, you must add `http_proxy` and `https_proxy` build args to the build command (Ex. `--build-arg http_proxy=http://proxy.example.com:8080 --build-arg https_proxy=http://proxy.example.com:8080`). Additionally, the `INSTALL_CERTS` build args may be required for custom certificates.

1. Set the `CONTEXT` environment variable

	```bash
    echo "export CONTEXT=$(hostname)" >> ~/.profile
    source ~/.profile
    ```

1. Start up the SCEPTRE docker containers (minimega and phenix)

    - Method 1: standard compose, that also starts Elasticsearch and Kibana, and builds a phenix image

        ```bash
        cd /opt/phenix/docker
        docker compose up -d
        ```

    - Method 2: only creates phenix and minimega, using the latest images from open-source

        ```bash
        cd /opt/phenix/docker
        docker compose -f no-build_docker-compose.yml up -d
        ```

1. **Optional** Add a few convenience aliases to your shell

	```bash
    cat <<EOF >> ~/.bash_aliases
    alias phenix='docker exec -it phenix phenix'
    alias mm='docker exec -it minimega minimega -e'
    alias mminfo='mm .columns name,state,snapshot,cc_active,id,ip vm info'
    alias mmsum='mm .columns name,state,cc_active,id,uuid vm info'
    alias ovs-vsctl='docker exec -it minimega ovs-vsctl'
    EOF

    source ~/.bash_aliases
	```

1. Access the phēnix web GUI at `<server-ip>:3000` or `localhost:3000`
1. Run phenix command line. **NOTE**: If you didn't configure the aliases above, then replace "phenix" with `docker exec -it phenix phenix`.

    ```bash
    phenix --version
    phenix --help
    ```


### *Distributed Installation Guide (RECOMMENDED)*

A distributed SCEPTRE installation requires one [headnode](01-cluster.md#headnode) computer and one or more [compute nodes](01-cluster.md#compute-node).

**Headnode Install** - The headnode is the computer where experiment management tools are installed. Virtual machines do not run on this machine. For hardware requirements, see [Headnode Requirements](01-cluster.md#software-requirements)

1. Check [Prerequisites](#prerequisites) and ensure you are running as `root` user (`sudo su`).
1. Follow the steps in the [Local Installation Guide](#local-installation-guide-quickest) to deploy SCEPTRE on the headnode.
1. Stop the Docker containers

    ```bash
    cd /opt/phenix/docker
    docker compose stop
    ```


1. Configure NFS share. Setting up a Network File Share allows sharing of the base KVM images across multiple nodes. **Tip** - This is much more efficient than copying large base KVM images to each node individually.

    ```bash
    echo '/phenix/images *(rw,sync,no_subtree_check)' >> /etc/exports
    service nfs-kernel-server restart
    ```

1. Start up the docker containers

    ```bash
    cd /opt/phenix/docker
    docker compose up -d
    ```

1. Access the phēnix web GUI at `<server-ip>:3000` or `localhost:3000`


**"Compute Node" Install** - The compute node is the computer where virtual machines run. For hardware requirements, see [Compute Node Requirements](01-cluster.md#software-requirements_1)

1. Check [Prerequisites](#prerequisites)
1. Install required packages

	```bash
    apt update
    apt install -y nfs-common openvswitch-switch qemu-kvm tmux vim
	```

1. Mount NFS share. Replace `X.X.X.X` with the IP address of the headnode.

    ```bash
    mkdir -p /phenix/images
    echo 'X.X.X.X:/phenix/images /phenix/images nfs auto,rw 0 0' >> /etc/fstab
    mount -a
    ```


## Getting Started - Helloworld Experiment

1. Download the required backing image `ubuntu.qc2`

    ```shell
    cd /phenix/images
    oras pull ghcr.io/sandialabs/sceptre-phenix-images/ubuntu.qc2:latest
    ```

1. Access phenix web

	- The phēnix web interface allows for creating, configuring, and managing SCEPTRE experiments. First open up `http://<Headnode IP address>:3000` in your browser, and you'll see the home page displayed:

		![](img/phenix_home.png)

1. Upload topology

	- You must first upload the topology file for phēnix to ingest. From the home page, click on the `Configs` tab to navigate to the configurations page. Next click the ![](img/upload_button.png) button and drag/drop the `helloworld.yaml` file into the dialog box to upload it:

		![](img/config_page.png)
   		![](img/upload_dialog.png)

	-  Alternatively, you can upload the topology via the CLI using the following command on the headnode:

		```bash
        phenix config create /phenix/topologies/helloworld.yaml
		```

	- You should now see the `helloworld` topology in the configs table:

		![](img/config_list.png)

1. Create Experiment

	- Navigate back to the home page by clicking the `Experiments` tab.
	- Click the new experiment ![](img/create_now.png) button to open the create experiment dialog.
	- Fill out the dialog as shown (leaving everything else blank) and then click the ![](img/create.png) button:

		- Experiment Name: __*my_first_experiment*__
		- Topology: __*helloworld*__

		![](img/phenix_create.png)

	- Alternatively, you can create the experiment via the CLI using the following command on the headnode:

		```bash
        phenix exp create my_first_experiment -t helloworld
		```

1. Deploy Experiment

	- Your newly created experiment will appear in the experiments table:

		![](img/phenix_list.png)

	- To start the experiment, click the ![](img/status.png) button and then click ![](img/start.png):

		![](img/phenix_start.png)

	-  Alternatively, you can deploy the experiment via the CLI using the following command on the headnode:

		```bash
        phenix exp start my_first_experiment
		```

	- Once your experiment starts up, its status will be marked as ![](img/started.png). Click on the name of the experiment ![](img/name.png), and phēnix will switch to the experiment info page:

		![](img/phenix_info.png)

	- **Tip** - Click on the State of Health ![](img/soh_button.png) button to see a network topology map, and click the Go Back ![](img/goback.png) button to return to the Experiment Info page.

		![](img/topo_graph.png)

1. Test
	- Congratulations! You've created and deployed your first SCEPTRE experiment.
	- From here you can interact with individual Virtual Machines (VMs) by clicking on the respective screenshot, which will open a new browser tab for that VM:

		![](img/vnc.png)

	- Login as the `root` user for either of the VMs and trying pinging the other IP address:

		![](img/ping.png)


## Getting Started - SCEPTRE-on-a-Platter (SOAP)

Now that you can run the basic helloworld topology, we are ready to run a topology of a notional ICS. This topology, called SCEPTRE-on-a-Platter (SOAP), models a notional SCADA system for a 300 bus microgrid system. The model uses PyPower to model the power system and Ignition SCADA software for visibility and control.

Instructions for launching the SOAP topology can be found at: [SCEPTRE Phenix Topologies - SOAP](https://github.com/sandialabs/sceptre-phenix-topologies/tree/main/soap)

## Debugging

- Check `phenix.log` and `minimega.log`

    ```bash
    sudo tail -f /var/log/phenix/*.log
    ```

- View container logs

    ```bash
    cd /opt/phenix/docker
    sudo docker compose logs -f
    ```

## Getting Help

To get help with SCEPTRE, open an issue on the relevant GitHub repository, or contact us at `wg-sceptre-core@sandia.gov` or `emulytics@sandia.gov`.
