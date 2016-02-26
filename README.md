quickvm
=======

Quickly clone/start/stop/run commands on/delete a VM.

This script can be used to quickly set up a VM based on a existing disk image,
connect to the new VM via ssh and after you are done, throw the VM away.

This script was inspired by tools like vagrant, docker, etc. These allow you to
quickly and deterministically create virtualized environments for development
or experimentation. Unfortunately both have the "download and run code from
somewhere on the internet" pattern baked deep into them. Unfortunately created
base images for both is kind of bothersome. This script is supposed to bring
some of this agility to libvirt.

## Usage

First create a VM based on a existing base VM. Currently this creates a
completely new VM with the first qcow2 image if the base-vm as a backing store
for the newly created qcow2 image of testvm.

    quickvm clone basevm testvm

and get a shell by ssh-ing into the testvm

    quickvm shell testvm

or just execute a command (again via ssh)

    quickvm shell testvm sudo pacman -S ruby

and then throw away the vm

    quickvm trash testvm

It's also possible to transfer data from and to the VM

    quickvm push testvm ./data /tmp/
    quickvm pull testvm /var/log/whatever.log ./logs/

The default is to use rsync, with a fallback to scp if it isn't installed.


## ssh with different username

Per default quickvm executes ssh with your current username (i.e. `$USER`) you
can change this using the `-u` option (supported by a couple of commands):

    quickvm shell -u root

or change the global default:

    mkdir -p ~/.config/quickvm
    echo "QUICKVM_USER=root" >> ~/.config/quickvm/config.sh

or change the default per VM:

    quickvm setuser myvm root


## Requirements

Basically none except for libvirt and things that come with libvirt:
`virsh, qemu-img, virt-install`


## Feature-Wishlist

  - set internal options via command line parameters
    (this includes more sophisticated cli parsing)
  - File sharing
    - Make it easy to share files via 9p
      (if possible even in the face of SELinux)
  - Template scripts to create VMs
    - Execute a script that sets up a VM out of the blue
  - Provisiong (vagrant style)
    - copy files
    - execute a shell script
    - wget and execute a shell script


## Comparison with other tools

Warning: some ranting ahead.

### why not vagrant?

To be fair vagrant is already pretty close to what I want. Unfortunately it
doesn't play so well with `libvirt`.

  - Doesn't work so well with `libvirt`
    - Most base boxes don't work
    - Throws away VM automatically if initial ssh fails, which is very annoying
      in combination with [this bug.](https://github.com/mitchellh/vagrant/issues/4367)
  - Creating base boxes is kind of annoying. packer works reasonably well.
    Still most of the time I just want to log in and install a package or two
    in my base VM. Using `vagrant package` all the time seems too much work to
    me.
  - `vagrant ssh` works only in the same dir as the `Vagrantfile`. I'd like
    this to work from any dir given an identifier.

### why not docker? or rkt? or lxc?

  - [Containers don't contain.](https://opensource.com/business/14/7/docker-security-selinux)
    and I might want to run completely untrusted software. Load some kernel or
    pam module from the internet or wherever. I like to think the damage is
    somewhat minimal in a VM that is going to be thrown away 10 minutes later.
  - Some things just can't be done inside unprivileged docker containers.
    For example chrooting, which is required on some distros for package
    building.
  - Most container managment software has this app-centric view, while I want a
    interactive environment.
