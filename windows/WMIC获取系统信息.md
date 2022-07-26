# 获取C盘的容量

```bat
wmic LogicalDisk where "Caption='C:'" get FreeSpace,Size /value


FreeSpace=12495155200
Size=127007715328
```

# 获取硬盘信息

```bat
wmic diskdrive
Availability  BytesPerSector  Capabilities  CapabilityDescriptions                 Caption               CompressionMethod  ConfigManagerErrorCode  ConfigManagerUserConfig  CreationClassName  DefaultBlockSize  Description        DeviceID            ErrorCleared  ErrorDescription  ErrorMethodology  FirmwareRevision  Index  InstallDate  InterfaceType  LastErrorCode  Manufacturer             MaxBlockSize  MaxMediaSize  MediaLoaded  MediaType              MinBlockSize  Model                 Name                NeedsCleaning  NumberOfMediaSupported  Partitions  PNPDeviceID                                                  PowerManagementCapabilities  PowerManagementSupported  SCSIBus  SCSILogicalUnit  SCSIPort  SCSITargetId  SectorsPerTrack  SerialNumber          Signature  Size           Status  StatusInfo  SystemCreationClassName  SystemName    TotalCylinders  TotalHeads  TotalSectors  TotalTracks  TracksPerCylinder
              512             {3, 4}        {"Random Access", "Supports Writing"}  WDC WD10EZEX-60WN4A1                     0                       FALSE                    Win32_DiskDrive                      ディスク ドライブ  \\.\PHYSICALDRIVE1                                                    03.01A03          1                   IDE                           (標準ディスク ドライブ)                              TRUE         Fixed hard disk media                WDC WD10EZEX-60WN4A1  \\.\PHYSICALDRIVE1                                         2           SCSI\DISK&VEN_WDC&PROD_WD10EZEX-60WN4A1\4&20CFC930&0&000000                                                         0        0                1         0             63               WD-WCC6Y3NF16SR                  1000202273280  OK                  Win32_ComputerSystem     GUT-D2019-06  121601          255         1953520065    31008255     255 
              512             {3, 4}        {"Random Access", "Supports Writing"}  LITEON CL1-8D128-HP                      0                       FALSE                    Win32_DiskDrive                      ディスク ドライブ  \\.\PHYSICALDRIVE0                                                    L182              0                   SCSI                          (標準ディスク ドライブ)                              TRUE         Fixed hard disk media                LITEON CL1-8D128-HP   \\.\PHYSICALDRIVE0                                         3           SCSI\DISK&VEN_NVME&PROD_LITEON_CL1-8D128\5&D55AAA1&0&000000                                                         0        0                0         0             63               0023_0356_3028_DA2B.             128034708480   OK                  Win32_ComputerSystem     GUT-D2019-06  15566           255         250067790     3969330      255 
```

# 获取逻辑分区信息

```bat
Wmic logicaldisk
Access  Availability  BlockSize  Caption  Compressed  ConfigManagerErrorCode  ConfigManagerUserConfig  CreationClassName  Description           DeviceID  DriveType  ErrorCleared  ErrorDescription  ErrorMethodology  FileSystem  FreeSpace     InstallDate  LastErrorCode  MaximumComponentLength  MediaType  Name  NumberOfBlocks  PNPDeviceID  PowerManagementCapabilities  PowerManagementSupported  ProviderName  Purpose  QuotasDisabled  QuotasIncomplete  QuotasRebuilding  Size          Status  StatusInfo  SupportsDiskQuotas  SupportsFileBasedCompression  SystemCreationClassName  SystemName    VolumeDirty  VolumeName                        VolumeSerialNumber
0                                C:       FALSE                                                        Win32_LogicalDisk  ローカル固定ディスク  C:        3                                                            NTFS        12501127168                               255                     12         C:                                                                                                                                                                   127007715328                      FALSE               TRUE                          Win32_ComputerSystem     GUT-D2019-06               Windows                           38E1D04B
0                                D:       FALSE                                                        Win32_LogicalDisk  ローカル固定ディスク  D:        3                                                            NTFS        399250268160                              255                     12         D:                                                                                                                                                                   895211270144                      FALSE               TRUE                          Win32_ComputerSystem     GUT-D2019-06               DATADRIVE1                        34799A4C
                                 E:                                                                    Win32_LogicalDisk  CD-ROM ディスク       E:        5                                                                                                                                          11         E:                                                                                                                                                                                                                                                       Win32_ComputerSystem     GUT-D2019-06
0                                F:       FALSE                                                        Win32_LogicalDisk  ローカル固定ディスク  F:        3                                                            NTFS        104625008640                              255                     12         F:                                                                                                                                                                   104856547328                      FALSE               TRUE                          Win32_ComputerSystem     GUT-D2019-06               ボリューム                        16665B47
1                                G:       FALSE                                                        Win32_LogicalDisk  CD-ROM ディスク       G:        5                                                            CDFS        0                                         110                     11         G:                                                                                                                                                                   3659776                           FALSE               FALSE                         Win32_ComputerSystem     GUT-D2019-06  FALSE        ??手机助手??????????  704381A2
```

