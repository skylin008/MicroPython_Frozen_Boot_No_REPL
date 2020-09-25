# MicroPython_Frozen_Boot_No_REPL
This repo provides a brief tutorial on how to set up MicroPython to boot and run a single or series of modules that are frozen into the firmware in addition to disabling the REPL.  This can be useful if you have a project that you do not want to allow a user to mod a boot.py or main.py on the file system and have the entire application frozen into the firmware in addition to removing the REPL to keep a user from examining the internals of the firmware.  This is an advanced usage of MicroPython and this repo is designed to help a developer with a workflow to provide some basic security if your application calls for such a feature.

## Clone MicroPython Repo
```bash
https://github.com/micropython/micropython.git
```

## Modify inisetup.py Module
#### SOURCE 
```bash
https://github.com/micropython/micropython/blob/master/ports/esp32/modules/inisetup.py
```
#### MODIFIED SOURCE
```python
import uos
from flashbdev import bdev


def check_bootsec():
    buf = bytearray(bdev.ioctl(5, 0))  # 5 is SEC_SIZE
    bdev.readblocks(0, buf)
    empty = True
    for b in buf:
        if b != 0xFF:
            empty = False
            break
    if empty:
        return True
    fs_corrupted()


def fs_corrupted():
    import time

    while 1:
        print(
            """\
FAT filesystem appears to be corrupted. If you had important data there, you
may want to make a flash snapshot to try to recover it. Otherwise, perform
factory reprogramming of MicroPython firmware (completely erase flash, followed
by firmware programming).
"""
        )
        time.sleep(3)


def setup():
    check_bootsec()
    print("Performing initial setup")
    uos.VfsLfs2.mkfs(bdev)
    vfs = uos.VfsLfs2(bdev)
    uos.mount(vfs, "/")
    return vfs
```

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.

## License
[Apache License, Version 2.0](https://www.apache.org/licenses/LICENSE-2.0)
