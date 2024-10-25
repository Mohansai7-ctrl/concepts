To add a new disk to a Docker host that has run out of space, mount the disk to the Docker directory (/var/lib/docker) and migrate existing data as follows:


Steps to Mount a New Disk to /var/lib/docker and Migrate Data:
1.Add and Prepare the New Disk:

Attach the new disk to the Docker host. On a cloud provider, follow their specific steps for attaching additional storage.
Identify the new disk using the lsblk or fdisk -l command. The new disk should appear as something like /dev/sdb or /dev/xvdf.
Create a partition and format it, for example, as ext4:

```
sudo mkfs.ext4 /dev/sdb
```

Mount the Disk:

Create a temporary mount point:
```
sudo mkdir /mnt/new-disk
```


2.Mount the new disk:

```
sudo mount /dev/sdb /mnt/new-disk
```


3.Stop Docker Service:

Temporarily stop Docker to ensure no writes occur during migration:

```
sudo systemctl stop docker
```


4.Copy Existing Docker Data to the New Disk:

Use rsync to copy data while preserving file permissions:

```
sudo rsync -aHAX /var/lib/docker/ /mnt/new-disk
```
This may take some time, depending on the size of the data.
Update /etc/fstab to Persist the New Mount:


5-a.Unmount the temporary mount point:

```
sudo umount /mnt/new-disk
```


5-b.Mount the new disk directly to /var/lib/docker. First, rename the current Docker directory:

```
sudo mv /var/lib/docker /var/lib/docker.old
```


6.Create a new mount point:

```
sudo mkdir /var/lib/docker
```
Update /etc/fstab to mount the new disk at /var/lib/docker on boot:

```
echo '/dev/sdb /var/lib/docker ext4 defaults 0 2' | sudo tee -a /etc/fstab
```


7.Mount the new disk:

```
sudo mount -a
```


8.Start Docker Service:

Restart the Docker service to confirm it recognizes the new disk:

```
sudo systemctl start docker
```


9.Verify and Clean Up:

Check if Docker is running correctly and that the data was successfully transferred.


10.After confirming everything works, you can delete the backup at /var/lib/docker.old:

```
sudo rm -rf /var/lib/docker.old
```
This process safely migrates the Docker directory to a new disk and ensures Docker has additional storage capacity.