# Custom Image VSI

This repo guides you through creating a custom linux image with Virtual Box and upload to IBM Cloud as a custom image.

## Prerequisites
1. Download [Virtual Box][VirtualBoxURL]
1. Download [Cent OS Image][CENTOS_IMAGE_URL]
1. Download [IBM Cloud CLI][IBM_CLOUD_CLI_URL]
1. Download [QEMU CLI][QEMU_CLI_URL]

## Creating the Disk Image

The example iso we are going to use will be Cent-OS 7.x.

You may use any linux iso as a base image to build whichever custom image you'd like, but there are a few requirements 
that are needed.

* Image does not exceed 100GB
* Contains a single file or volume
* Has cloud-init enabled 
* Kernel arguments (We will go into detail on this later)
    * nomodeset
    * nofb
    * vga=normal
    * console=ttyS0
    
### Launching VM
1. Launch Virtual Box
1. Select __New__ from the top menu and follow the on-screen instructions to create a VM
    > Here you will be asked for basic information, to learn more about using Virtual Box please look at the docs 
   > [here][VIRTUAL_BOX_DOCS_URL]
1. During the VM creation process it will ask you to choose a __Hard Disk file type__, here you
must choose __VHD(Virtual Hard Disk)__
   
1. Now that you've selected the __Hard Disk file type__, the remainder of the settings can be customized or the defaults
1. On the left hand side, select the VM you just created and select __Start__
1. You will then be prompted to attach a __Live CD__(ISO), select desired ISO
1. Your VM will boot, and you can run through the installation process of the OS
***
## Post Install
After you've done the installation in __Virtual Box__, the VM will reboot into the console.
 From here you may login with the credentials you created during your installation.

Please make sure you are able to log into the VM and have root powers.

### Kernel Arguments

You will need to verify/modify the kernel arguments in the grub config file.

The arguments that are needed are:
* ```nomodeset```
* ```nofb```
* ```vga=normal```
* ```console=ttyS0```

### Modifying the grub config
1. Backup ```/etc/default/grub```
1. Edit ```/etc/default/grub``` by adding the required arugments to the ```GRUB_CMDLINE_LINUX``` line
1. Generate a new _grub.cfg_ file
1. Reboot your system

### Check for virtio drivers
1. Ensure that your operating system image has virtio drivers installed, along with any code that is needed by virtio. 
   Virtio network drivers are required to enable networking. Check if virtio drivers are installed in the kernel, by 
   running the following command
   ```
   grep -i virtio /boot/config-$(uname -r)
   ```
   _Verify ```VIRTIO_BLK``` and ```VIRTIO_NET``` are present in the output_
1. Ensure that the drivers are present in the temporary root filesystem by running
    ```shell
    lsinitrd /boot/initramfs-$(uname -r).img | grep virtio
    ```
   _Verify the the virtio blk driver and its dependencies ```virtio.ko```, ```virtio_pci```, and ```virtio_ring``` are present. If the 
   virtio dependencies are not present, you must recover the root filesystem._
   
### Network interafce is set to auto-configure
>This will depend on the base image of your custom image and configuration will be different on each. Here we will go 
over how to achieve this in CentOS

1. List all devices on the machine
   ```shell
   nmcli -d
   ```
   _You should see your NIC device, in this example we will use ```eth0```

2. Verify/Modify the network configuration file to retrieve the IP on machine boot
   ```shell
   vi /etc/sysconf ig/network-scripts/ifcfg-[network_device_name]
   
   # Example
   vi /etc/sysconf ig/network-scripts/ifcfg-eth0
   ```
   _Add ```ONBOOT=yes```_

### Install and configure cloud-init
1. To determine if cloud-init is installed, run the following command: 
   ```shell
   cloud-init --version
   
   # If you do not have cloud-init installed, 
   # please use your package manager to install 
   yum install cloud-init -y
   ```

1. If the datasources_list property exists in /etc/cloud/cloud.cfg, verify that it contains ConfigDrive and NoCloud or 
   remove the datasources_list property entirely.
   
1. In the /etc/cloud/cloud.cfg file, verify that the cloud_final_modules section includes the scripts-vendor module and 
   that it is enabled. By default, Red Hat Enterprise Linux and CentOS do not include the scripts-vendor module that is 
   required to provision instances in the IBM Cloud VPC infrastructure.
   
## Upload image to IBM Cloud Object Storage

You have 2 options for this 
1. You may use [IBM Cloud CLI][IBM_CLOUD_CLI_URL]
   
1. Via the UI on [IBM Cloud Dashboard][IBM_CLOUD_URL]

## Create Custom Image

1. Go to ```Menu > VPC Infrastructure > Compute > Custom Images```
1. Select _Create_ at the top of the screen
1. Follow the on screen instructions for basic information and create the custom image

## Conclusion

Now that we have created the custom image we can select the image during the VSI creation process in the IBM Cloud. You 
will be able to select the custom image under the _Operating System_ section during the VSI creation process.

[IBM_CLOUD_URL]: https://cloud.ibm.com
[VirtualBoxURL]: https://www.virtualbox.org/
[CENTOS_IMAGE_URL]: https://www.centos.org/download/
[IBM_CLOUD_CLI_URL]: https://cloud.ibm.com/docs/cli?topic=cli-getting-started
[VIRTUAL_BOX_DOCS_URL]: https://www.virtualbox.org/wiki/Documentation
[QEMU_CLI_URL]: https://www.qemu.org/download/
