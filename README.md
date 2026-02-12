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
az vm create \
--resource-group myGroup \
--name myvm \
--image Ubuntu2204 \
--size Standard_NV12ads_A10_v5 \
--admin-username ubuntu \
--generate-ssh-keys
```

4. Open ports

```bash
az vm open-port -g myGroup -n myvm --ports 22
az vm open-port -g myGroup -n myvm --ports 80
az vm open-port -g myGroup -n myvm --ports 443
```
