# Environment Pre-Check Playbook

Ansible playbook that collects system configuration data from FIO client hosts for performance test baseline analysis. All output is saved as JSON files on the control node.

## Prerequisites

- Ansible installed on the control node
- SSH access (with root/sudo privileges) to all target hosts
- The following tools installed on target hosts:
  - `numactl`
  - `ethtool`
  - `setpci` (part of `pciutils`)
  - `sysctl`

## Inventory Setup

Create a `hosts.ini` file in this directory with your target hosts:

```ini
[control]
control-node.example.com

[fio_clients]
client-1.example.com
client-2.example.com
```

## Variables

| Variable      | Default                        | Description                                       |
|---------------|--------------------------------|---------------------------------------------------|
| `nics`        | `[]`                           | List of network interface names to inspect         |
| `result_dir`  | `/root/ntap-sysconfig-output`  | Directory on the control node where output is saved|

## Running the Playbook

NIC interfaces specified at runtime:

```bash
ansible-playbook -i hosts.ini environment-pre-check.yaml -e '{"nics": ["eth0", "eth1"]}'
```

Override the output directory:

```bash
ansible-playbook -i hosts.ini environment-pre-check.yaml \
  -e '{"nics": ["eth0"], "result_dir": "/tmp/my-output"}'
```

## Output Files

All output is written as pretty-printed JSON to `<result_dir>/` on the control node, one file per host:

| File                                      | Content                                        |
|-------------------------------------------|------------------------------------------------|
| `<hostname>_setup.json`                   | Full Ansible facts (OS, CPU, memory, network)  |
| `<hostname>_sysctl.json`                  | Kernel parameters (`sysctl -a`)                |
| `<hostname>_numactl_hardware.json`        | NUMA topology                                  |
| `<hostname>_nic_numa_node.json`           | NUMA node per NIC                              |
| `<hostname>_nic_local_cpulist.json`       | CPU affinity per NIC                           |
| `<hostname>_nic_interrupt_handling.json`   | Combined/separate interrupt channels           |
| `<hostname>_nic_tx_rx_queue_size.json`    | Ring buffer sizes per NIC                      |
| `<hostname>_nic_mtu_size.json`            | MTU and interface state                        |
| `<hostname>_pcie_mrrs.json`               | PCIe Max Read Request Size                     |
| `<hostname>_pcie_10b_tags.json`           | PCIe 10-bit tag capability                     |