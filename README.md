# uConsole CM3

![Photo of a uConsole with CM3 core](pic/photo-uconsole-cm3.jpeg)

Code and docs of uConsole with CM3 core.

Here I use ~~mainline linux~~ RPi's downstream fork.

I started from archlinuxarm's linux-aarch64. ~~It looks like RPi3's mainline support is enough mature.~~ I can reuse the infrastructure built by forks at archlinuxarm, focus on tweaking/optimizing linux.

## Current status

- [x] port drivers
- [x] write device tree
- [x] build test
- [x] test on real hardware
    - [x] kernel boots on a RPi 3B
    - [x] kernel boots on uConsole with CM3
    - [x] HDMI(video) works
    - [x] WiFi works
    - [x] Bluetooth works
    - [x] PMU works(mostly)
    - [x] DSI panel works
        - sometimes panel will stay black after boot, reboot to fix
    - [x] Audio works
        - [x] with 3.5mm jack detection
- ~~[ ] trim build config~~
- [ ] setup CI/CD?

I've successfully adapted the uConsole patches to CM3. I've even written a new kernel driver to support automatic amplifier switch, so the speaker will automatically shutdown when 3.5mm jack is used.

Raise issue if you have any problems.

## How to install ArchLinux on uConsole/CM3 from scratch

Please read [the guide in doc(still draft)](doc/how-to-install-archlinux-from-scratch.md).

## QAs

### Do you plan to support more OSes?

They are essentially the same. Only the packaging methods differ. You can build your own kernel with patches and config in `PKGBUILDs/linux-uconsole-cm3-rpi64`.

### Why not create a full disk image?

kinda lazy ;)

_And I need time/investment for other personal projects._

## Notes

### WiFi

Because of the operation voltage(3.3V by default), the wireless module cannot run at its highest speed. But it should be enough.

__THIS IS NOT TESTED__: If you want to experiment with higher speed, you might want to change the jumpers(ZERO Ohm resistors) on the main board. Then you need to modify the device tree overlay to adjust the corresponding voltage.

### 4G/LTE modem

On uConsole with CM3, the official LTE modem will __ALWAYS__ be powered up on boot because the initial pulls of the pins.

### PMU/Power control

When plugged in, the system might not able to fully shutdown itself.

The charging LED is not configured yet. So it's normal that the orange LED is off when power cable plugged in.

The power button is the system power button, that means you can shutdown your uConsole just by pressing the power button.

### DSI panel

Sometimes the screen will stay black. This is because a data transfer timeout. It rarely occurs and can be fixed with a reboot.
