# TAOA Browser VM

by: alex@taoa.io

This repo build the taoa browser shell meant to run in [`v86`](https://github.com/copy/v86/).

### How to build

First build the "custom" `v86` taoa fork with `podman` (or `docker` if you prefer).

```bash
rm -f build/fs.json
rm -f build/fs/*
podman build -t taoa-v86 -f Dockerfile.taoa-v86 .
```

Now build the `9pfs`

```bash
podman run \
       --rm \
       -v $(pwd)/taoa_fs:/taoa_fs \
       -v $(pwd)/build:/build \
       -it taoa-v86 \
       sh -c "/v86/tools/fs2json.py --out /build/fs.json /taoa_fs"

podman run \
       --rm \
       -v $(pwd)/taoa_fs:/taoa_fs \
       -v $(pwd)/build:/build \
       -it taoa-v86 \
       sh -c "/v86/tools/copy-to-sha256.py /taoa_fs/ /build/fs/"
```

Now copy over the important `v86` assets

```bash
podman run --rm \
       -v $(pwd)/build:/build \
       -it taoa-v86 \
       sh -c "
       cp /v86/build/libv86.js /build/libv86.js;
       cp /v86/build/v86.wasm /build/v86.wasm;
       cp /v86/bios/seabios.bin /build/seabios.bin;
       cp /v86/bios/vgabios.bin /build/vgabios.bin;
       "
```

Now build the taoa iso via buildroot

```bash
export J=4

podman build \
       -t taoa-buildroot \
       -f Dockerfile.taoa-buildroot \
       .

podman run \
       --rm \
       -v $(pwd)/build:/build \
       -v $(pwd)/taoa_buildroot:/taoa_buildroot \
       -it taoa-buildroot \
       bash -c "make BR2_EXTERNAL=/taoa_buildroot v86_defconfig && make -j${J} && sudo cp output/images/rootfs.iso9660 /build/taoa.iso"
```

and HUZZAH, you should have all the assets you need to host the shell

you can host the shell via

```bash
python3 -m http.server
```
