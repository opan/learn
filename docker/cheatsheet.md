# Docker

#### Docker vs Vagrant

[source](https://www.quora.com/What-is-the-difference-between-Docker-and-Vagrant-When-should-you-use-each-one)

Docker relies on `containerization` while Vagrant utilizes `virtualization`.

With `virtualization`, each virtual machine runs its own entire operating system
inside a simulated hardware environment provided by program called __hypervisor__ running on
the physical hardware.

Pros:
- near-complete separation between virtual machine and the host enables you to have linux virtual machine on
a windows host or vice versa.

Cons:
- you have dedicated a static amount of resources (CPU, RAM, storage) to the virtual machines, and
the hypervisor will eat up a lot of resources (this is usually referred to as overhead)

With `containerization` allows multiple applications to run in isolated partitions of a single linux kernel
running directly on physical hardware.

Pros:
- performance is higher than virtualization since there is no hypervisor overhead, and you are closer to the bare metal.
Also the container simply uses whatever resoucers its needed, period.

Cons:
- Containers use the host machine's kernel.
