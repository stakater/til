# Setup HP Storageworks P4300 G2

## NICs

The server has the following NICs configured

| Name  | Type          | IP           |
|-------|---------------|--------------|
| NIC 1 | Primary       | 192.168.0.13 |
| NIC 2 | Secondary     | -            |
| Mgmt  | Management    | 10.10.10.30  |

- `NIC 1` will be used for discovery over the network
  - We will attach it to a VLAN
- `Mgmt` NIC will have a public IP address


## Configure the Server

### Physical Configuration

- Connect the following components
  - Power
  - Mouse
  - Keyboard
  - VGA
- Power on the machine (Don't panic if the fans are loud)
- After a while you will see a login screen
- Type `start` and press enter
- Hit enter on the blue interface to login
- Go to `Network TCP/IP Settings`
- Choose `eth0` since that is the main NIC
- Enter hostname `p4300.stakater.local`
- Check the option `Use the following IP address` by using arrow keys and then pressing space
- Use the following values and press `OK`

| Name        | Value           |
|-------------|-----------------|
| IP          | 192.168.0.13    |
| Subnet Mask | 255.255.255.0   |
| Gateway     | 192.168.0.1     |

### Install CMC

CMC is the management tool for HP StorageWorks servers. You need to download it on another machine on the same network to set the server up. You can download it for windows using this [link](https://h30538.www3.hpe.com/prdownloads/HPE_StoreVirtual_Centralized_Management_Console_for_Windows_BM480-10634.exe?downloadid=srCf16oheY5ATyOl5-oADiYXcK7eAXUf6q_7rKQlsvIU7akAa4_FXmh-OrkL3m2-LCtzXi5qHzBSTYy03m3YaHCkDBwH_Bh0xhZVEfAh9GHu0A1xm0GksOJUWjnNnaDibq9C2uPVRfYfeuNPdYzjSrwvi9D0vhW7j-42XF3zTUuC0YcYIhRg7Q==&merchantId=SW_DEPOT&rnid=1.0&bpid=SWD&egid=F&__dlk__=1573580044_7c6cbd73f05170bec039210854b73cac&dummy=file.exe).

Once it is downloaded, open it up

### Find the server

- Click on `Find` on the menu bar and go to `Find Systems...`
- Click `Add...` and type the IP Address of the HP Storageworks server that you specified when setting it up, in our case it is `192.168.0.13`
- You will see that it has been found in the list with the hostname
- Close the window

### Create a Management Group

- Expand `Available Systems` on the left heirarchy to see the added server
- Right click on the server that you added, the name is `p4300.stakater.local` and click `Add to New Management Group...`
- Type `stakater.local` in Management Group Name
- Select the `p4300.stakater.local` server from the list and click `Next`
- Choose a username i.e., `admin` and a strong password and click `Next`
- Click `Add` on the NTP servers screen and add the following servers:
  - 0.europe.pool.ntp.org
  - 1.europe.pool.ntp.org
  - 2.europe.pool.ntp.org
  - 3.europe.pool.ntp.org 
- Click `Next`
- Enter `stakater.local` for DNS Domain Name and click `Next`
- Click `Next` again to skip the email section and then click `Accept Incomplete`
- Choose `Standard Cluster` and click `Next`
- Enter `stakatercluster` for the Cluster Name and select the server and click `Next`
- Click `Add` on the Virtual IP screen and specify `192.168.0.30` as the IP Address and `255.255.255.0` as the Subnet Mask and click `Ok`
- This Virtual IP will become the IP of the storage cluster and we will use this IP from the ESXi host to connect to iSCSI connections
- Click `Next` and then check the `Skip Volume Creation` box and click `Finish` as we will setup the volumes later
- Once it is finished, click `Close`

### Create a Volume for use as iSCSI target

- Expand `stakater.local` management group
- Expand `stakatercluster` under it
- Right click on `Volumes` and click `New Volume...`
- Name the volume `ESX_Datastore1`
- Since we have approximately `5.5TB` of storage available on the server, lets assign a size of `1TB` for the 1st volume
- Do this by typing `1` in the size and choose `TB` from the dropdown
- Click `Advanced` and click `Thin` for provisioning field
- You can see the we only have `Network RAID-0` available for Data Protection Level. This is because we only have 1 server in the cluster. If we had 2 servers, then we could do a `Network RAID-10` setup which will be more fault tolerant
- Click `Ok` to finish creating the volume
