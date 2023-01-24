<!-- # **vSwarm-&mu;:** Microarchitectural Research for Serverless -->
# Lukewarm Serverless Functions : Characterisation and Optimisation

## CS422 Project
### Under Prof Debadatta Mishra

## Group Members
- Sarthak Rout (190772)
- Manish (190477)
- Nibir Baruah (190545)

## Description
#### Complete Code is present at https://git.cse.iitk.ac.in/manishy/cs422_luke-warm-serverless-function-characterisation-and-optimisations-implementation. Due to large files we cannot upload it on github.

We extend the vSwarm-$\mu$ framework to implement Jukebox, which records L2 instruction misses and replays them at L2 cache level by prefetching them again when that particular function is called again.

Paper:
https://dl.acm.org/doi/10.1145/3470496.3527390


## How to setup the project
- Clone the vSwarm-u repository from https://github.com/vhive-serverless/vSwarm-u

- Follow the quick-start instructions at https://vhive-serverless.github.io/vSwarm-u/to setup the vSwarm-u framework

- Copy the linux/ and resources/gem5 folder from this repository and build the linux kernel with
```bash
make setup/kernel.Makefile build
```

and gem5 with 
```bash
make setup/gem5.Makefile all
```

- Replace the `vmlinux` kernel to wkdir as kernel.

- After building the wkdir, get shell access to the `disk.img` with
```bash
sudo qemu-system-x86_64 
-nographic 
-cpu host -enable-kvm 
-smp 2 
-m 2G 
-drive file=wkdir/disk.img,format=raw 
-kernel wkdir/kernel 
-append 'console=ttyS0 root=/dev/hda2'
-netdev user,id=net0,hostfwd=tcp::10022-:22,hostfwd=tcp::8080-:80 -device virtio-net-pci,netdev=net0
```

- Copy the all.zip file to disk.img in the shell using scp to port 10022

- Install go 1.18 on the disk and unzip the all.zip file locally

- Shutdown the shell using 
```bash
shutdown --n now
```

- Make a copy of disk.img as backup as it can become corrupted if qemu/gem5 are run together or if `setup_function.sh` and `sim_function.sh` is run before the other is complete.

- Copy the gem5utils folder to the vSwarm-u framework.

- Setup the checkpoint using
```bash
cd wkdir
./setup_function.sh
```

- Simulate the experiment using
```bash
cd wkdir
./sim_function.sh
```

## How to run different experiments

- Open up disk.img with qemu and edit test/main.go with
```bash
sudo nano test/main.go
```

- For each region of interest where any function is called, use m5ops to fail with code 69 and terminate with m5 fail code -1 (Custom m5 fail codes can be configure in run_sim.py)

- Make the client binary with
```bash
cd test
make all
```

- Make sure the function binary is present in the test folder

- Shutdown the shell and run 
```bash
cd wkdir
./setup_function.sh
./sim_function.sh
```

- Check the output with
```bash
telnet localhost 3456
```
and checkout the stats at 
`
wkdir/results/fibonacci-go/stats.txt
`

### Note:
For L2 cache misses, check out
`system.llc.demandMisses` and for IPC
check out `system.cpu.exec_context.thread_0.numInsts`

Since we are currently dumping and reseting the stas before and after each region of interest, except the first stats dump, rest dumps will correspond to the statistics after running over a region of interest.


## Common problems:
- If you encounter problem such as file system corruption (deleted inode reference) of disk.img, you can try running  `setup_function.sh` again or replace disk.img with a a backup
