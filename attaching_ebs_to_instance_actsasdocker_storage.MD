To add a new disk to a Docker host that has run out of space, mount the disk to the Docker directory (/var/lib/docker) and migrate existing data as follows:

```
1)create a temporary mount /mnt/docker-container \
2)create format the new ebs as sudo mkfs -t ext4 /dev/nvme1n1 \
3)now mount this file system /dev/nvme1n1 to created mountpoint /mnt/docker-container \
4)stop the docker service \
5)now transfer the /var/lib/docker --in root volume to created mountpoint /mnt/docker-container: 
sudo rsync -aHAX /var/lib/docker /mnt/docker-container \
6)once the existing docker data is transfered, now unmount this with file system:
sudo umount /mnt/docker-container \
7)rename the /var/lib/docker in root volume to /var/lib/docker.old \
8)also create another /var/lib/docker in root volume ec2-user \
9)Now, update the /etc/fstab :
echo '/dev/sdb /var/lib/docker ext4 defaults 0 2' | sudo tee -a /etc/fstab \
10)update the mount now so that the filesystem /dev/nvme1n1 with /var/lib/docker which is in ebs volume now
sudo mount -a \
11)can delete old file existing data ---> /var/lib/docker.old
now check using lsblk ---> /dev/nvme1n1 with /var/lib/docker(it is in root volume but mounted with ebs storage)
```



Steps to Mount a New Disk to /var/lib/docker and Migrate Data:

1) Add and Prepare the New Disk:

Attach the new disk to the Docker host. On a cloud provider, follow their specific steps for attaching additional storage.
Identify the new disk using the lsblk or fdisk -l command. The new disk should appear as something like /dev/sdb or /dev/xvdf.
Create a partition and format it, for example, as ext4:

```
sudo mkfs.ext4 /dev/sdb
sudo mkfs -t ext4 /dev/nvme1n1
```

2) Mount the Disk:

Create a temporary mount point:
```
sudo mkdir /mnt/new-disk
```


3) Mount the new disk:

```
sudo mount /dev/sdb /mnt/new-disk
```


4) Stop Docker Service:

Temporarily stop Docker to ensure no writes occur during migration:

```
sudo systemctl stop docker
```


5) Copy Existing Docker Data to the New Disk:

Use rsync to copy data while preserving file permissions:

```
sudo rsync -aHAX /var/lib/docker/ /mnt/new-disk
```
This may take some time, depending on the size of the data.
Update /etc/fstab to Persist the New Mount:


6) Unmount the temporary mount point:

```
sudo umount /mnt/new-disk
```


7) Mount the new disk directly to /var/lib/docker. First, rename the current Docker directory:

```
sudo mv /var/lib/docker /var/lib/docker.old
```


8) Create a new mount point:

```
sudo mkdir /var/lib/docker
```
Update /etc/fstab to mount the new disk at /var/lib/docker on boot:

```
echo '/dev/sdb /var/lib/docker ext4 defaults 0 2' | sudo tee -a /etc/fstab
```


9) Mount the new disk:

```
sudo mount -a
```


10) Start Docker Service:

Restart the Docker service to confirm it recognizes the new disk:

```
sudo systemctl start docker
```


11) Verify and Clean Up:

Check if Docker is running correctly and that the data was successfully transferred.


12) After confirming everything works, you can delete the backup at /var/lib/docker.old:

```
sudo rm -rf /var/lib/docker.old
```
This process safely migrates the Docker directory to a new disk and ensures Docker has additional storage capacity.