# Vagrant

- `vagrant init your_os` is used to create vagrant box. `your_os` is will be the OS for the box and this is optional.
If you leave it empty (`vagrant init`) then the OS will be set to default by vagrant. You can check the `Vagrantfile`
for more explanation that generated automatically after you execute `vagrant init`.

- `vagrant box add` is pretty similar command to `vagrant init`. The difference is this command will not create `Vagrantfile`. There is subcommand supplied for `vagrant box`.
- `vagrant up` is used to make the box up and running.
- `vagrant ssh` is used to make SSH connection to the box.
- `vagrant destroy` is used to terminate the virtual machine.
- `vagrant halt` is used to shutdown the running machine vagrant is managing.
- `vagrant package` is used to packages current running virtual box into a re-usable box. This command is useful for cloning vagrant box.

## How to clone vagrant box

- run `vagrant package` inside vagrant directory that you want to clone.
- It will output new file called (by default) `package.box`.
- Move the `package.box` outside the vagrant directory so it won't clash (maybe? just in case).
- Then create new vagrant directory and run `vagrant init` inside them.
- Once `Vagrantfile` created, edit the content like below:
```
config.vm.box = "your-package-name"
config.vm.box_url = "the-absolute-path-to-package.box"
```
- If you want to setup `private_network` IP, you can use this [private network](https://en.wikipedia.org/wiki/Private_network#Private_IPv4_address_spaces)
- To access host machine from guest (inside Vagrant), just use the gateway provided with `netstat -rn` command inside vm.

## Sending file to vagrant box

To send file from host machine to vagrant box, we can use `file` provisioning.
To make sure the file provisioning is running we can run `vagrant provision`

## Sync folder between Vagrant and Host

source for below: https://blog.theodo.fr/2017/07/speed-vagrant-synced-folders/
```
config.vm.synced_folder "source/dir", "destination/dir", type: "nfs",
  mount_options: ['rw', 'vers=3', 'tcp'],
  linux__nfs_options: ['rw', 'no_subtree_check', 'all_squash', 'async']
```
