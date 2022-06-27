# Containers

> This is just an experiment, docker do all this and more in a much easier way.

```bash
mkdir cocacola
cd cocacola
chroot . bash
```

Bash does't exist on that context.

```bash
mkdir bin
cp /bin/bash ./bin
chroot . bash
```

Bash require libs to work

```bash
ldd /bin/bash
```

The dependecies will be listed

```bash
mkdir lib{,64} # Create two folder: lib and lib64
cp /lib/x86_64-linux-gnu/libtinfo.so.6 /lib/x86_64-linux-gnu/libdl.so.2 /lib/x86_64-linux-gnu/libc.so.6 ./lib
cp /lib64/ld-linux-x86-64.so.2 ./lib64
```

Now with all settled

```bash
chroot . bash
```

But there's no command other than the ones built in the linux terminal (ex: `pwd`, `cd`)

```bash
ls
# bash: ls: command not found
```

To use the command in the container, it's necessary to add it into the container like bash

```bash
cp /bin/ls ./bin
ldd /bin/ls
cp /lib/x86_64-linux-gnu/libselinux.so.1 /lib/x86_64-linux-gnu/libc.so.6 /lib/x86_64-linux-gnu/libpcre2-8.so.0 /lib/x86_64-linux-gnu/libdl.so.2 /lib/x86_64-linux-gnu/libpthread.so.0 ./lib
cp /lib64/ld-linux-x86-64.so.2 ./lib64
```

The container can't see outsite itself, its the root directory is the `cocacola` directory.

The file system is isolated, but it's still possible to see and control processes running in the machine. Wouldn't be cool to pepsi kill coca-cola process.

So we don't have to copy and paste all tools from the machine to the containers, let's install debootstrap and it will do it automatically.

```bash
apt-get update
apt-get install debootstrap -y
```

```bash
rm -rf cocacola # Replacing old root to the one created by debootstrap
debootstrap --variant=minbase bionic ./cocacola
cd cocacola
```

```bash
unshare --mount --uts --ipc --net --pid --fork --user --map-root-user chroot ./ bash
ps aux
# Error, do this: mount -t proc proc/proc
mount -t proc none /proc
mount -t sysfs none /sys
mount -t tmpfs none /tmp
```

Now the container is completely isolated.

To control how much resources a container can use, the cgroups-tools is a great tool. And to visualize it, htop.

```bash
apt-get install cgroup-tools htop -y
```

```bash
cgcreate -g cpu,memory,blkio,devices,freezer:/sandbox # sandbox is the name of the controller, can be anything
# With unshare running in another terminal
ps aux # Get the PID from the cocacola bash (comes right after the unshare proccess)
cgclassify -g cpu,memory,blkio,devices,freezer:sandbox <cocacola_PID>
```

cocacola container is inside the sandbox group.

```bash
cat /sys/fs/cgroup/cpu/sandbox/tasks
# <cocacola_PID>
```

It's also possible to see how much resource sandbox has.

```bash
cat /sys/fs/cgroup/cpu/sandbox/cpu.shares
# 1024 
```

It's possible to limit how much resource a group can get. For example, limit the cpu usage to 5% and limit the memory usage to 80M.

```bash
cgset -r cpu.cfs_period_us=100000 -r cpu.cfs_quota_us=$[ 5000 * $(getconf _NPROCESSORS_ONLN) ] sandbox
cgset -r memory.limit_in_bytes=80M sandbox
```

---

A docker container uses cgroup

Docker images are just zip files ¯\_(ツ)_/¯

```bash
# start a container on background
docker run --rm -dit --name some-container alpine:3.10 sh
# export the container to .tar
docker export -o ./dockercontainer.tar some-container
mkdir some-container-root
tar xf dockercontainer.tar -C some-container-root/
cd some-container-root
```

```bash
unshare --mount --uts --ipc --net --pid --fork --user --map-root-user chroot ./ sh
mount -t proc none /proc
mount -t sysfs none /sys
mount -t tmpfs none /tmp
```

