# eBPF-PayloadFilter

A payload filter designed for running experiments that includes dropping packets.

Run the experiment using the terminal version of WireShark: TShark; tcpdump can also be used.

## Prerequisites

I am using the following setup:

```
Linux 5.13.0-30-generic #33~20.04.1-Ubuntu SMP x86_64 x86_64 x86_64 GNU/Linux
```

There should be the following packages/utils installed:

 - Clang
 - tshark

## Clone and build the filter

This repository leverages libbpf, which is a submodule and needed to be built here.

```bash
git submodule update --init --recursive
make
```

## Run the experiment

Check the loopback interface first using `sudo tshark -D`; assuming we have `lo` as loopback. If not so, modify it yourself in `run_experiments.sh`.

Please use `sudo`:

```bash
sudo ./run_experiments.sh
```

Then you will get a `result.pcapng` file for analysis, also there would be terminal outputs.

Here is a sample result:

![pcapng-result](assets/pcapng-result.png)

## Hints

### How was the filter loaded

First, ensure that you have sudo access, and you know what you are doing about!

How to load the filter:

```bash
ip link set dev lo xdp obj filter.o
```
How to unload the filter:

```bash
ip link set dev lo xdp off
```

How to check if the filter is properly loaded or unloaded:

```bash
ip link list
```

If the filter cannot be loaded successfully, you could try disassemble it to ponder why:

```bash
llvm-objdump -D filter.o
```

### How was it tested

```bash
touch result.pcapng
chmod o=rw result.pcapng # Due to the stupid tshark permission bug
sudo tshark -i lo -f "tcp port 1145" -w "./result.pcapng" -P -a duration:5
```

### How to build the debug version and read the debug outputs

You should change macros in `filter.c` and recompile to run in debug mode.

```bash
sudo cat /sys/kernel/debug/tracing/trace_pipe
```