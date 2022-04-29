# rawfile-csi

CSI driver to provision local PVs backed by imaged files in kubernetes cluster.

## Usage

On all Kubernetes's nodes, create directories that will be used to
provision image files for local PVs.

For example,

``` bash
$ mkdir -p /mnt/disk_a1 /mnt/disk_b1 /mnt/disk_c1

# (Optional) mount disks to the directories created.
$ mount /dev/sda1 /mnt/disk_a1
$ mount /dev/sdb1 /mnt/disk_b1
$ mount /dev/sdc1 /mnt/disk_c1
```

Now it is possible to create a StorageClass that will provision image file backed
PVs in the directories created.

``` yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: ymatrix-rawfile
provisioner: rawfile.csi.ymatrix.cn
reclaimPolicy: Delete
volumeBindingMode: WaitForFirstConsumer
parameters:
  dataPaths: "/mnt/disk_a1:/mnt/disk_b1:/mnt/disk_c1"
  allocatePolicy: roundrobin
  fsType: xfs
allowVolumeExpansion: false # Current release doesn't support expansion yet.
```

When creating the StorageClass, the following parameters are available.

**dataPaths**:

Specify the directories in the nodes of the Kubernetes cluster where
image files will be created. The paths should be joined with `:` as an array.

**allocatePolicy**:

Decide in which directory an image file will be created to back local PVs.

- **sequential** (default)

  Create image file in the first directory that has enough free space
  according to PV's size in `dataPaths` array on one node. 

- **random**

  Create image file randomly in directories that have enough free space
  according to PV's size in `dataPaths` array on one node.

- **roundrobin**

  Create image file in the next directory that has enough free space according
  to PV's size in `dataPaths` array since last PV image file creation on one node.
  It will go back to the first directory in the `dataPaths` array if it reaches the end of it.

**fsType**:

File system type to be created on the PV.

- **ext4** (default)
- **xfs**
- **btrfs**
