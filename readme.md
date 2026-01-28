# LA9310 iqplayer repository

la931x_iqplayer repository contains basic application allowing to stream
time domain IQ samples with la9310 SOC. It includes VSPA firmware source code.
It includes host linux scripts to control VSPA firmware.
It also includes host linux library and application example.

# Requirements

- This firmware is using vspa kernels from https://github.com/nxp-qoriq/la931x_vspa_common
- This firmware can be used with LA9310 PCI driver located at https://github.com/nxp-qoriq/la93xx_host_sw
- A default linux integration is provided for imx8mp platforms at https://github.com/nxp-real-time-edge-sw/yocto-real-time-edge/tree/la93xx
- Linux VSPA tool chain is freely availabel when downloading eval version of CodeWarrior for VSPA 10.3.0 or at https://www.nxp.com/lgfiles/sdk/la1224/imx-la9310-sdk-10/cwvspa.vbeta_14_00_781_vspa.linux.tgz 

# Libraries

This repository includes following elements:
- `iqplayer_cwproj` source code, Makefile and CW projects allowing to compile VSPA firmwares supporting usecases 1T1R, 1T, 1R, 1T2R.
- `host-utils` linux user space utilities:
  - Basic linux shell scripts to start/stop transfers ./iq-replay.sh, ./iq-capture.sh, ./iq-stop.sh
  - iq_mon : statistic monitoring showing traffic in various dma/fifos
  - iq_trace : dump VSPA trace information used to debug/profile VSPA firmware
  - iq_app/lib_iqplayer : Application example, showing how to stream traffic to/from upper stack

## Compile iqplayer firmware and utilities

set compiler install path default 
(VSPA_TOOL ?= /opt/VSPA_Tools_vbeta_14_00_781)
```sh
make
make install
scp -r install/* root@<targetIP>:/
```

## Usage - Play/Capture waveforms from/to binary files  

load and start VSPA firmware
```sh
 ./load-la9310.sh
```

confugre RF (optional for axiq loopback or breakout card)
```sh
 ./config_run_rf.sh limetx
 ./config_run_rf.sh limerx
```

play (repeat) waveform from file 
```sh
 ./iq-replay.sh ./tone_td_3p072Mhz_20ms_4KB1200_2c.bin 1200
```

capture (one time) iq samples into file
```sh
 ./iq-capture.sh ./iqdata.bin 1200
```

capture (repeat) in DDR buffer 
```sh
 ./iq-capture-ddr.sh 1200
 ./iq-stop.sh
 bin2mem -f iqdata.bin -a 0x9CC00000 -r 4915200
```

## digital/axiq loopback using dfe app control 
```sh
 mount -t hugetlbfs none /dev/hugepages
 echo 24 > /sys/kernel/mm/hugepages/hugepages-2048kB/nr_hugepages
 dpdk-dfe_app -c "axiq_lb enable"
```

## Usage - iq_app Tx use case 

 Default examples use 32KB TX FIFO in DDR  

```sh
   ./iq-app-tx.sh ./tone_td_3p072Mhz_20ms_4KB1200_2c.bin 
   ./iq-stop.sh
```

## Usage - iq_app Rx use case  

 Default examples use 128KB RX FIFO in DDR 
 
```sh
   ./iq-app-rx.sh ./iq-data.bin
   ./iq-stop.sh
   kill <iq_streamer PID>
```

