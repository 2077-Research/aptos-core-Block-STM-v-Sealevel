This repository contains the aptos-core fork code used for the Block-STM vs. Sealevle paper tests. Follow these instructions to reproduce the results on a machine of your choice.

# Building Aptos-Core

## **1. Download the source code.**
```bash
git clone https://github.com/2077-Research/aptos-core-Block-STM-v-Sealevel-.git
cd aptos-core
```

**2. Install dependencies**
For Linux: (Tested on Ubuntu 20.04 and Ubuntu 22.04)
```bash
./scripts/dev_setup.sh
```
b. Update shell environment:
```bash
source ~/.cargo/env
```

## **3. Build.**
```bash
cargo build
```

# Benchmarking
Run the command below to run the benchmarks. Use the ubuntu ```taskset``` command to adjust the number of CPUs used.
```bash
cd aptos-move/diem-transaction-benchmarks/src
cargo run --release main
```

## Script to Automate Data Collection
```bash
#!/bin/bash
#create output directory
cd
mkdir data

output_directory="/home/ubuntu/data/tc_"${threadcount}".txt" #adjust as necessary.

#run BSTM benchmarks
threadcounts=(0xF 0xFF 0xFFF 0xFFFF 0xFFFFFF 0xFFFFFFFF)
cd
cd aptos-core
cd aptos-move/aptos-transaction-benchmarks/benches

for threadcount in "${threadcounts[@]}"; do
  taskset "${threadcount}" cargo run --release main |& tee "$output_file_path"
done
```
