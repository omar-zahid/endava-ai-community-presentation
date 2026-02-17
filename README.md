## Azure VM

1. Create new resource group

```bash
az group create --name myGroup --location malaysiawest

```
2. List available images and SKUs

```bash
az vm image list -o table

az vm list-skus --location malaysiawest -o table --query "[?contains(name, 'Standard_NV')]"

MaxDataDiskCount    MemoryInMB    Name                      NumberOfCores    OsDiskSizeInMB    ResourceDiskSizeInMB
------------------  ------------  ------------------------  ---------------  ----------------  ----------------------
4                   56320         Standard_NV6ads_A10_v5    6                1047552           184320
8                   112640        Standard_NV12ads_A10_v5   12               1047552           368640
16                  225280        Standard_NV18ads_A10_v5   18               1047552           737280
32                  901120        Standard_NV36adms_A10_v5  36               1047552           2949120
32                  450560        Standard_NV36ads_A10_v5   36               1047552           1474560
32                  901120        Standard_NV72ads_A10_v5   72               1047552           2949120
```

Note: You can view quota for sizes by:

```bash

az vm list-usage --location malaysiawest -o table --query "[?contains(name.value, 'NV')]"
```

3. Create the VM

```bash
# 1. Create a simple cloud-init to install the compiler the extension needs
cat <<EOF > cloud-config.txt
#cloud-config
package_upgrade: true
packages:
  - build-essential
  - linux-headers-generic
EOF

# 2. Create the VM with Secure Boot OFF
az vm create \
  --resource-group aitest \
  --name localllm \
  --image Ubuntu2204 \
  --size Standard_NV12ads_A10_v5 \
  --admin-username ubuntu \
  --generate-ssh-keys \
  --security-type TrustedLaunch \
  --enable-secure-boot false \
  --enable-vtpm false \
  --os-disk-size-gb 512 \
  --custom-data cloud-config.txt

# 3. Apply the NVIDIA Extension (wait ~2 mins for cloud-init to finish first)
az vm extension set \
  --resource-group aitest \
  --vm-name localllm \
  --name NvidiaGpuDriverLinux \
  --publisher Microsoft.HpcCompute \
  --version 1.6
```

4. Open ports

```bash
az vm open-port -g myGroup -n myvm --ports 22
az vm open-port -g myGroup -n myvm --ports 80
az vm open-port -g myGroup -n myvm --ports 443
```

5. Install Nvidia drivers

```bash
sudo apt update && sudo apt install -y ubuntu-drivers-common
sudo ubuntu-drivers install
sudo reboot

wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2204/x86_64/cuda-keyring_1.1-1_all.deb
sudo apt install -y ./cuda-keyring_1.1-1_all.deb
sudo apt update
sudo apt -y install cuda-toolkit-12-5
sudo reboot

nvidia-smi
```
