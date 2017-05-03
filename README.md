# Failover-Clustering-Domain
Steps to deploy Failover Clustering in Domain on Windows Server 2016 + Storage Spaces Direct (S2D)


## Install Features on each node ##
Install the following features on each node remotely: Failover Clustering, File Server and management tools. You must replace value "node" to the name of your servers

```
$Servers = 'nodedomain1','nodedomain2','nodedomain3' 
$Servers | ForEach { Install-WindowsFeature -ComputerName $_ -Name Failover-Clustering,FS-FileServer -IncludeManagementTools -restart -verbose}
```

## Check the installation of the features
Execute the command bellow to check the installation. Must be performed for each server

```
get-windowsfeature -computername nodedomain2
```

## Check if the nodes can support Storage Spaces Direct

```
Test-cluster -node nodedomain1, nodedomain2, nodedomain3 -Include "Storage Spaces Direct",Inventory,Network,"System Configuration"
```

## Create Failover Clustering
This command bellow create the cluster without a shared storage and uses a static IP Address

```
new-cluster -name democluster -node nodedomain1, nodedomain2, nodedomain3 -nostorage -staticaddress 192.168.1.130 -verbose
```

## Enable Storage Spaces Direct ##
Enable S2D to be used by Cluster. Ensure that you have enough extra disks on each node (same quantity and size)

```
enable-clusters2d -verbose
```

## Create a Storage Pool using S2D ##
This command will create your first Storage Pool using disks from S2D

```
New-Volume -StoragePoolFriendlyName S2D* -FriendlyName MultiResilient -FileSystem CSVFS_REFS -StorageTierFriendlyName Capacity -StorageTierSizes 10GB
```

## Check installation ##
- Open Failover Cluster Manager and select the properties of "democluster"
- List disks
- Open the folder c:\ClusterStorage to check if the CSV was enabled
- Execute the following PowerShell command to check disks
```
Get-ClusterAvailableDisk | Add-ClusterDisk
```
