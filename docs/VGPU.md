# NVIDIA vGPU

VFiasco supports two modes of NVIDIA vGPU passthrough on RHEL 10 and
CentOS Stream 10:

- **`--feature=vgpu`** (SR-IOV vGPU) — each VF is assigned a vGPU type
  via sysfs and passed through to the guest as a PCI hostdev device.
- **`--feature=mdev_vgpu`** (mediated device vGPU) — a mediated device is
  created directly on the PF via `virsh nodedev-define/start` and attached
  to the guest as an mdev hostdev device. Does not require SR-IOV.

## Supported hardware

- **`--feature=vgpu`** — NVIDIA GPUs with SR-IOV support (e.g. L4, A100, A30, H100).
- **`--feature=mdev_vgpu`** — NVIDIA GPUs that support mediated devices
  (e.g. Quadro RTX 6000). SR-IOV is not required.

The GPU must be listed by `lspci` and not bound to the `nouveau` driver.

---

## Device configuration

Add the GPU to `config/host_devices.ini`:

```ini
[nvidia_gpu]
device_addr   = 0000:01:00.0
vgpu_type     = 913
```

- **`device_addr`** — PCI address of the physical GPU (`lspci -D | grep -i nvidia`)
- **`vgpu_type`** — numeric vGPU type ID (required for `--feature=vgpu` and `mdev_vgpu`)

### Finding the vGPU type ID

**For `--feature=vgpu` (SR-IOV):** enable VFs first, then check creatable
types on a VF:

```bash
/usr/lib/nvidia/sriov-manage -e 0000:01:00.0
cat /sys/bus/pci/devices/0000:01:00.4/nvidia/creatable_vgpu_types
```

**For `--feature=mdev_vgpu` (mediated device):** list supported mdev types
on the PF:

```bash
ls /sys/class/mdev_bus/0000:01:00.0/mdev_supported_types/
```

Each type directory name is `nvidia-<ID>`. Use the numeric `<ID>` as the
`vgpu_type` value in `host_devices.ini`.

---

## Running tests

### Pretest check

```bash
python3 vfiasco.py \
  --feature vgpu --device_name nvidia_gpu --domain centos10 \
  --machine_type q35 --check
```

The pretest check verifies:
- `nvidia-smi` is available
- Device address exists in sysfs

### Single test

```bash
python3 vfiasco.py \
  --feature vgpu --device_name nvidia_gpu --domain centos10 \
  --machine_type q35 --bios ovmf --test_list boot_test
```

---

## How it works

### SR-IOV vGPU (`--feature=vgpu`)

1. The PF address from `host_devices.ini` is passed to
   `/usr/lib/nvidia/sriov-manage -e` to create VFs
2. Each VF gets the configured `vgpu_type` written to
   `/sys/bus/pci/devices/<vf>/nvidia/current_vgpu_type`
3. VFs are passed through to the guest as PCI hostdev devices
4. After tests complete, VFs are disabled via `sriov-manage -d`

### Mediated device vGPU (`--feature=mdev_vgpu`)

1. A mediated device is created directly on the PF using
   `virsh nodedev-define` with type `nvidia-<vgpu_type>` and a generated UUID
2. The mdev is started (`virsh nodedev-start`) and attached to the guest
   using the `domain_xml/mdev_vgpu.xml` template
3. After tests complete, the mdev is destroyed (`virsh nodedev-destroy`)
   and undefined

---

See also: [Setup](SETUP.md) | [Configuration](CONFIGURATION.md) | [Tests](TESTS.md) | [GraceHopper H100 GPU](GRACEHOPPER.md)
