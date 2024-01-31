# Multi Layered Detection Model (MLED) for Error Detection

Multi-Layer Error Detection (MLED) is a layered architecture designed to significantly reduce the Undetected Error
Probability (UEP) in file transfers. This is particularly important for petabyte-scale file transfers, which are often
used for data collected from scientific instruments.

The Multi-Layer Error Detection (MLED) architecture is parameterized by a number of layers *(n)*, and a policy *(P<sub>
i</sub>)* for each layer that describes its operation. The architecture is flexible and allows a different number of
layers over different regions of the network, and different policies to be applied at different layers. Importantly,
error detection schemes used by existing file transfer tools can be expressed in this architecture.
In the MLED architecture, each layer performs error detection between processes at the same layer communicating
logically.

The MLED architecture also includes transmitter/receiver protocols. The transmitter at layer *i* forms a PDU with `data`
in the payload, computes CRC according to *P<sub>i</sub>* and appends it. If it's not the base case of recursion, it
fragments the PDU into *k* fragments according to MTU of *P<sub>i+1</sub>* and sends each fragment to *layer (i+1)*. The
receiver at layer i appends the received fragment to a PDU. If a *layer i's* PDU is reassembled, it checks the CRC. If
the PDU is error-free, its payload is delivered to *layer (i - 1)*, which reassembles all fragments to form its own *
layer (i - 1)* PDU.

## Pre-requisites
There are few steps that needs be done before running the code (runs only on `linux` for now). First, you need to install the following packages -

You'll need to install openssl, libssl-dev, mlocate, and g++. Run below command to install them:
```bash
sudo apt update && sudo apt install -y openssl libssl-dev mlocate g++
```

You have to upgrade your gcc version to 10.0.0 or higher. Run below commands to do so (Press ENTER if pop-up appears):
```bash
sudo add-apt-repository ppa:ubuntu-toolchain-r/test

sudo apt update && sudo apt install g++-10 -y
```

We are done with the pre-requisites. Now, we can move on to the next step.

## Clone & Build
Clone the repository using the following command:
```bash
git clone https://github.com/prateekdceit06/mledcpp.git

cd mledcpp

# IMPORTANT: You'll need to make few changes in utils.cpp file. Open it using 
# your favourite editor and change the IP address in get_ip_address() function 
# to the IP address of the node where you'll run the manager server.
# You'll also need to mention all the nodes IP's in the `scripts/run_experiments.py`
# file. Find a variable named `WORKER_IPS` and replace the ips.

make  # Alternatively you can use make -j4 to build in parallel
```

## Configuration File
This configuration file sets up a sophisticated network with multiple nodes and layers, each with specific roles, communication protocols, and data transfer rules. These are the options that can be configured throught the configuration file:

1. **HeartBeatTimeInterval**: This sets the interval (in milliseconds) at which a heartbeat signal is sent to check if a node in the network is active and responsive. (not used currently)

2. **HeartBeatTimeOut**: The time (in milliseconds) after which a node is considered non-responsive if a heartbeat signal is not received. (not used currently)

3. **RetryTimeInterval**: This could be the interval (in milliseconds) before retrying a connection or a process after failure. (not used currently)

4. **Network**: Specifies the type of network protocol used, such as TCP (Transmission Control Protocol). This option does not support any other network type right now

5. **Routing**: Defines the routing strategy, like 'static', indicating pre-defined network paths rather than dynamic routing. (not used currently)

6. **MaxLayers**: Presumably the maximum number of layers (or levels) in the network or system. (not used currently)

7. **DestinationNodeId** and **SourceNodeId**: Identify the destination and source nodes in the network, for data transfer or communication.

8. **Nodes**: An array of node configurations, each containing:
   - **NodeId**: A unique identifier for the node.
   - **NodeName**: A human-readable name for the node.
   - **NodeIp**: The IP address of the node for data transfer.
   - **NodeStatus**: Indicates if the node is active/up or inactive/down.
   - **NodePort**: The network port the node uses for data transfer.
   - **forwardTo**: A list of nodes to which this node forwards data.
   - **processes**: Details about processes running on the node, including their status, type, and other properties which represents one MLED process.
   - **NodeAckIp** and **NodeAckPort**: IP and port used for acknowledgment signals.
   - **forwardAckTo**: Nodes to which acknowledgment signals are sent.

9. **LayerManagers**: An array representing entities that manage different layers in the network, each containing:
   - **LayerManagerId**: Identifier for the Layer Manager.
   - **NodeId**, **NodeName**, **NodeIp**, **NodeStatus**, **NodePort**: Similar to the nodes' configurations.
   - **CheckMethod** and **CheckParameter**: Mechanisms for checking integrity or status, possibly using a hash method like MD5.
   - **ChunkSize**: The size of data chunks to be transferred on each layer.
   - **WindowSize**: Represents the data transfer window or buffer size.
   - **routingTable**: Specifies routing rules, including source, destination, priority, and next hop node.

## Before running the code
Generate the config file (`config/node_info.json`). Afterward, push your public key (`scripts/mledcpp.pub`) to all nodes mentioned in the config file, including the manager node if used.

1. Copy the public key content to `~/.ssh/authorized_keys` file on all nodes.
2. Create a directory named `node_Node1` in the home directory of Node 1.
3. Change the mode of the private key file (`scripts/mledcpp`) to 600.

Alright, we are almost there.

## Run the code
You will find a `scripts` folder in the repository. It contains all the scripts that you'll need to run the code. You have to provide the executable permissions to all the scripts before running them (including the `ssh_server_script.sh` file that is used by the manager server to push the configurations and executable to all the nodes).

```bash
chmod +x *.sh
chmod +x *.py
```

## Tranferring File

Once you are done setting the permissions, you can run the code using the following command:

```bash

python3 run_experiment.py <window_size> <lowest_layer_chunk_size> <file_name> 

# Example: python3 run_experiment.py 100 20000 1024MB
```

- `<window_size>`: Window size to be used on all layers in the config file.
- `<lowest_layer_chunk_size>`: Chunk size of the lowest layer to be used in the config file. The chunk size for the layer `i` will be double of chunk size at layer `i+1`.
- `<file_name>`: Name of the file to be transferred. The file should be in a folder `node_Node1` on `node1`.

## Extra: Creating a Test File
To create a binary file with random data of a specific size
- Run the following command from the `mledcpp` directory and it will create a file with name `<size in MB>MB` in the `node_Node1` directory. Use this name to transfer this file from client to server.
```
python3 ./scripts/create_test_file.py <size in MB>
```

Replace `<size in MB>` with the desired file size.