# CPU信息

```bat
wmic cpu
AddressWidth  Architecture  AssetTag                Availability  Caption                                 Characteristics  ConfigManagerErrorCode  ConfigManagerUserConfig  CpuStatus  CreationClassName  CurrentClockSpeed  CurrentVoltage  DataWidth  Description                             DeviceID  ErrorCleared  ErrorDescription  ExtClock  Family  InstallDate  L2CacheSize  L2CacheSpeed  L3CacheSize  L3CacheSpeed  LastErrorCode  Level  LoadPercentage  Manufacturer  MaxClockSpeed  Name                                     NumberOfCores  NumberOfEnabledCore  NumberOfLogicalProcessors  OtherFamilyDescription  PartNumber              PNPDeviceID  PowerManagementCapabilities  PowerManagementSupported  ProcessorId       ProcessorType  Revision  Role  SecondLevelAddressTranslationExtensions  SerialNumber            SocketDesignation  Status  StatusInfo  Stepping  SystemCreationClassName  SystemName    ThreadCount  UniqueId  UpgradeMethod  Version  VirtualizationFirmwareEnabled  VMMonitorModeExtensions  VoltageCaps
64            9             To Be Filled By O.E.M.  3             Intel64 Family 6 Model 158 Stepping 11  236                                                               1          Win32_Processor    3600               11              64         Intel64 Family 6 Model 158 Stepping 11  CPU0                                      100       206                  1024                       6144         0                            6      6               GenuineIntel  3600           Intel(R) Core(TM) i3-9100 CPU @ 3.60GHz  4              4                    4                                                  To Be Filled By O.E.M.                                            FALSE                     BFEBFBFF000906EB  3                        CPU   FALSE                                    To Be Filled By O.E.M.  U3E1               OK      3                     Win32_ComputerSystem     GUT-D2019-06  4                      50                      FALSE                          FALSE
```

# 内存信息

```bat
wmic memorychip
Attributes  BankLabel  Capacity    Caption     ConfiguredClockSpeed  ConfiguredVoltage  CreationClassName     DataWidth  Description  DeviceLocator  FormFactor  HotSwappable  InstallDate  InterleaveDataDepth  InterleavePosition  Manufacturer  MaxVoltage  MemoryType  MinVoltage  Model  Name        OtherIdentifyingInfo  PartNumber        PositionInRow  PoweredOn  Removable  Replaceable  SerialNumber  SKU  SMBIOSMemoryType  Speed  Status  Tag                TotalWidth  TypeDetail  Version
1           ChannelB   8589934592  物理メモリ  2400                  1200               Win32_PhysicalMemory  64         物理メモリ   DIMM1          8                                      0                    0                   Samsung       0           0           0                  物理メモリ                        M378A1K43CB2-CTD  1                                                 13E4ACA2           26                2667           Physical Memory 0  64          128
1           ChannelA   8589934592  物理メモリ  2400                  1200               Win32_PhysicalMemory  64         物理メモリ   DIMM3          8                                      0                    1                   Samsung       0           0           0                  物理メモリ                        M378A1K43CB2-CTD  1                                                 13E4ACA9           26                2667           Physical Memory 1  64          128

```



# 进程信息

```bat
# 查看所有进程
wmic process
# 查看进程所有信息
f:\huanghui>wmic process where name="chrome.exe" list full


CommandLine="C:\Program Files\Google\Chrome\Application\chrome.exe"
CSName=GUT-D2019-06
Description=chrome.exe
ExecutablePath=C:\Program Files\Google\Chrome\Application\chrome.exe
ExecutionState=
Handle=18344
HandleCount=4125
InstallDate=
KernelModeTime=6364687500
MaximumWorkingSetSize=1380
MinimumWorkingSetSize=200
Name=chrome.exe

# 停止进程
wmic process where name='QQ.exe' call terminate

```

# 查找文件DATAFILE - DataFile 管理

```bat
# 查找e盘下test目录(不包括子目录)下的cc.cmd文件
wmic datafile where "drive='e:' and path='\\test\\' and FileName='cc' and Extension='cmd'" list
# ★★查找e盘下所有目录和子目录下的cc.cmd文件,且文件大小大于1K
wmic datafile where "drive='e:' and FileName='cc' and Extension='cmd' and FileSize>'1000'" list
# ★★删除e盘下test目录(不包括子目录)下的非.cmd文件
wmic datafile where "drive='e:' and Extension<>'cmd' and path='test'" call delete
# ★★复制e盘下test目录(不包括子目录)下的cc.cmd文件到e:\,并改名为aa.bat
wmic datafile where "drive='e:' and path='\\test\\' and FileName='cc' and Extension='cmd'" call copy "e:\aa.bat"
# ★★查找h盘下目录含有test,文件名含有perl,后缀为txt的文件
wmic datafile where "drive='h:' and extension='txt' and path like '%\\test\\%' and filename like '%perl%'" get name

```







[参考文档](https://www.cnblogs.com/lsgxeva/p/8283662.html)



