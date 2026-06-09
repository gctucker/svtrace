<!-- Copyright (C) 2024, RTE (http://www.rte-france.com)
Copyright (C) 2024 Savoir-faire Linux, Inc.
SPDX-License-Identifier: Apache-2.0 -->

# svtrace

svtrace is a tool used to evaluate the performance of IEC61850
Sample Values kernel processing on a machine.

## Table of Contents

- [Introduction](#introduction)
- [Features](#features)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)

## Introduction

The `svtrace` project is a tool focused on measuring the latency of
IEC61850 Sample Values. Powered by bpftrace, it achieves to computes the
transit latency of IEC61850 Sample Values from their arrival on the
interface IRQ process, to their processing by the virtualization agent
(currently vhost driver, hosted by qemu).

svtrace is designed to work either on a physical machine
(hypervisor) or Virtual Machine. Currently, only qemu using vhost-net
drivers virtualization on the host side and virtio drivers on the VM side
is supported.

svtrace uses the power of bpftrace and Linux kprobes to achieve
the measurement of Sample Values latency. Currently, on both physical
and virtual machines, the following kprobes are used:
* `tpacket_rcv` on SV arrival, in SV interface IRQ process
* `consume_skb` on SV proceed, in vhost driver QEMU thread

The following kprobes have been selected as they represent well the
lifetime of an SV on a specific machine. But note that it represents
only a small part of the lifetime; For more scalable tests we recommend
using the seapath performance tool project.

## Features

- Python-based implementation for ease of use and flexibility
- Utilizes bpftrace for high-precision kernel latency measurements
- Designed specifically for IEC61850 SV packets
- `Live` feature for direct in terminal latency measurements
- `record` feature for recording latencies measurements in a file,
   which can be used later for post-processing


## Installation

bpftrace is required on version > v0.20.0. For some Linux distros as
Ubuntu 22.04 the bpftrace package is pretty old, and you would need to
install it directly from the repository:

```bash
wget https://github.com/bpftrace/bpftrace/releases/download/v0.21.2/bpftrace .
chmod +x bpftrace
```

Make sure you satisfy the following dependencies:
```
python3
python3-setuptools
tshark
```

Then, follow these steps to install `svtrace`:

```bash
git clone https://github.com/seapath/svtrace.git
cd svtrace
python3 setup.py install
```


## Usage

The `svtrace.cfg` configuration file must be provided to svtrace.
The network interface used to receive the IEC61850 Sample Values must
be set in the `SV_INTERFACE` field.

For example:

```bash
[DEFAULT]
SV_INTERFACE=enp7s0
```

To start a measurement and print it directly in terminal, on a
hypervisor use the following command:

```bash
python3 svtrace.py --live --conf svtrace.cfg --machine hypervisor
```

To exit svtrace, hit CTRL+C.

In the same way, to start a measurement and record it in a file, use
the following command:

```bash
python3 svtrace.py --record --conf svtrace.cfg --machine hypervisor
```

Note: By default:
* results are saved in the `/tmp/` directory.
* svtrace is running in `live` mode.

## Release notes

### Version v0.1

* Live measurement in terminal
* Record measurement in a file



```
2026-06-09 12:41:22,487 - LiveCapture - DEBUG - Creating Dumpcap subprocess with parameters: /usr/bin/dumpcap -q -i br0 -w -
2026-06-09 12:41:22,488 - LiveCapture - DEBUG - Dumpcap subprocess (pid 81648) created
2026-06-09 12:41:22,699 - LiveCapture - DEBUG - Creating TShark subprocess with parameters: /usr/bin/tshark -l -n -T pdml -c 1 -i -
2026-06-09 12:41:22,699 - LiveCapture - DEBUG - Executable: /usr/bin/tshark
2026-06-09 12:41:22,699 - LiveCapture - DEBUG - Capturing on 'br0'
2026-06-09 12:41:22,699 - LiveCapture - DEBUG - dumpcap: You do not have permission to capture on device "br0".
2026-06-09 12:41:22,699 - LiveCapture - DEBUG - (socket: Operation not permitted)
2026-06-09 12:41:22,700 - LiveCapture - DEBUG - Please check to make sure you have sufficient permissions.
2026-06-09 12:41:22,700 - LiveCapture - DEBUG -
2026-06-09 12:41:22,700 - LiveCapture - DEBUG - On Debian and Debian derivatives such as Ubuntu, if you have installed Wireshark from a package, try running
2026-06-09 12:41:22,700 - LiveCapture - DEBUG -
2026-06-09 12:41:22,700 - LiveCapture - DEBUG - sudo dpkg-reconfigure wireshark-common
2026-06-09 12:41:22,700 - LiveCapture - DEBUG -
2026-06-09 12:41:22,700 - LiveCapture - DEBUG - selecting "<Yes>" in response to the question
2026-06-09 12:41:22,700 - LiveCapture - DEBUG -
2026-06-09 12:41:22,700 - LiveCapture - DEBUG - Should non-superusers be able to capture packets?
2026-06-09 12:41:22,700 - LiveCapture - DEBUG -
2026-06-09 12:41:22,700 - LiveCapture - DEBUG - adding yourself to the "wireshark" group by running
2026-06-09 12:41:22,700 - LiveCapture - DEBUG -
2026-06-09 12:41:22,700 - LiveCapture - DEBUG - sudo usermod -a -G wireshark {your username}
2026-06-09 12:41:22,700 - LiveCapture - DEBUG -
2026-06-09 12:41:22,700 - LiveCapture - DEBUG - and then logging out and logging back in again.
2026-06-09 12:41:22,700 - LiveCapture - DEBUG -
2026-06-09 12:41:22,700 - LiveCapture - DEBUG - If you did not install Wireshark from a package, ensure that Dumpcap has the needed CAP_NET_RAW and CAP_NET_ADMIN capabilities by running
2026-06-09 12:41:22,700 - LiveCapture - DEBUG -
2026-06-09 12:41:22,700 - LiveCapture - DEBUG - sudo setcap cap_net_raw,cap_net_admin=ep {path/to/}dumpcap
2026-06-09 12:41:22,700 - LiveCapture - DEBUG -
2026-06-09 12:41:22,700 - LiveCapture - DEBUG - and then restarting Wireshark.
2026-06-09 12:41:22,700 - LiveCapture - DEBUG - TShark subprocess (pid 81677) created
2026-06-09 12:41:22,701 - LiveCapture - DEBUG - Starting to go through packets
2026-06-09 12:41:22,889 - LiveCapture - DEBUG - Capturing on 'Standard input'
```

```
Detected SV stream: svID0000
Fatal: Required PID process enp7s0 not found
```

```
/home/gtucker/project/energy/seapath/svtrace/svtracing/live.bt:150:47-48: WARNING: signed operands for '/' can lead to undefined behavior (cast to unsigned to silence warning)
    printf("SV buffer fill: %d%\n", (len(@t1) / 100000) * 100);
```

```
DEBUG total:      106907
DEBUG proto:      96000
DEBUG stream:     12000
DEBUG skb:        62559
DEBUG skb proto:  0
DEBUG skb pid:    0
SV received: 0
SV missed: 0
SV dropped: 0
SV buffer fill: 0%
No SV collected.
PROTO 47752 56710
PROTO 47752 56710
```
