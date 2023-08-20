# A. Installation

## I. Directory Structure

```bash
l2
|__ build #contains files required to compile the code
|   |__ common #contains individual module's makefile
|   |__ config #contains main makefile to generate an 
|   |__ odu #contains scripts for logging, installing executable binary
|   |__ scripts #contains the configuration files netconf libraries and starting netopeer server
|   |__ yang #contains the YANG modules
|   |__ Makefile
|__ docs #contains README and other configuration files for building docs
|__ releases
|__ src #contains layer specific source code
|   |__ 5gnrmac : #MAC source code
|   |__ 5gnrrlc : #RLC source code
|   |__ cm : #common, environment and interface files
|   |__ cu_stub : #Stub code for CU
|   |__ du_app : #DU application and F1 code 
|   |__ mt : #wrapper functions over OS
|   |__ phy_stub : #Stub code for Physical layer
|   |__ rlog : #logging module
|   |__ O1 : #O1 module
|__ test
|__ tools
```

## II. Pre-requisite for Compilation

1. Linux 32-bit/64-bit machine
2. GCC version 4.6.3 and above
3. Install LKSCTP
   - On Ubuntu : 
        ```bash
        sudo apt-get install -y libsctp-dev
        ```
   - On CentOS : 
        ```bash
        sudo yum install -y lksctp-tools-devel
        ```
4. Install PCAP:
   - On ubuntu : 
        ```bash
        sudo apt-get install -y libpcap-dev
        ```
   - On CentOS : 
        ```bash
        sudo yum install -y libpcap-devel
        ```


