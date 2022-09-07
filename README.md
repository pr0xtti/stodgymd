# StodgyMD

stodgymd is a tool for analyzing multicast data (FAST) which stored in pcap files.
It uses scapy (https://scapy.net).

## Usage

```bash
stodgymd [options] infile
```

| Option       | Description                                                                                                |
|--------------|------------------------------------------------------------------------------------------------------------|
| -c _count_   | Process only _count_ frame from pcap file begining                                                         |
| -h           | Help                                                                                                       |
| -p           | Don't print packet info on every udp packet, line by line                                                  |
| -s           | Do sequence number analysis for gaps, reordering and duplication                                           |
| -t           | Calculate time difference between multicast source timestamp and pcap written timestamp.                   |
| --version    | Print version                                                                                              |
| -w _outfile_ | Write to file instead of stdout. Txt extension will be added to the infile file name, if no outfile given. |

### Examples:

```bash
# Time delta check for 100 packets
$ python3 ../bin/stodgymd -t -c 100 test.pcap 
Time delta:
     delta(us) src_time                     pcap_time                                 seq_num   pkt_num
min:        15 2022-07-21 15:34:20.589226   2022-07-21 15:34:20.589241       6581676639633198         1
max:        41 2022-07-21 15:34:20.590931   2022-07-21 15:34:20.590972       6581676639633203         6
avg:        24
Elapsed: 0.3 s

$ python3 ../bin/stodgymd -s test-loss-reorder.pcap -p
WARN: pkt 104 seq_num 6581676639633302 received, 6581676639633301 expected, lost 1
WARN: pkt 111 seq_num 6581676639633314 received, 6581676639633309 expected, lost 5
WARN: pkt 116 seq_num 6581676639633309 out-of-order
WARN: pkt 117 seq_num 6581676639633310 out-of-order
WARN: pkt 118 seq_num 6581676639633311 out-of-order
WARN: pkt 119 seq_num 6581676639633312 out-of-order
WARN: pkt 120 seq_num 6581676639633313 out-of-order
Lost: 1
Duplicated: 0
Out-of-order: 5
Duplicated or out-of-order: 0 (do manual investigation)
Processed: 1099 frames/pkts
Elapsed: 1.2 s
```

# Description

It is assumed that:
- Pcap file _infile_ contains only IPv4 UDP multicast data and for a single multicast group.
- Pcap file _infile_ contains at least 20 bytes of UDP data per each packet. 
The program will parse only udp packets. 

Pcap file can be written with tcpdump as follows:
```bash
tcpdump -nn -i eth0 -s 62 udp and host x.x.x.x -w file.pcap
```
Where x.x.x.x is a multicast group IPv4 address.

# Installation

## 1. Copy stodgymd

```bash
git pull https://github.com/pr0xtti/stodgymd.git
```

## 2. Install scapy

### Install scapy with pip in user directory.

```bash
pip install --user scapy
```
Run
```bash
python3 stodgymd -h 
```


### Install scapy in venv.

```bash
python3 -m venv stodgymd-venv
. ./stodgymd-venv/bin/activate 
pip install scapy
```
Run
```bash
. ./stodgy-venv/bin/activate
python3 stodgymd -h
```


