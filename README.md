# iBanks NVIDIA powered GPU cluster

## Create an account at Caltech T2 / iBanks GPU cluster

Either your postdoc contact at Caltech or the T2 admins (t2admin@hep.caltech.edu) can create an account for you. Send the following info to them:

* your full name and institutional contact email
* your CERN lxplus account name. If you are not a CERN user, give the access.caltech or other established institutional account name.
* an SSH public key
* initial account expiry date, which can be extended upon need (max 4 years from now for Caltech internal, max 1 year for external)
* an X509 grid certificate distinguished name in the form of: /DC=ch/DC=cern/OU=Organic Units/OU=Users/CN=... from CRAB Prerequisites - Grid certificate or http://cilogon.org (search for your institute)

The account is created by making a pull request to Caltech Puppet Repo (private only for admins), after which it takes about 1h to propagate to the cluster. The teamwork account needs to be created separately by hand.

## Credentials

If you are not familiar on how to create an ssh key, from a remote client (your laptop) run the following command
<pre>
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
</pre>
and place the content of the public key in the ~/.ssh/authorize_keys on any machine (As it is shared file system, same files are available on all machines)

## Nodes and Access

Caltech Tier2 and iBanks GPU Cluster uses share CEPH Shared Filesystem and everyone has it's own home directory on CEPH (default home directory for all users).

All server have a public (regular network) and a private (10G) IP.
SSH key is the only authentication (Administrators use 2FA). Please let the admins (t2admin AT hep.caltech.edu) in case of issues.

Login nodes:
* login-1.hep.caltech.edu and login-2.hep.caltech.edu - can be used to access Caltech Tier2 and GPU Clusters. Be aware that login nodes do not have any GPUs attached. (There is an option to use GPU HTcondor Scheduling, but it is WIP now.)

Worker nodes are a follows, in chronological order of creation
* `culture-plate-sm.hep.caltech.edu` (alias `gpu-ibanks-4.hep.caltech.edu`) is a Supermicro server with 2TB of local SSD, and runs 8 NVidia GeForce GTX 1080
* `imperium-sm.hep.caltech.edu` (alias `gpu-ibanks-3.hep.caltech.edu`) is a Supermicro server with 2TB of local SSD, and runs 8 NVidia GeForce GTX 1080
* `flere-imsaho-sm.hep.caltech.edu` (alias `gpu-ibanks-2.hep.caltech.edu`) is a Supermicro server with 2TB of local SSD, and runs 6 NVidia Titan Xp
* `mawhrin-skel-sm.hep.caltech.edu` (alias `gpu-ibanks-1.hep.caltech.edu`) is a Supermicro server running 2 NVidia GeForce GTX Titan X
 
## Credits

If you are a user of the cluster, we are happy to help make progress.
When producing publication or public presentation, please be so kind as to warn Prof. Spiropulu and Dr. Vlimant, just for accounting purpose.
Please include the following latex acknowledgement for support
```latex
Part of this work was conducted at  "\textit{iBanks}", the AI GPU cluster at Caltech. We acknowledge NVIDIA, SuperMicro  and the Kavli Foundation for their support of "\textit{iBanks}".
```


## Credentials

