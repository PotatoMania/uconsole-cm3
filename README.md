# uConsole CM3

Code and docs of uConsole with CM3 core.

Here I use mainline linux rather than RPi's downstream.

I started from archlinuxarm's linux-aarch64. It looks like RPi3's mainline support is enough mature. I can reuse the infrastructure built by forks at archlinuxarm, focus on tweaking/optimizing linux.

## Current status

- [x] port drivers
- [x] write device tree
- [x] build test
- [ ] test on real hardware
    - [x] kernel boots on a RPi 3B
    - [ ] kernel boots on uConsole with CM3
- [ ] trim build config
- [ ] setup CI/CD?
