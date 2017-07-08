# Linux

## Template: `cortex-m-quickstart` + `cortex-m-rtfm`

https://docs.rs/cortex-m-quickstart/0.1.5/cortex_m_quickstart

### Workflow

```
# Install Rust (once)
$ curl https://sh.rustup.rs -sSf | sh
$ rustup default nightly

# Install non-Rust embedded tools (once)
$ sudo apt-get install binutils-arm-none-eabi gdb-arm-none-eabi openocd

# Install Xargo and subcommands (once)
$ cargo install cargo-clone [cargo-edit] xargo
$ rustup component add rust-src # for Xargo

# Start from template
$ cargo clone cortex-m-quickstart --vers 0.1.2
$ mv cortex-m-quickstart demo && cd $_
$ edit Cargo.toml # rename project

# Configure template
$ edit memory.x # specify memory layout

# Set default build target
$ cat >>.cargo/config <<'EOF'
[build]
target = "thumbv7em-none-eabihf"
EOF

# Add dependencies as needed
$ cargo add cortex-m-rtfm ..

# Spawn OpenOCD instance (once per development session)
$ openocd -f .. &

# edit-compile-test cycle
$ xargo watch -x check &
$ edit src/main.rs
$ arm-none-eabi-gdb target/.. # test

# deploy
$ xargo build --release
$ arm-none-eabi-gdb target/..
```

### Setup errors

#### Forgot to install the `rust-src` component

Original error message:

```
$ xargo build
error: couldn't walk the sysroot
caused by: IO error for operation on /home/japaric/.rustup/toolchains/nightly-x86_64-unknown-linux-gnu/lib/rustlib/src: No such file or directory (os error 2)
caused by: No such file or directory (os error 2)
note: run with `RUST_BACKTRACE=1` for a backtrace
```

Rating: Not helpful at all.

Improved error message:

```
$ xargo build
error: `rust-src` component not found. Run `rustup component add rust-src`.
note: run with `RUST_BACKTRACE=1` for a backtrace
```

https://github.com/japaric/xargo/pull/136

#### Forgot to install the `arm-none-eabi-ld` linker

Error message:

```
$ xargo build
(..)
   Compiling cortex-m-quickstart v0.1.2 (file:///home/japaric/tmp/demo)
error: could not exec the linker `arm-none-eabi-ld`: No such file or directory (os error 2)
  |
  = note: "arm-none-eabi-ld" "-L" (..)

error: aborting due to previous error

error: Could not compile `demo`.
```

Rating: Clear enough IMO. You need to install that `arm-none-eabi-ld` binary.

Improvement: None

#### Forgot to launch OpenOCD instance

Error message:

```
$ arm-none-eabi-gdb target/..
Reading symbols from target/thumbv7em-none-eabihf/debug/examples/hello...done.
.gdbinit:1: Error in sourced command file:
:3333: Connection timed out.
```

Rating: Not helpful. Doesn't tell you how to solve the problem.

Improvement: Can't improve the error message. Document fix in troubleshooting
guide.

### Configuration errors

#### Didn't modify the `memory.x` linker script

Error message:

