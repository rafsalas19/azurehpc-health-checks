# AzureHPC Node Health Check #

[![Build Status](https://dev.azure.com/hpc-platform-team/hpc-vm-health-check-framework/_apis/build/status%2Fhpc-vm-health-check-framework?branchName=master)](https://dev.azure.com/hpc-platform-team/hpc-vm-health-check-framework/_build/latest?definitionId=29&branchName=master)

|OS Version|Status Badge|
|----------|------------|
|ND96asr v4|[![Build Status](https://dev.azure.com/hpc-platform-team/hpc-vm-health-check-framework/_apis/build/status%2Fhpc-vm-health-check-framework?branchName=master&jobName=Run_Health_Checks)](https://dev.azure.com/hpc-platform-team/hpc-vm-health-check-framework/_build/latest?definitionId=29&branchName=master)

## Description ##

AzureHPC Node Health Checks provides an automated suite of test that targets specific Azure HPC offerings. This is an extension of [LBNL Node Health Checks](https://github.com/mej/nhc). 

## Supported Offerings ##

- [NDm H100 v5-series](https://learn.microsoft.com/en-us/azure/virtual-machines/nd-h100-v5-series)
- [NCads H100 v5-series](https://learn.microsoft.com/en-us/azure/virtual-machines/ncads-h100-v5)
- [NDm A100 v4-series](https://learn.microsoft.com/en-us/azure/virtual-machines/ndm-a100-v4-series)
- [ND A100 v4-series](https://learn.microsoft.com/en-us/azure/virtual-machines/nda100-v4-series)
- [NC A100 v4-series](https://learn.microsoft.com/en-us/azure/virtual-machines/nc-a100-v4-series)
- [HBv4-series](https://learn.microsoft.com/en-us/azure/virtual-machines/hbv4-series)
- [HX-series](https://learn.microsoft.com/en-us/azure/virtual-machines/hx-series)
- [HBv3-series](https://learn.microsoft.com/en-us/azure/virtual-machines/hbv3-series)
- [HBv2-series](https://learn.microsoft.com/en-us/azure/virtual-machines/hbv2-series)
- [NCv3-series](https://learn.microsoft.com/en-us/azure/virtual-machines/ncv3-series)
- [NDv2-serries](https://learn.microsoft.com/en-us/azure/virtual-machines/ndv2-series)

## Minimum Requirements ##

- Ubunutu 20.0, 22.04
- AlamaLinux >= 8.6
- Cuda >= 12 (for GPU SKUs)
- AMD Clang compiler >= 4.0.0 (Non GPU SKUs)
- Mellanox OFED drivers (For IB related SKUs)
- HPC-X MPI >= v2.11 (This needs to be sourced and added to path. Installed by default in Azure AI/HPC marketplace image)
- [NCCL-tests](https://github.com/NVIDIA/nccl-tests) (clone and build in /opt/ or modify the env variable paths in azure_nccl_allreduce.nhc and azure_nccl_allreduce_ib_loopback.nhc). Nccl-test is installed in the Azure AI/HPC marketplace image

Note: Other distributions may work but are not supported.

## Setup ##

1. To install AzureHPC Node Health Checks run install script:
    - Install in the src directory (recommended): ```./install-nhc.sh```
    - Different installation location: ```./install-nhc.sh <install path>```
    - By default we assume CUDA is installed in /usr/local/cuda if the cuda path is different: ```./install-nhc.sh <install path> <cuda path>```

2. By default following dependencies (or equivalent) will be installed:
    - libpci-dev
    - hwloc
    - build-essential
    - libboost-program-options-dev
    - libssl-dev
    - cmake >= 3.20
    - NHC: https://github.com/mej/nhc
    - Nvbandwidth: https://github.com/NVIDIA/nvbandwidth
    - Perf-test: https://github.com/linux-rdma/perftest
    - Stream: https://www.cs.virginia.edu/stream

3. During installation 'aznhc_env_init.sh' will be generated in the install location. When sourced the follwing will be exported to the environment:
    - AZ_NHC_ROOT=\<install directory\>
    - aznhc=AZ_NHC_ROOT/run-health-checks.sh
    Note: The AZ_NHC_ROOT variable is by various NHC tests. By default 'aznhc_env_init.sh' is sourced in run-health-checks.sh.
    Note: If sourced manually the alias 'aznhc' can be used in place of 'sudo AZ_NHC_ROOT/run-health-checks.sh'

## Configuration ##

This project comes with default VM SKU test configuration files that list the tests to be run. You can modify existing configuration files to suit your testing needs. For information on modifying or creating configuration files please reference [LBNL Node Health Checks documentation](https://github.com/mej/nhc).

## Usage ##

- Invoke health checks using a script that determines SKU and runs the configuration file according to SKU for you:
```sudo ./run-health-checks.sh [-h|--help] [-c|--config <path to an NHC .conf file>] [-o|--output <directory path to output all log files>] [-a|--all_tests] [-v|--verbose]```
  - Default log file path is set to the current directory
  - See help menu for more options:

    | Option        | Argument    | Description                                                                                                                                   |
    |---------------|-------------|-----------------------------------------------------------------------------------------------------------------------------------------------|
    | -h, --help    |             | Display this help                                                                                                                             |
    | -c, --config  | conf file   | Optional path to a custom NHC config file. If not specified the current VM SKU will be detected and the appropriate conf file will be used.   |
    | -o, --output  | log file    | Optional path to output the health check logs to. All directories in the path must exist. If not specified it will use output to ./health.log |
    | -t, --timeout | n seconds   | Optional timeout in seconds for each health check. If not specified it will default to 500 seconds.                                           |
    | -a, --all     |             | Run ALL checks; don't exit on first failure.                                                                                                  |
    | -v, --verbose |             | If set, enables verbose and debug outputs.                                                                                                    |

  - Adding more tests to the configuration files may require modifying the time flag (-t) to avoid timeout. For the default tests provided we recommend setting the timing to 300 seconds but this may vary from machine to machine.

  Notes:
  - Invoke health checks directly: ```sudo nhc -c ./conf/"CONFNAME".conf -l ~/health.log -t 300```
  - If 'aznhc_env_init.sh' sourced the alias 'aznhc' can be used in place of 'sudo AZ_NHC_ROOT/run-health-checks.sh'

## Distributed NHC ##

AzureHPC Node Health Checks also comes bundled with a distributed version of NHC, which is designed to run on a cluster of machines and report back to a central location. This is useful for running health checks on a large cluster with dozens or hundreds of nodes.

See [Distributed NHC](./distributed-nhc/README.md) for more information.

## Health Checks ##

Many of the hardware checks are part of the default NHC project. If you would like to learn more about these check out the [Node Health Checks project](https://github.com/mej/nhc).

The following are Azure custom checks added to the existing NHC suite of tests:

| Check | Component Tested | nd96asr_v4 expected| nd96amsr_a100_v4 expected | nd96isr_h100_v5 expected | hx176rs expected | hb176rs_v4 expected |
|-----|-----|-----|-----|-----|-----|-----|
| check_gpu_count | GPU count | 8 | 8 | 8 | NA | NA |
| check_nvlink_status | NVlink | no inactive links | no inactive links  | no inactive links  | NA | NA |
| check_gpu_xid | GPU XID errors | not present | not present | not present | NA | NA |
| check_nvsmi_healthmon | Nvidia-smi GPU health check | pass | pass | pass | NA | NA |
| check_gpu_bandwidth | GPU DtH/HtD bandwidth | 23 GB/s | 23 GB/s | 52 GB/s | NA | NA |
| check_gpu_ecc | GPU Mem Errors (ECC) |  20000000 | 20000000 | 20000000 | NA | NA |
| check_gpu_clock_throttling | GPU Throttle codes assertion | not present | not present | not present | NA | NA |
| check_nccl_allreduce | GPU NVLink bandwidth | 228 GB/s | 228 GB/s | 460 GB/s | NA | NA |
| check_ib_bw_gdr | IB device (GDR) bandwidth | 175 GB/s | 175 GB/s | 380 GB/s | NA | NA |
| check_ib_bw_non_gdr | IB device (non GDR) bandwidth | NA | NA | NA | 390 GB/s | 390 GB/s |
| check_nccl_allreduce_ib_loopback | GPU/GPU Direct RDMA(GDR) + IB device bandwidth | 18 GB/s | 18 GB/s | NA | NA | NA |
| check_hw_topology | IB/GPU device topology/PCIE mapping | pass | pass | pass | NA | NA |
| check_ib_link_flapping | IB link flap occurrence  | not present | not present | not present | not present | not present |
| check_cpu_stream | CPU compute/memory bandwidth | NA | NA | NA | 665500 MB/s | 665500 MB/s |

Notes:
- The scripts for all tests can be found in the [custom test directory](./customTests/)
- Not all supported SKUs are listed in the above table

## _References_ ##

- [LBNL Node Health Checks](https://github.com/mej/nhc)
- [Azure HPC Images](https://github.com/Azure/azhpc-images)

## Contributing ##

This project welcomes contributions and suggestions.  Most contributions require you to agree to a
Contributor License Agreement (CLA) declaring that you have the right to, and actually do, grant us
the rights to use your contribution. For details, visit https://cla.opensource.microsoft.com.

When you submit a pull request, a CLA bot will automatically determine whether you need to provide
a CLA and decorate the PR appropriately (e.g., status check, comment). Simply follow the instructions
provided by the bot. You will only need to do this once across all repos using our CLA.

This project has adopted the [Microsoft Open Source Code of Conduct](https://opensource.microsoft.com/codeofconduct/).
For more information see the [Code of Conduct FAQ](https://opensource.microsoft.com/codeofconduct/faq/) or
contact [opencode@microsoft.com](mailto:opencode@microsoft.com) with any additional questions or comments.

## Trademarks ##

This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft 
trademarks or logos is subject to and must follow 
[Microsoft's Trademark & Brand Guidelines](https://www.microsoft.com/en-us/legal/intellectualproperty/trademarks/usage/general).
Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship.
Any use of third-party trademarks or logos are subject to those third-party's policies.