## III. How to Clean and Build:
- **If you just wanna build the odu quickly, please go to [6. Quick Build](#6-Quick-Build)**

### 1. Build commands:
- ```odu```       - Builds all components of ODU
- ```cu_stub```   - Builds all CU Stub
- ```ric_stub```  - Builds all RIC_Stub
- ```clean_odu``` - clean up ODU
- ```clean_cu```  - clean up CU Stub
- ```clean_ric``` - clean up RIC Stub
- ```clean_all``` - cleanup everything

#### **Options**:
- ```MACHINE=BIT64/BIT32``` - Specify underlying machine type. Default is BIT32
- ```NODE=TEST_STUB```      - Specify if it is a test node. Mandatory for cu_stub/ric_stub. Must not be used for odu
- ```MODE=FDD/TDD```        - Specify duplex mode. Default is FDD
- ```PHY=INTEL_L1```        - Specify type of phy. If not specified, PHY stub is used
- ```PHY_MODE=TIMER```      - Specify mode of phy. Used only if PHY=INTEL_L1. Default is radio mode
- ```O1_ENABLE=YES```       - Specify if O1 interface is enabled. If not specified, it is disabled 

### 2. Building ODU binary:
1. Change to building directory
    ```bash
    cd l2/build/odu
    ```
2. Building ODU binary
   
    Follow the [Options](#Options) commands in [Section III.1](#1-Build-commands)
    ```bash
    make odu MACHINE=...  MODE=...
    ```
3. Cleaning ODU binary
   
    Follow the [Options](#Options) commands in [Section III.1](#1-Build-commands)
    ```bash
    make clean_odu
    ```

### 3. Building CU Stub binary:
1. Build folder
    ```bash
    cd l2/build/odu
    ```
2. Building CU Stub binary
   
    Follow the [Options](#Options) commands in [Section III.1](#1-Build-commands)
    ```bash
    make cu_stub NODE=... MACHINE=... MODE=...
    ```
3. Cleaning CU Stub binary
   
    Follow the [Options](#Options) commands in [Section III.1](#1-Build-commands)
    ```bash
    make clean_cu
    ```

### 4. Building RIC Stub binary:
1. Build folder
    ```bash
    cd l2/build/odu
    ```
2. Building RIC Stub binary
   
    Follow the [Options](#Options) commands in [Section III.1](#1-Build-commands)
    ```bash
    make ric_stub NODE=... MACHINE=... MODE=...
    ```
3. Cleaning RIC Stub binary

    Follow the [Options](#Options) commands in [Section III.1](#1-Build-commands)
    ```bash
    make clean_ric
    ```

### 5. Cleaning ODU, CU Stub and RIC Stub:
```bash
make clean_all
```

### 6. Quick Build
Once you want to re-compile the code, just run the command below.
```bash=
cd l2/build/odu
make clean_all
make odu MACHINE=BIT64 MODE=FDD
make cu_stub MACHINE=BIT64 NODE=TEST_STUB MODE=FDD
make ric_stub MACHINE=BIT64 NODE=TEST_STUB MODE=FDD
```
# B. Execution
## 1. Assign virtual IP addresses as follows:
- **These commands should be run whenever rebooting**

```bash=
sudo ifconfig <interface name>:ODU "192.168.130.81"
sudo ifconfig <interface name>:CU_STUB "192.168.130.82"
sudo ifconfig <interface name>:RIC_STUB "192.168.130.80"
```
> You can assign the virtual IP on `lo` interface

## 2. Execute CU Stub:
```bash=
cd l2/bin/cu_stub
sudo ./cu_stub
```


## 3. Execute RIC Stub:
```bash=
cd l2/bin/ric_stub
sudo ./ric_stub
```

## 4. Execute DU:
```bash=
cd l2/bin/odu
sudo ./odu
```
>
**PS**:
- CU stub and RIC stub must be run (in no particular sequence) before ODU
- If you encounter the problem, try to add the `sudo` to execute

## 5. Generate DL Traffic
Once the UE attach is finished, press `d` in CU Stub window to generate the DL traffic



# C. Adjust the slice configuration
## 1. Adjust the number of slices
- In `du_cfg.h`:
    ```c=300
    #define NUM_OF_SUPPORTED_SLICE  3
    ```

## 2. Adjust the slices which PLMN supports 
-  In `du_cfg.c`:
Adjust the S-NSSAI for each supported slices
```c=649
#ifndef O1_ENABLE
   /* Note: Added these below variable for local testing*/
   Snssai snssai[NUM_OF_SUPPORTED_SLICE] = {{1,{2,3,4}},{2,{3,3,4}},{3,{4,3,4}},{4,{5,3,4}}};
#endif 
```
> S-NSSAI consists of the SST and SD. This step is to configure the SST and SD of each slices. But it is just for local testing.


## 3. Adjust the default RRMPolicyRatio for each slice
- In `du_cfg.c`:
    - Adjust the RRMPolicyRatio based on the rule of RRMPolicyRatio
    -  SST and SD should be same as the step 2 you set
    -  Add / Remove the RRMPolicyRatio according to your requirement
```c=326
   /*Note: Static Configuration, when O1 is not configuring the RRM policy*/
   RrmPolicyList rrmPolicy[NUM_OF_SUPPORTED_SLICE];
   rrmPolicy[0].id[0] = 1;
   rrmPolicy[0].resourceType = PRB;
   rrmPolicy[0].rRMMemberNum = 1;
   memcpy(rrmPolicy[0].rRMPolicyMemberList[0].mcc,duCfgParam.macCellCfg.cellCfg.plmnInfoList[0].plmn.mcc, 3*sizeof(uint8_t));
   memcpy(rrmPolicy[0].rRMPolicyMemberList[0].mnc,duCfgParam.macCellCfg.cellCfg.plmnInfoList[0].plmn.mnc, 3*sizeof(uint8_t));
   rrmPolicy[0].rRMPolicyMemberList[0].sst = 1;
   rrmPolicy[0].rRMPolicyMemberList[0].sd[0] = 2;
   rrmPolicy[0].rRMPolicyMemberList[0].sd[1] = 3;
   rrmPolicy[0].rRMPolicyMemberList[0].sd[2] = 4;
   rrmPolicy[0].rRMPolicyMaxRatio = 100;
   rrmPolicy[0].rRMPolicyMinRatio = 50;
   rrmPolicy[0].rRMPolicyDedicatedRatio = 10;

   rrmPolicy[1].id[0] = 2;
   rrmPolicy[1].resourceType = PRB;
   rrmPolicy[1].rRMMemberNum = 1;
   memcpy(rrmPolicy[1].rRMPolicyMemberList[0].mcc,duCfgParam.macCellCfg.cellCfg.plmnInfoList[0].plmn.mcc, 3*sizeof(uint8_t));
   memcpy(rrmPolicy[1].rRMPolicyMemberList[0].mnc,duCfgParam.macCellCfg.cellCfg.plmnInfoList[0].plmn.mnc, 3*sizeof(uint8_t));
   rrmPolicy[1].rRMPolicyMemberList[0].sst = 2;
   rrmPolicy[1].rRMPolicyMemberList[0].sd[0] = 3;
   rrmPolicy[1].rRMPolicyMemberList[0].sd[1] = 3;
   rrmPolicy[1].rRMPolicyMemberList[0].sd[2] = 4;
   rrmPolicy[1].rRMPolicyMaxRatio = 100;
   rrmPolicy[1].rRMPolicyMinRatio = 50;
   rrmPolicy[1].rRMPolicyDedicatedRatio = 10;
```

## 4. Adjust the traffic of each slice
Because the slice is associated with the DRB, we should know how the DRB is associated with the slice in the code: 

- In `cu_f1ap_msg_hdl.c`:
    - `idx` is the cnt which is traversing the DRB
    - In this code, we can know that the slice which is associated with would be the `DRB ID % Number of Slices`
    - We don't need to change this rule.
```c=3012
      /*SNSSAI*/
      snssaiIdx = (idx% cuCb.numSnssaiSupported);
```

Adjust the number of DRB to generate the traffic of each slice

- In `cu_f1ap_msg_hdl.h`:
    - In this case, we create 3 DRBs for each UE. If we configure the 3 slices, then each DRB is associated with the distinct slice
```c=37
#define MAX_DRB_SET_UE_CONTEXT_SETUP_REQ 3  /*Number of DRBs to be added using UE CONTEXT SETUP procedure*/
```


# D. Adjust the traffic in CU STUB
![](https://github.com/dennis5581407/test/blob/main/illustration_of_traffic_generator.jpg)
- `Number of Transmission`: Specify the number of transmission in one traffic generating
- `Number of Packet`: Specify the number of packet in one transmission
- `Data Size`: Specify the data size of one packet
- `Generating Time Interval`: If CU STUB generates the infinite continuous data transfer, this parameter determines the time interval between traffic generating



In this guide, the **infinite continuous data transfer** is adopted. However, generating the infinite data makes the system unstable. We need to adjust the parameter **carefully**.
> [The problem we encounter while generating the infinite data](https://hackmd.io/jhWvRXOQSEiygElLGmxbdg?view#Problem-2-O-DU-is-crashed-when-generating-full-load-traffic)


Up to now, we observe some settings which is able to reduce the possibility of system crashed:
- The key is to **avoid sending a lot of packets at the same time**:
    - Set `Number of Transmission = 1` 
    - Increase the `Data Size`, reduce the `Number of Packet`
    - Reduce the `Generating Time Interval`

## 1. Generate continuous full load traffic
The default DL traffic is not full load. Hence, we should modify it to generate full load traffic first.

### 1.1. Comment the the default generating time interval
- Modified File: `cu_stub.c`
- Modified Function: `startDlData()`
- Comment the **line 406** :

```c=396
    while(cnt < NUM_DL_PACKETS)
    {
      ret =  cuEgtpDatReq(duId, teId);      
      if(ret != ROK)
      {
         DU_LOG("\nERROR --> EGTP: Issue with teid=%d\n",teId);
         break;
      }
      /* TODO : sleep(1) will be removed later once we will be able to
       * support the continuous data pack transfer */
      // sleep(1);  <<< Comment this line
      cnt++;
    }
```

### 1.2. Add the infinite loop outside `startDlData()`
In order to generate continuous full load traffic, we can add the infinite loop outside `startDlData()`. You can also add a counter here to limit the generating time if needed.
- Modified File: `cu_stub.c`
- Modified Function: `cuConsoleHandler()`
- Add the infinite loop outside `startDlData()` on **line 481**:
- Set the generating time interval as `50ms`. We will have a further discussion for **Generating Time Interval** in [5. Generating time interval](#5-Generating-time-interval).
```c=481
 while(1)
 {
    startDlData();
    usleep(50000); /* Generating Time Interval */
 }
```
    
## 2. Number of transmission

- File Name: `cu_stub.c`
- Function Name: `startDlData()`
- To reduce the possibility of system crash, we set the **Number of transmission** to `1` to decrease the number of transmitting packet at the same time.
```c=381
   int32_t totalNumOfTestFlow = 1; /* Default value is 20 */
```

## 3. Number of packet
- File Name: `cu_stub_egtp.h`
- This parameter is the one of the main parameter to control the loading of traffic. This parameter should **tune according to the number of tunnel (number of DRB) you create**.
- This parameter is set as lower as possible, it could also reduce the possibility of system crash. It is good to **set `4` or less**.
- If you wanna increase the traffic load, it is better to adjust **Data size** (Packet size) discussed in [4. Data size](#4-Data-size).
```c=47
#define NUM_DL_PACKETS 4 /* Default value is 1 */
```

## 4. Data size
Data size is also known as **Packet size**. This parameter could **increase the traffic load** as well.
### 4.1. Adjust the max data size
- File Name: `cm_inet.h`
- For example, we change it to double. You could change it according to your needs.
```c=275
#define CM_INET_MAX_UDPRAW_MSGSIZE 4096 /* Default value is 2048 */
```

### 4.2. Adjust the payload of a packet
- File Name: `cu_stub_egtp.c`
- Function Name: `BuildAppMsg()`
- For example, we adjust the packet payload size from `1215` to `2430`. You could adjust it according to your needs
```c=720
   /* Default data size is 1215 */
   char data[2430] = "In telecommunications, 5G is the fifth generation technology standard for broadband cellular"
   " networks, which cellular phone companies began deploying worldwide in 2019, and is the planned successor to the 4G "
   " networks which provide connectivity to most current cellphones. 5G networks are predicted to have more than 1.7"
   " billion subscribers worldwide by 2025, according to the GSM Association.Like its predecessors, 5G networks are"
   " cellular networks,in which the service area is divided into small geographical areas called cells.All 5G wireless"
   " devices in a cell are connected to the Internet and telephone network by radio waves through local antenna in the"
   " cell. The main advantage of the new networks is that they ...";

   int datSize = 2430;
```
## 5. Generating time interval
In [1.2. Add the infinite loop outside startDlData](#12-Add-the-infinite-loop-outside-startDlData), we add the infinite loop outside `startDlData()`. We can set the the **Generating Time Interval** via `usleep()` function. If we don't add the interval here, the system would happen crashed.
- File Name: `cu_stub.c`
- Function Name: `startDlData()`
- This parameter can adjust the **traffic load** as well. If we adjust it lower, it means it can generate more packets in 1 second. It is suitable to do the fine tuning. 

```c=481
 while(1)
 {
    startDlData();
    usleep(50000); /* Generating Time Interval */
 }
```

## 6. Adjust the tunnel to be transmitted
**Each DRB can map to one tunnel**. In case there are **2 UEs**, and each UE has **3 DRBs**:
```
- UE 1: DRB 1 ~ 3 --> Tunnel ID 1 ~ 3
- UE 2: DRB 1 ~ 3 --> Tunnel ID 4 ~ 6
```
The traffic is generated on tunnel. Hence, if the number of tunnel is changed, we should change the parameter to generate the traffic on correct tunnel.

- File Name: `cu_stub_egtp.h`
- Take the example above, there are **6 tunnels** is set up. This changes should apply on traffic generator as well:

```c=46
#define NUM_TUNNEL_TO_PUMP_DATA 6 /* Number of tunnel */
```

# E. Enable the multi-thread feature

- In `sch_slice_based.h`:
    - If we comment this line, odu runs with the single thread
    - If we don't comment this line, odu runs with the multi-thread
```c=19
#define SCH_MULTI_THREAD      /* Enable the multi-thread intra-slice scheduling feature */
```

> Note that enabling the multi-thread feature will cause the CPU usage higher, make sure your hardware is powerful enough (> 16 cores)
# F. Add / Select a scheduling algorithm
- In this framework, each slice can apply different scheduling algortihm.
- Currently, the scheduling algorithm **Round-Robin** and **Weight Fair Queue** are supported. 
- Based on the framework of slice-based scheduler, the developer can add a new scheduling algorithm as he/she wants.

## 1. Add a scheduling algortihm


### 1.1. Add your algorithm name in enumerate
- File Name: `sch_slice_based`
```c=29
typedef enum
{
   RR, /* Round Robin */
   WFQ, /* Weight Fair Queue */
   New Algorithm
}SchAlgorithm;
```

### 1.2. Follow the existing algorithm to write one
- In existing algortihm, the main functionality is allocating the available PRB to each UE and logical channel.
- You could follow and trace the existing algorithm to write a new one. Take **RR** algorithm as an example:
- File Name: `sch_slice_based.c`
- Function Name: `schSliceBasedRoundRobinAlgo()`

### 1.3. Activate the new algortihm
- To activate the new algortihm, there are somewhere should be modified:
- File Name: `sch_slice_based.c`
- Function Name: 
    - `schSliceBasedDlIntraSliceScheduling()`
    - `schSliceBasedDlIntraSliceThreadScheduling()`
    - `schSliceBasedDlFinalScheduling()`
- Try to add a new **if condition** for your algortihm:
```c=
else if(sliceCb->algorithm == New Algorithm)
         {
            if(minimumPrb != 0)
            {
               remainingPrb = minimumPrb;            
               schSliceBasedNewAlgo(cellCb, ueDlNewTransmission, sliceCb->lcInfoList, \
                                          pdschNumSymbols, &remainingPrb, sliceCb->algoMethod, NULLP);
            }
}
```


## 2. Select the scheduling algorithm for each slice
- Currently, the scheduling algorithm of each slice could be assigned under initialization:
- File Name: `sch_slice_based.c`
- Function Name: `SchSliceBasedSliceCfgReq()`
```c=234
    sliceCbToStore->algorithm = RR;
    sliceCbToStore->algoMethod = FLAT;
```