If you are not familiar on how to create an ssh key, from a remote client (your laptop) run the following command
<pre>
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
</pre>
this should create an ssh key (default id_rsa) and a public key (id_rsa.pub). Send content of the public key (id_rsa.pub) to an administrator.
You can log in to the nodes using
<pre>
ssh -i id_rsa flere-imsaho-sm.hep.caltech.edu
</pre>
or [configure the ssh client](https://www.ssh.com/ssh/config/) to present the key automatically.

### Data Storage

The home directory should be used for software and although there is room, please prevent from putting too much data within your home directory.

The `/data/` volume is mounted on some nodes, not all on SSD. This is the prefered temporary location for data needed for intensive I/O.

The `/imdata/` volume is a ramdisk of 40G with very high throughput, but utilizing the RAM of the machine. Please use this in case of need of very high i/o, but clean the space tightly, as this will use the node memory. There is a 2-day-since-last-access retention policy on it.

The `/mnt/hadoop/` path is the readonly access to the full Caltech Tier2 Hadoop Storage.

The `/storage/group/gpu/shared` path is CEPH volume

#### CERNbox

You can synchronize your cernbox with local directory `/storage/user/$USER/cernbox` by launching
```
/storage/group/gpu/software/gpuservers/scripts/sync-cernbox.sh
```
at least once interactively to setup the password. Then it can be run in the background, **on one node only**, using 
```
screen -S cernbox -d -m /storage/group/gpu/software/gpuservers/scripts/sync-cernbox.sh
```

### Setup

For ipython, the following directory has to be local
<pre>
mkdir /tmp/$USER/ipython -p
ln -s /tmp/$USER/ipython .ipython
</pre>

For cuda, the same applies to
<pre>
mkdir -p /tmp/$USER/cuda/
export CUDA_CACHE_PATH=/tmp/$USER/cuda/      
</pre>
It is recommended to have `export CUDA_CACHE_PATH` in your login file.

To use only a selected GPU, run `nvidia-smi` or `gpustat` to see GPU utilization, then set `export CUDA_VISIBLE_DEVICES=n` to a the index of the GPU you want to use.
In python one can either set the environment variable or use `import setGPU` (gets one device automatically).

## Software

### CVMFS

cvmfs is mounted on the nodes and can be used accordingly.

### Singularity

All the software is provided with singularity images located in `/storage/group/gpu/software/singularity/ibanks/`
Configuration of the images is located at https://github.com/cmscaltech/gpuservers/tree/master/singularity and in `/storage/group/gpu/software/singularity/`

| image | description |
|-------|-------------|
| legacy.simg | A fixed image with the software already installed on ibanks nodes |
| edge.simg | An image with many of the useful libraries, with the latest versions | 

Let admins know of any missing library that can be put in the image. A build service will be setup later.


To start a shell in an cutting edge image
<pre>
/storage/group/gpu/software/gpuservers/singularity/run.sh
</pre>
or to start with a given image
<pre>
/storage/group/gpu/software/gpuservers/singularity/run.sh /storage/group/gpu/software/singularity/ibanks/legacy.simg 
</pre>

#### Building an image
To build an image, first make sure that there are not an existing image that is usable, or extendable for your purpose. There are example of image specifications under the [singularity directory](https://github.com/cmscaltech/gpuservers/tree/master/singularity), to create your `specification.singularity` file
To build the image `myimage.simg` from the spec
<pre>
/storage/group/gpu/software/gpuservers/singularity/build myimage.simg specification.singularity
</pre>
if you make changes to existing image, please provide suggestion via a pull request modifying the specification file.

If you are building on top of an existing image, you can use that image as base and the build time will be greatly reduced. See an example with [building on top the edge image](https://github.com/cmscaltech/gpuservers/blob/master/singularity/over_edge.singularity).

### Tensorflow

Tensorflow is greedy in using GPUs and it is mandatory to use `export CUDA_VISIBLE_DEVICES=n` (where n is the index of a device, or coma separated index) to use only a selected device, if not explicitly controlled within the application.
In python, please use `import setGPU` that selects automatically the next available GPU.

### Jupyter Hub

Work in progress to set this up properly on the cluster.

### Jupyter Notebook

The users can start a jupyter notebook server on each machine using either

<pre>
/storage/group/gpu/software/gpuservers/jupyter/start_S.sh
</pre>
 
 to start a notebook with the latest singularity image. Or 

<pre>
/storage/group/gpu/software/gpuservers/jupyter/start_S.sh /storage/group/gpu/software/singularity/ibanks/legacy.simg
</pre>
in a given image.

To start the notebook in screen directly:
<pre>
screen -S jupyter -d -m /storage/group/gpu/software/gpuservers/jupyter/start_S.sh
</pre>

This will provide back a url to which you can connect, including an authentication token, that changes each time you restart the jupyter server. You should keep this token private, but can also share momentarily to let other people edit your notebooks ; beware anyone with the token is "you".

To list the jupyter notebook already running on the machine, and the url to be used, one can run
<pre>
/storage/group/gpu/software/singularity/run.sh "jupyter notebook list"
</pre>

The port that is assigned to you is your user id, it should be opened automatically, let an admin know if it's not the case.

### MPI

mpi is available within nodes and accross nodes (as long as you have public-key pass-less ssh between nodes). 
To run a program with mpi
<pre>
mpirun --prefix /opt/openmpi-3.1.0 -np 3 nvidia-smi
</pre>

To run a program using singularity with mpi
<pre>
mpirun --prefix /opt/openmpi-3.1.0 -np 3 singularity exec -B /storage --nv /storage/group/gpu/software/singularity/ibanks/edge.simg python3 /storage/group/gpu/software/gpuservers/mpi/mpi4py-examples/03-scatter-gather
</pre>

To run accross nodes, first copy `/storage/group/gpu/software/gpuservers/mpi/mca-params.conf` into the `$HOME/.openmpi/` directory

<pre>
mpirun  --prefix /opt/openmpi-3.1.0 --hostfile /storage/group/gpu/software/gpuservers/mpi/hostfile -np 10 singularity exec -B /storage --nv /storage/group/gpu/software/singularity/ibanks/edge.simg python3 /storage/group/gpu/software/gpuservers/mpi/mpi4py-examples/03-scatter-gather
</pre>
