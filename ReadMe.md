# How to rescue a server after loosing SSH KEYS:

 - Create a server with a key and have it initialize.
 - Stop the server that is missing the key and detach the root volume.
 - Attach the detached root volume from step 2 above to the newly created server
   with key.
 - ssh into the server with key.
 - Run the command `$lsblk` to see the volume attached to the system, and you should see
   an additionally attached volume listed but not mounted.
 - Create a mount point in the server.
```yaml
mkdir -p /data
```
 - Mount the additional volume at the created mount point. Note that volumes created
  from the same `ami` will have the same` uuid`, which will only be mounted once on an instance.
  If you try mounting the device without the `nouuid` option, it will give you an error.
 - You can check this using the following command
```
# for RHEL
sudo blkid 

# for ubuntu
sudo lsblk -o +UUID   
```
 - Use the following command to mount the device if the UUID are not the same.   
```
#Will not mount if created from the same ami
mount /dev/xvdf2 /data   
```
 - Use the below command instead if the volume was created from the same ami.
```
mount -t xfs -o nouuid /dev/xvdf2 /data
```
 - After mounting, cd into the `/data/home/ec2-user/.ssh/`

- Copy or cat the authorized_keys from the home of the user and append it to
    `/data/home/ec2-user/.ssh/authorized_keys`
```
cat ~/.ssh/authorized_keys >> /data/home/ec2-user/.ssh/authorized_keys
```
- After successfully appending the keys, unmount the volume using
```
umount -d /dev/xvdf2
```
- If you list the volumes using `#lsblk`, it should show that it is unmounted.
- Now go back to the aws console and reattach the volume to the server that was missing
 the key. Ensure that the device name is `/dev/sda1` which is the root volume.
- Now start the server and you should now log in using the new key.
