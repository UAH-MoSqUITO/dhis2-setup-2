# Prepare virtual machine using Vagrant

* Install [Vagrant](https://www.vagrantup.com/)

* Initialize `Vagrantfile`

    ```shell
    vagrant init bento/ubuntu-18.04
    ```

    ```console
    $ vagrant init bento/ubuntu-18.04
    A `Vagrantfile` has been placed in this directory. You are now
    ready to `vagrant up` your first virtual environment! Please read
    the comments in the Vagrantfile as well as documentation on
    `vagrantup.com` for more information on using Vagrant.
    ```

* Start VM

    ```shell
    vagrant up
    ```

    ```console
    $ vagrant up
    Bringing machine 'default' up with 'virtualbox' provider...
    ==> default: Importing base box 'bento/ubuntu-18.04'...
    ==> default: Matching MAC address for NAT networking...
    ==> default: Checking if box 'bento/ubuntu-18.04' version '201812.27.0' is up to date...
    ==> default: Setting the name of the VM: dhis2-setup-2_default_█████████████_█████
    ==> default: Clearing any previously set network interfaces...
    ==> default: Preparing network interfaces based on configuration...
        default: Adapter 1: nat
    ==> default: Forwarding ports...
        default: 22 (guest) => 2222 (host) (adapter 1)
    ==> default: Booting VM...
    ==> default: Waiting for machine to boot. This may take a few minutes...
        default: SSH address: 127.0.0.1:2222
        default: SSH username: vagrant
        default: SSH auth method: private key
        default: Warning: Connection aborted. Retrying...
        default: Warning: Connection reset. Retrying...
        default: Warning: Remote connection disconnect. Retrying...
        default: Warning: Connection aborted. Retrying...
        default:
        default: Vagrant insecure key detected. Vagrant will automatically replace
        default: this with a newly generated keypair for better security.
        default:
        default: Inserting generated public key within guest...
        default: Removing insecure key from the guest if it's present...
        default: Key inserted! Disconnecting and reconnecting using new SSH key...
    ==> default: Machine booted and ready!
    ==> default: Checking for guest additions in VM...
        default: The guest additions on this VM do not match the installed version of
        default: VirtualBox! In most cases this is fine, but in rare cases it can
        default: prevent things such as shared folders from working properly. If you see
        default: shared folder errors, please make sure the guest additions within the
        default: virtual machine match the version of VirtualBox you have installed on
        default: your host and reload your VM.
        default:
        default: Guest Additions Version: 5.2.22
        default: VirtualBox Version: 6.0
    ==> default: Mounting shared folders...
        default: /vagrant => C:/████████/dhis2-setup-2
    ```
