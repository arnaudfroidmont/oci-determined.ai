# oci-determined.ai
How to launch Determined.ai on OCI OKE

Start by launching an OKE cluster from here: https://github.com/oracle-quickstart/oci-hpc-oke
Make sure you're able to run a NCCl Allreduce test to validate the nodes. 

Download the determined HELM chart: https://docs.determined.ai/latest/_downloads/389266101877e29ab82805a88a6fc4a6/determined-latest.tgz
Edit the values.yaml file with `maxSlotsPerPod: 8`

Install the Helm chart: 
`helm install determined determined` 
with determined the folder including the values.yaml file. 

Find the Public IP address of the Load Balancer with `k get services`

Install the determined CLI: 

`sudo osms unregister`
`sudo yum install python3.8`
`sudo pip3.8 install determined`
`export DET_MASTER=PublicIP:8080`

It is recommended to add `export DET_MASTER=PublicIP:8080` to your `~/.bashrc` file. 

Now you can run this small example. 
`wget https://docs.determined.ai/latest/_downloads/61c6df286ba829cb9730a0407275ce50/mnist_pytorch.tgz`
`tar -xf mnist_pytorch.tgz`
`cd mnist_pytorch`

Edit the `distributed.yaml` file with the Right number of ressources (GPUs) as well as the right environment variables: 
For GPU4.8
```
environment:
  environment_variables:
    - RX_QUEUE_LEN=8192
    - NCCL_ALGO=Ring
    - NCCL_DEBUG=WARN
    - NCCL_IB_SL=0
    - NCCL_IB_TC=41
    - NCCL_IB_QPS_PER_CONNECTION=4
    - UCX_TLS=ud,self,sm
    - NCCL_IB_HCA==mlx5_0,mlx5_2,mlx5_6,mlx5_8,mlx5_10,mlx5_12,mlx5_14,mlx5_16,mlx5_1,mlx5_3,mlx5_7,mlx5_9,mlx5_11,mlx5_13,mlx5_15,mlx5_17
    - NCCL_IGNORE_CPU_AFFINITY=1
    - NCCL_IB_GID_INDEX=3
    - UCX_NET_DEVICES=mlx5_4:1
resources:
  slots_per_trial: 16
```
For GPU.B4.8 and GPU.A100-v2.8
```
environment:
  environment_variables:
    - RX_QUEUE_LEN=8192
    - NCCL_ALGO=Ring
    - NCCL_DEBUG=WARN
    - NCCL_IB_SL=0
    - NCCL_IB_TC=41
    - NCCL_IB_QPS_PER_CONNECTION=4
    - UCX_TLS=ud,self,sm
    - NCCL_IB_HCA==mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_1,mlx5_2,mlx5_3,mlx5_4,mlx5_14,mlx5_15,mlx5_16,mlx5_17,mlx5_9,mlx5_10,mlx5_11,mlx5_12
    - NCCL_IGNORE_CPU_AFFINITY=1
    - NCCL_IB_GID_INDEX=3
    - UCX_NET_DEVICES=mlx5_0:1
resources:
  slots_per_trial: 16
```
For BM.GPU.H100.8
```
environment:
  environment_variables:
    - NCCL_CROSS_NIC=2
    - NCCL_DEBUG=WARN
    - NCCL_CUMEM_ENABLE=0
    - NCCL_IB_SL=0
    - NCCL_IB_TC=41
    - NCCL_IB_SPLIT_DATA_ON_QPS=0
    - NCCL_IB_QPS_PER_CONNECTION=16
    - NCCL_IB_TIMEOUT=22
    - NCCL_NET_PLUGIN=none
    - HCOLL_ENABLE_MCAST_ALL=0
    - UCX_TLS=tcp
    - NCCL_IB_HCA==mlx5_0,mlx5_1,mlx5_3,mlx5_4,mlx5_5,mlx5_6,mlx5_7,mlx5_8,mlx5_9,mlx5_10,mlx5_12,mlx5_13,mlx5_14,mlx5_15,mlx5_16,mlx5_17
    - NCCL_IGNORE_CPU_AFFINITY=1
    - NCCL_IB_GID_INDEX=3
    - NCCL_SOCKET_IFNAME=eth0
    - RX_QUEUE_LEN=8192
    - IB_RX_QUEUE_LEN=8192
    - NCCL_BUFFSIZE=16777216
resources:
  slots_per_trial: 16
```
You can then run your example with (Don't forget the `.`)

`det experiment create distributed.yaml .`

You can extend the number of Epoch to 100 and compare the difference with and without the environment variable to take advantage of the network. 
