# uConsole CM3

Code and docs of uConsole with CM3 core.

Here I use mainline linux rather than RPi's downstream.

I started from archlinuxarm's linux-aarch64. ~~It looks like RPi3's mainline support is enough mature.~~ I can reuse the infrastructure built by forks at archlinuxarm, focus on tweaking/optimizing linux.

## Current status

- [x] port drivers
- [x] write device tree
- [x] build test
- [ ] test on real hardware
    - [x] kernel boots on a RPi 3B
    - [x] kernel boots on uConsole with CM3
    - [x] HDMI(video) works
    - [ ] WiFi works
    - [x] Bluetooth works
    - [x] PMU works(partial)
    - [ ] DSI panel works
    - [ ] Audio works
- [ ] trim build config
- [ ] setup CI/CD?

Raise issue if you have questions.

I've tested mainline(6.2, 6.5) and downstream(6.1, 6.5) kernels. Things become complicated.

Mainline and downstream use different device trees and downstream have them all in one place(because of merge rebase I guess) but the trees from upstream will not work with downstream!

Mainline and downstream uses different drivers for SDHOST(for sdcard) and SDHCI(the general one, often used with SDIO chips). Mainline driver won't work on downstream. Take care of the configurations.

### WiFi

I cannot get WiFi working with mainline linux, and I don't know why. WiFi on RPi 3B won't work with mainline as well. So I assume it's a driver issue related to the sdhci interface.

### Bluetooth

Bluetooth somehow works with necessary device tree node properties. It works on both kernels. There will be some warnings though.

### 4G/LTE modem

On uConsole with CM3, the LTE modem will __ALWAYS__ be powered up on boot because the initial pulls of the pins.

### PMU/Power control

With unmodified driver, power won't be cut off after powered off. Charging LED is not configured(GPIO0 on PMU). Button shutdown requires driver for AXP20X PEK enabled.

### DSI panel

It's a mystery for me. I cannot get it work.
