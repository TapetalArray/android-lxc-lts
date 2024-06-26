# Android LXC LTS

LXC LTS ported for Android

# Build with Termux Packages

Clone this repo
```bash
git clone https://github.com/TapetalArray/android-lxc-lts
```

Copy build configuration and patches
```bash
cp ../android-lxc-lts/lxc-lts ./termux-packages/packages
```

Build
```bash
./build-package.sh -i -a aarch64 lxc-lts
```

Build for Termux
```bash
./build-package.sh -I lxc-lts
```

# Install
Install dependencies
```bash
pkg upg
pkg i x11-repo -y
pkg i tsu pulseaudio termux-x11-nightly -y
pkg i ./lxc-lts-****.deb
```

# Usage
You can refer to [this](https://gist.github.com/lateautumn233/939be0528a2cc34af66864bead58e68a) (Chinese)

Start Pulseaudio
```bash
pulseaudio --start \
    --load="module-native-protocol-tcp auth-ip-acl=127.0.0.1 auth-anonymous=1" \
    --exit-idle-time=-1
```

Start Termux X11
```bash
termux-x11 &
```

Mount Cgroup (Optional)
```bash
for cg in blkio cpu cpuacct cpuset devices freezer memory; do
   if [ ! -d "/sys/fs/cgroup/${cg}" ]; then
       sudo mkdir -p "/sys/fs/cgroup/${cg}"
   fi

   if ! sudo mountpoint -q "/sys/fs/cgroup/${cg}"; then
       sudo mount -t cgroup -o "${cg}" cgroup "/sys/fs/cgroup/${cg}" || true
   fi
done
```

Systemd-binfmt
```bash
sudo mount binfmt_misc -t binfmt_misc /proc/sys/fs/binfmt_misc
```

Configure Network
```bash
sed -i 's/lxc\.net\.0\.type = empty/lxc.net.0.type = none/g' $PREFIX/etc/lxc/default.conf
```

Create Container
```bash
sudo lxc-create -n name -t download
```

Start
```bash
sudo lxc-start -n name
sudo lxc-info -n name
LD_PRELOAD= sudo lxc-attach -n name /bin/su -
```

# Config
Write to $PREFIX/var/lib/lxc/****/config
```conf
lxc.cgroup.devices.allow = a *:* rwm
lxc.mount.entry = /data/data/com.termux/files/usr/tmp tmp none bind,optional,create=dir
lxc.mount.entry = /data/data/com.termux/files/usr/tmp/.X11-unix tmp/.X11-unix none bind,ro,optional,create=dir
lxc.mount.entry = /dev/dri dev/dri none bind,optional,create=dir
lxc.mount.entry = /dev/dma_heap dev/dma_heap none bind,optional,create=dir
lxc.mount.entry = /dev/kgsl-3d0 dev/kgsl-3d0 none bind,optional,create=file

# For systemd-binfmt
lxc.mount.entry = /proc/sys/fs/binfmt_misc proc/sys/fs/binfmt_misc none bind,optional,create=dir

# For fuse
lxc.mount.entry = /dev/fuse dev/fuse none bind,optional,create=file
```

# Credits

* [lateautumn233](https://github.com/lateautumn233)
* [termux-packages](https://github.com/termux/termux-packages)
* [termux-packags-lxc](https://github.com/termux/termux-packages/tree/master/root-packages/lxc)
