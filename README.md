# Rescuing an EC2 Server After Losing SSH Keys Using EBS Volume

This guide details the steps to rescue an AWS EC2 instance (`target-instance`) after losing SSH keys using EBS volume concepts. The process involves creating a new instance (`rescue-instance`), transferring the root volume, and updating SSH keys.


## Steps:

### 1.  Create a New EC2 Instance
- Launch a new EC2 instance called `rescue-instance` with the key pair `rescue-key.pem` in the same Availability zone (AZ) as the `targe-instance` or server with the missing or corrupted SSH keys.

### 2. Stop the Target EC2 Instance
- Stop the target instance (`target-instance`) from the EC2 Dashboard.

### 3. Detach the Root EBS Volume from `target-instance`
- Go to the EC2 Dashboard.
- Select the `target-instance` instance.
- Navigate to the **Storage** tab.
- Select the root volume.
- Click on **Actions** and choose **Detach Volume**.

### 4. Attach the Detached Root Volume from `target-instance` to `rescue-instance`
- Go to the **Volumes** section in the EC2 Dashboard.
- Select the detached volume.
- Click on **Actions** and choose **Attach Volume**.
- Attach it to the `rescue-instance` as `/dev/sdf` (or `/dev/xvdf`).

### 5. SSH into the `rescue-instance` server
```sh
ssh -i rescue-key.pem ec2-user@<rescue-instance-public-dns>
```

### 6. Create a Mount Point and Mount the Volume
```sh
sudo mkdir /data
```
#### Note: 
- Mount the additional volume at the created mount point. Volumes created from the same AMI will have the same UUID.Attempting to mount the device without using the `nouuid` option will result in an error. You can verify this using the following command:
```sh
# For RHEL
sudo blkid 

# For Ubuntu
sudo lsblk -o +UUID
```

- Use the following command to mount the device if the UUIDs are not the same:
```sh
# Will not mount if created from the same AMI
sudo mount /dev/xvdf2 /data
```

- Use the below command instead if the volume was created from the same AMI:
```sh
sudo mount -t xfs -o nouuid /dev/xvdf2 /data
```

- After mounting, change directory into `/data/home/ec2-user/.ssh/:`
```sh
cd /data/home/ec2-user/.ssh/
```


### 7. Copy the New Private SSH Key to the `authorized_keys` File
- Copy or cat the authorized keys from the home of the user and append it to `/data/home/ec2-user/.ssh/authorized_keys:`
```sh
cat ~/.ssh/authorized_keys >> /data/home/ec2-user/.ssh/authorized_keys
```

### 8. Unmount the Volume and Detach It
- After successfully appending the keys, unmount the volume using:
- `sudo umount -d /dev/xvdf2`
- If you list the volumes using `lsblk,` it should show that it is unmounted. Now go back to the AWS console and reattach the volume to the server that was missing the key. Ensure that the device name is `/dev/sda1` which is the root volume.

### 9. Reattach the Volume to `target-instance` as the Root Volume

- Attach the volume back to the target-instance instance as /dev/sda1 (or the original device name).

### 10. Start the `target-instance` Instance
- Start the ```sh target-instance``` instance from the EC2 Dashboard.

### 11. SSH into the `target-instance` Instance Using the `rescue-key.pem` key
```sh
ssh -i rescue-key.pem ec2-user@<target-instance-public-dns>
```
