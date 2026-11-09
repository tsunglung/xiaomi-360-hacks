# MJSXJ02CM / IPC009 — autostart fix

On firmware **4.3.7_0417**, the hack services do not start automatically
after boot. Three things go wrong:

1. `/etc/init.d/S49factory` creates `/tmp/factory_mode`, and the stock
   `/etc/init.d/S60service` exits immediately when that file exists.
2. `S49factory` unmounts `/mnt/sdcard` when it finishes, so
   `/mnt/sdcard/manu_test/configure_services.sh` is not available when
   `S60service` runs.
3. `/mnt/data` is not guaranteed to be mounted when `S60service` runs.

## Patch

Replace `/etc/init.d/S60service` with
[`S60service.ipc009-4.3.7`](S60service.ipc009-4.3.7).

The patched script:

- Removes the `/tmp/factory_mode` exit.
- Runs `mount -a`.
- Mounts `/mnt/sdcard` again.
- Creates the `libshbf`/`libshbfev` symlinks that `framegrabber` needs.
- Starts `configure_services.sh` directly.

## Apply

```bash
unsquashfs -f -d rootfs rootfs_new.bin
cp S60service.ipc009-4.3.7 rootfs/etc/init.d/S60service
chmod +x rootfs/etc/init.d/S60service
mksquashfs rootfs rootfs_new_patched.bin -comp xz -noappend
```

Flash the result with the **real file size**. For the included
`rootfs_new.bin` the size is:

```text
6856704 bytes = 0x68A200
```

The length in the main README (`0x674000`) is smaller than the file and
truncates the squashfs. A truncated image panics with:

```text
squashfs: unable to read id index table
Kernel panic - not syncing: VFS: Unable to mount root fs
```

Use:

```text
sf erase 0x260000 0x750000
sf write 0x21000000 0x260000 0x68A200
```

For a rebuilt image, use `wc -c` and convert the size to hexadecimal.