```
$ xargo build
   Compiling demo v0.1.0 (file:///home/japaric/tmp/demo)
error: linking with `arm-none-eabi-ld` failed: exit code: 1
  |
  = note: "arm-none-eabi-ld" "-L" (..)
  = note: arm-none-eabi-ld: address 0xbaaab838 of /home/japaric/tmp/demo/target/thumbv7em-none-eabihf/debug/examples/hello-2ba7b301b6ed599c section `.text' is not within region `FLASH'
          arm-none-eabi-ld: address 0xbaaab838 of /home/japaric/tmp/demo/target/thumbv7em-none-eabihf/debug/examples/hello-2ba7b301b6ed599c section `.text' is not within region `FLASH'
          arm-none-eabi-ld:
          Invalid '.rodata.exceptions' section.
          Make sure to place a static with type `cortex_m::exception::Handlers`
          in that section (cf. #[link_section]) ONLY ONCE.
```

Rating: Not helpful.

Improvement: Can't improve the error message. Document fix in troubleshooting
guide.

#### Forgot to set a default build target

Error message:

```
$ xargo build
(..)
   Compiling cortex-m-semihosting v0.1.3
error[E0463]: can't find crate for `std`

error: aborting due to previous error
```

This is building the crate for the host (e.g. `x86_64-unknown-linux-gnu`)
instead of for the target device.

Rating: Totally misleading.

Improvement: Can't improve the error message. Document fix in troubleshooting
guide.

### edit-compile-test cycle errors

#### Called OpenOCD with wrong arguments

Error message:

```
$ openocd -f ..
(..)
Error: open failed
in procedure 'init'
in procedure 'ocd_bouncer'
```

Rating: Not helpful.

Improvement: Can't improve the error message. Document fix in troubleshooting
guide.

#### Used Cargo instead of Xargo

Error message:

```
$ cargo build
   Compiling vcell v0.1.0
   Compiling cortex-m-semihosting v0.1.3
   Compiling r0 v0.2.1
error[E0463]: can't find crate for `core`
  |
  = note: the `thumbv7em-none-eabihf` target may not be installed

error: aborting due to previous error
```

Rating: Not helpful. `rustup target add thumbv7em-none-eabihf` won't work.

Improvement: Can't improve the error message. Document fix in troubleshooting
guide.

#### Used the stable toolchain

Error message:

```
$ xargo build
error: failed to run `rustc` to learn about target-specific information

To learn more, run the command again with --verbose.
```

Rating: Not helpful at all.

Improvement: Can't improve the error message. Document fix in troubleshooting
guide.

#### Used `CARGO_INCREMENTAL=1`

Error message:

```
$ xargo build
error: linking with `arm-none-eabi-ld` failed: exit code: 1
  |
  = note: "arm-none-eabi-ld" (..)
  = note: arm-none-eabi-ld:
          You must specify the exception handlers.
          Create a non `pub` static variable with type
          `cortex_m::exception::Handlers` and place it in the
          '.rodata.exceptions' section. (cf. #[link_section]). Apply the
          `#[used]` attribute to the variable to make it reach the linker.
          arm-none-eabi-ld:
          Invalid '.rodata.exceptions' section.
          Make sure to place a static with type `cortex_m::exception::Handlers`
          in that section (cf. #[link_section]) ONLY ONCE.
```

Rating: Totally misleading. Even if all the linker related stuff is correctly
configured incremental compilation will break it somehow.

Improvement: Can't improve the error message. Document fix in troubleshooting
guide.

#### Used `gdb` instead of `arm-none-eabi-gdb`

Current error message:

```
$ gdb target/..
Reading symbols from target/thumbv7em-none-eabihf/debug/examples/hello...done.
warning: Architecture rejected target-supplied description
warning: Cannot convert floating-point register value to non-floating-point type.
value has been optimized out
Cannot write the dashboard
Traceback (most recent call last):
  File "<string>", line 353, in render
  File "<string>", line 846, in lines
gdb.error: Frame is invalid.
0x00000000 in ?? ()
semihosting is enabled
Loading section .text, size 0xd88 lma 0x8000000
Start address 0x8000000, load size 3464
.gdbinit:6: Error in sourced command file:
Remote connection closed
```

Rating: Not too helpful. There's some warning text that mentions the word
"architecture".

Improvement: Can't improve the error message. Document fix in troubleshooting
guide.

## Template: `photon-quickstart`

https://particle.io
https://github.com/japaric/photon-quickstart

### Workflow

```
# Install Rust (once)
$ curl https://sh.rustup.rs -sSf | sh
$ rustup default nightly

# Install non-Rust embedded tools (once)
$ sudo apt-get install gcc-arm-none-eabi libnewlib-arm-none-eabi libstdc++-arm-none-eabi-newlib
$ sudo npm install -g particle-cli

# Install Xargo and extra Rust tooling (once)
$ cargo install xargo
$ cargo install --git https://github.com/japaric/particle-tools # for elf2bin

# pair to the photon device (once)
$ particle setup

# start from template
$ git clone https://github.com/japaric/photon-quickstart
$ mv photon-quickstart demo && $_
$ edit Cargo.toml # rename project

# edit-compile-test cycle
$ xargo watch -x check &
$ edit src/main.rs
$ elf2bin target/.. && particle flash $device hello.bin # test

# deploy
$ xargo build --release
$ elf2bin target/.. && particle flash $device hello.bin
```

### Setup errors

#### Forgot to install the `rust-src` component

Same as the `cortex-m-quickstart` error.

#### Forgot to install the `arm-none-eabi-g++` linker

Same as the `cortex-m-quickstart` error.

#### Forgot to install the `libnewlib-arm-none-eabi` package

Error message:

```
$ xargo build
   Compiling my-app v0.1.0 (file:///home/japaric/tmp/my-app)
error: linking with `arm-none-eabi-g++` failed: exit code: 1
  |
  = note: "arm-none-eabi-g++" (..)
  = note: arm-none-eabi-g++: error: nano.specs: No such file or directory
```

Rating: Not helpful.

Improvement: Can't improve the error message. Document fix in troubleshooting
guide.

## Forgot to install `libstdc++-arm-none-eabi-newlib` package

Error message:

```
$ xargo build
   Compiling my-app v0.1.0 (file:///home/japaric/tmp/my-app)
error: linking with `arm-none-eabi-g++` failed: exit code: 1
  |
  = note: "arm-none-eabi-g++" (..)
  = note: /usr/lib/gcc/arm-none-eabi/4.9.3/../../../arm-none-eabi/bin/ld: cannot find -lstdc++_nano
```

Rating: Not helpful.

Improvement: Can't improve the error message. Document fix in troubleshooting
guide.

### edit-compile-test cycle errors

#### Used Cargo instead of Xargo

Same as the `cortex-m-quickstart` error.

#### Used the stable toolchain

Same as the `cortex-m-quickstart` error.

#### Forgot to call the `elf2bin` tool

```
$ particle flash $device target/..
Including:
    target/photon/release/examples/blinky
attempting to flash firmware to your device Ferris
Flash device failed.
[object Object]
```

Rating: Not helpful.

Improvement: Can't improve the error message. Document fix in troubleshooting
guide.

## Template: TockOS userland

### Workflow

```
# Install Rust (once)
$ curl https://sh.rustup.rs -sSf | sh

# Install non-Rust embedded tools (once)
$ sudo apt-get install binutils-arm-none-eabi gdb-arm-none-eabi openocd
$ sudo pip3 install tockloader

# Install Xargo (once)
$ cargo install xargo
$ rustup component add rust-src # for Xargo

# Start from template
$ git clone https://github.com/helena-project/libtock-rs

# edit-compile-test cycle
$ xargo watch -x 'check --target thumbv7em-tock-eabi' &
$ edit src/main.rs
$ tockloader install target/.. # test

# deploy
$ xargo build --target thumbv7em-tock-eabi --relase
$ tockloader install target/..
```

### Setup errors

#### Forgot to install the `rust-src` component

Same as the `cortex-m-quickstart` error.

#### Forgot to install the `arm-none-eabi-ld` linker

Same as the `cortex-m-quickstart` error.

### edit-compile-test cycle errors

#### Used Cargo instead of Xargo

Same as the `cortex-m-quickstart` error.

#### Used the stable toolchain

Same as the `cortex-m-quickstart` error.

#### Used `CARGO_INCREMENTAL=1`

Similar to the `cortex-m-quickstart` one:

```
$ xargo build --target thumbv7em-tock-eabi
error: linking with `arm-none-eabi-ld` failed: exit code: 1
  |
  = note: "arm-none-eabi-ld" (..)
  = note: /tmp/rustc.emRfeLEBRyiO/libtock-732df4b91856a1c6.rlib(tock-732df4b91856a1c6.tock.o):(.ARM.exidx.text._ZN4core3ptr4null17hb03c75bddf04db9dE+0x0): undefined reference to `__aeabi_unwind_cpp_pr0'
          /home/japaric/tmp/libtock-rs/target/thumbv7em-tock-eabi/debug/examples/blink-dcd1858d767d67cf.blink.o:(.ARM.exidx.text._ZN40_$LT$isize$u20$as$u20$core..ops..Add$GT$3add17h85a161a4cee129b4E+0x0): undefined reference to `__aeabi_unwind_cpp_pr0'
(..)
```

# Windows

> **NOTE** Shell is cmd.exe / ConEmu, *not* MSYS(2)

## Template: `cortex-m-quickstart` + `cortex-m-rtfm`

### Workflow

```
# Install Rust (once)
# https://win.rustup.rs
$ rustup default nightly

# Install non-Rust embedded tools (once)
# https://launchpad.net/gcc-arm-embedded/5.0/5-2016-q3-update/+download/gcc-arm-none-eabi-5_4-2016q3-20160926-win32.exe

# Install Xargo and subcommands (once)
$ cargo install cargo-clone [cargo-edit] xargo
$ rustup component add rust-src # for Xargo

# Start from template
$ cargo clone cortex-m-quickstart --vers 0.1.2
$ move cortex-m-quickstart demo
$ cd demo
$ edit Cargo.toml # rename project

# Configure template
$ edit memory.x # specify memory layout

# Set default build target
$ echo [build] >> .cargo/config
$ echo target = "thumbv7em-none-eabihf" >> .cargo/config

# Add dependencies as needed
$ cargo add cortex-m-rtfm ..

# Spawn OpenOCD instance (once per development session)
$ openocd -f .. &

# edit-compile-test cycle
$ xargo watch -x check &
$ edit src/main.rs
$ arm-none-eabi-gdb target/.. # test

# deploy
$ xargo build --release
$ arm-none-eabi-gdb target/..
```

### Different errors

None. All error messages match the ones you get on Linux.

## Template: `photon-quickstart`

```
# Rust setup (once)
# https://win.rustup.rs
$ rustup default nightly

# Embedded tooling (once)
# Install ARM toolchain
# https://launchpad.net/gcc-arm-embedded/5.0/5-2016-q3-update/+download/gcc-arm-none-eabi-5_4-2016q3-20160926-win32.exe
$ npm install -g particle-cli

# Install Xargo and extra Rust tooling
$ cargo install xargo
$ cargo install --git https://github.com/japaric/particle-tools # for elf2bin

# pair to the photon device
$ particle setup

# start from template
$ git clone https://github.com/japaric/photon-quickstart
$ move photon-quickstart demo & cd demo
$ edit Cargo.toml # rename project

# edit-compile-test cycle
$ xargo watch -x check &
$ edit src/main.rs
$ elf2bin target/.. & particle flash $device hello.bin # test

# deploy
$ xargo build --release
$ elf2bin target/.. & particle flash $device hello.bin
```

### Different errors

None. All error messages match the ones you get on Linux.

# Troubleshooting guides

[`cortex-m-quickstart`](https://docs.rs/cortex-m-quickstart/0.1.5/cortex_m_quickstart/#troubleshooting)

[`photon-quickstart`](https://github.com/japaric/photon-quickstart#troubleshooting)
