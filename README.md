# StodgyMD

stodgymd is a tool for analyzing multicast market data from pcap files.
It uses scapy (https://scapy.net).

## Usage

```shell
stodgymd [options] -f format infile
```

| Option        | Description                                                                                                                                                                                                       |
|---------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| -c _count_    | Process only _count_ frame from pcap file begining                                                                                                                                                                |
| -f _format_   | Data format to decode MsgSeqNum: <br/>p4 - FAST with a 4-bytes MsgSeqNum added before every FAST-message; <br/>p8 - FAST with an 8-bytes MsgSeqNum added before every FAST-message; <br/>sb - SPBExchange MD Binary |
| -h            | Help                                                                                                                                                                                                              |
| -p            | Don't print packet info on every udp packet, line by line                                                                                                                                                         |
| -s            | Do sequence number analysis for gaps, reordering and duplication                                                                                                                                                  |
| -t            | Time delta. Calculate time difference between multicast source timestamp and pcap written timestamp.                                                                                                              |
| --version     | Print version                                                                                                                                                                                                     |
| -w _outfile_  | Write to file instead of stdout. Txt extension will be added to the infile file name, if no outfile given.                                                                                                        |

### Examples:

Time delta check for 100 packets:
```shell
$ python3 ../bin/stodgymd -f sb -t -p -c 100 test.pcap 
Time delta:
     delta(us) src_time                     pcap_time                               msgseqnum   pkt_num
min:        15 2022-07-21 18:34:20.589226   2022-07-21 18:34:20.589241       6581676639633198         1
max:        41 2022-07-21 18:34:20.590931   2022-07-21 18:34:20.590972       6581676639633203         6
avg:        24
Processed: 100 frames (pkts)
Elapsed: 0.7 s
```

MsgSeqNum analysis:
```shell

$ python3 ../bin/stodgymd -f sb -s test-loss-reorder.pcap -p
WARN: pkt 104 msgseqnum 6581676639633302 received, 6581676639633301 expected, lost 1
WARN: pkt 111 msgseqnum 6581676639633314 received, 6581676639633309 expected, lost 5
WARN: pkt 116 msgseqnum 6581676639633309 out-of-order
WARN: pkt 117 msgseqnum 6581676639633310 out-of-order
WARN: pkt 118 msgseqnum 6581676639633311 out-of-order
WARN: pkt 119 msgseqnum 6581676639633312 out-of-order
WARN: pkt 120 msgseqnum 6581676639633313 out-of-order
Lost: 1
Duplicated: 0
Out-of-order: 5
Duplicated or out-of-order: 0 (do manual investigation)
Processed: 1099 frames (pkts)
Elapsed: 1.1 s
```

# Description

It is assumed that:
- Pcap file _infile_ contains only IPv4 UDP multicast data and for a single multicast group.
- Pcap file _infile_ contains at least this amount of UDP data per each packet:
  - 20 bytes for `sb` format. Tcpdump snaplen 62.
  - 4 bytes for `p4` format. Tcpdump snaplen 46.
  - 8 bytes for `p8` format. Tcpdump snaplen 50.

The program will parse only udp packets.
Time delta calculation implemented only for `sb` format for now. 

Pcap file can be written with tcpdump as follows:
```shell
tcpdump -nn -i eth0 -s 62 udp and host x.x.x.x -w file.pcap
```
Where x.x.x.x is a multicast group IPv4 address.

Supported data formats:
- `-f sb`: SPBExchange MD Binary ([en](https://archives.spbexchange.ru/TS/DOCS/english/MDbinary_en.pdf), [ru](https://archives.spbexchange.ru/TS/DOCS/MDbinary.pdf)).
- `-f p8`: SPBExchange FAST ([ru](https://archives.spbexchange.ru/TS/DOCS/MDfast.pdf)): 
  with an 8-bytes preamble added before every FAST-message for MsgSeqNum.
- `-f p4`: MOEX FAST (ASTS ([en](http://ftp.moex.com/pub/FAST/ASTS/docs/ENG_Market_Data_Multicast_User_Guide_Ver_4_8.pdf), 
  [ru](http://ftp.moex.com/pub/FAST/ASTS/docs/RUS_Market_Data_Multicast_User_Guide_Ver_4_8.pdf)), 
  Spectra ([en](http://ftp.moex.com/pub/FAST/Spectra/prod/docs/spectra_fastgate_en.pdf), 
  [ru](http://ftp.moex.com/pub/FAST/Spectra/prod/docs/spectra_fastgate_ru.pdf))): 
  with a 4-bytes preamble added before every FAST-message for MsgSeqNum.

# Installation

## 1. Copy stodgymd

```shell
git clone https://github.com/pr0xtti/stodgymd.git
```

## 2. Install scapy

### Install scapy with pip in user directory.

```shell
pip install --user scapy
```
Run
```shell
python3 ./stodgymd/stodgymd -h 
```

In this case you can put stodgymd somewhere within PATH reachability, if you like:
```shell
sudo cp ./stodgymd/stodgymd /usr/local/bin/stodgymd
sudo chmod +x /usr/local/bin/stodgymd

stodgymd -h
```

### Install scapy in venv.

```shell
python3 -m venv stodgymd-venv
. ./stodgymd-venv/bin/activate 
pip install scapy
```
Run
```shell
. ./stodgy-venv/bin/activate
python3 stodgymd -h
```



