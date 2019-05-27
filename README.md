# Custom Leap for DigitalOcean
__This is a Packer template for automatically creating custom OS images with openSUSE Leap 15.1, suitable for uploading to the DigitalOcean cloud.__

To create such images, you need the following:

* A modern Linux host with QEMU and libvirt
* Packer from HashiCorp (get the latest Linux 64bit version from the [Download Page](https://www.packer.io/downloads.html))
* Ansible (see the [Installation Guide](https://docs.ansible.com/ansible/latest/installation_guide/intro_installation.html))
* An account on [DigitalOcean](https://www.digitalocean.com)

### How to create a new image

Fire up your favorite terminal application, clone this GitHub repository and change into the corresponding directory:

	git clone https://github.com/colder-is-better/digitalocean-custom-images.git
	cd digitalocean-custom-images

Delete all files in directory `provisioners/ssh_keys` and then create one or more new `*.pub` files. Each one of the new `*.pub` files should contain a single public SSH key you own. All those keys will be included in the `authorized_keys` file of user root, in the custom image you are about to create. Change into the `definitions` directory and start a new Packer build:

	PACKER_CACHE_DIR=/tmp/packer_cache ~/bin/packer build leap151do.json

To the `PACKER_CACHE_DIR` variable, be careful to assign a full path to an existing directory: this is where Packer will be saving ISO images into. From the example above, it is apparent that we keep the `packer` executable in the `bin` directory of our user's home. You may of course have `packer` under a different directory, so adjust your command invocation accordingly.

When Packer finishes preparing the image (the build may take some time, so be patient), it leaves us with a compressed QCOW2 file: that would be `openSUSE-Leap-15.1.qcow2.gz`, under `/tmp/packer_out`. This is the file you should upload to your account in DigitalOcean, and keep it there as a _custom image_. Off of that image you may create any number of droplets of any size. More details on how to upload custom images and spawn droplets based on them, are available in the [official documentation](https://www.digitalocean.com/docs/images/custom-images/how-to).

### Things you should have in mind

* Right after the Packer build we invoke a simple Ansible provisioner, and with a quick look in `provisioners/leap151do.playbook.yml` you can immediately see our configuration choices.
* The Leap 15.1 system we build has no plain user account.
* The __default root password__ is `topsecret`, but remote SSH logins to the root account are possible __only via public key authentication__. So besides logging into the system via the web console available in your DigitalOcean account, remote SSH logins are only possible from user accounts with private keys corresponding to the public keys we keep in the `provisioners/ssh_keys` directory.
* According to the [official documentation](https://www.digitalocean.com/docs/images/custom-images/how-to), custom images we upload to our DigitalOcean account should be equipped with [cloud-init](https://cloud-init.io). For simplicity's sake, though, we chose __not__ to install cloud-init in our openSUSE Leap 15.1 custom image.
* Last but not least, please be aware that custom images kept in your DigitalOcean account __incur a fee__ -- specifically __$0.05 per GB per month__. This is certainly not much, but you shouldn't necessarily be paying for keeping your images on the DigitalOcean cloud: just upload your custom image, spawn one or two VMs based on that, then delete it.
