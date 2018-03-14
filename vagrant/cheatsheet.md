# Vagrant

- `vagrant init your_os` is used to create vagrant box. `your_os` is will be the OS for the box and this is optional.
If you leave it empty (`vagrant init`) then the OS will be set to default by vagrant. You can check the `Vagrantfile`
for more explanation that generated automatically after you execute `vagrant init`.

- `vagrant box add` is pretty similar command to `vagrant init`. The difference is this command will not create `Vagrantfile`.
- `vagrant up` is used to make the box up and running.
- `vagrant ssh` is used to make SSH connection to the box.
- `vagrant destroy` is used to terminate the virtual machine.
