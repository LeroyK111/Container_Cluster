# kubernetes 学习部署笔记

**这里我们以linux平台为主**
官网: https://kubernetes.io/

![](assets/Pasted%20image%2020240804232958.png)
node: 节点服务器
pod: 调度单位, node节点服务器上的一个虚拟环境.
container: 调度单位中的一个容器

**实现了网络的四层抽象.**
## 使用虚拟机实现k8s的单机部署

这里咱就使用windows自带的Microsoft Hyper-V 功能. 去创建三个node节点服务器
https://learn.microsoft.com/zh-cn/virtualization/hyper-v-on-windows/about/

1. VMware家族
	1. VMware Worstation Pro VMware Workstation Pro 是行业标准桌面 Hypervisor，使用它可在 Windows 或 Linux 桌面上运行 Windows、Linux 和 BSD 虚拟机。
	2. VMware Workstation Player对于大多数个人用户来说，免费版的 VMware Workstation Player 其实已经足够了。此外它体积小巧，性能好，支持3D加速，运行流畅稳定。可以说是值得推荐的首选虚拟机软件。
	3. VMware Fusion VMWare Fusion 13 Pro 是 macOS 优秀的虚拟机软件「同类软件 Parallels Desktop」，兼容 macOS Ventura 和苹果 M 系列芯片，可以直接在 Mac 下运行 Windows 11、Linux 等系统。
 2. VirtualBox. VirtualBox 是一个用于 x86 硬件的通用全虚拟器，面向服务器、桌面和嵌入式应用;相对 VMware 来说 ，VirtualBox 是轻量级的虚拟软件, 最关键的是 VirtualBox 是开源免费的。
KVM
3. KVM是Kernel-based Virtual Machine的简称，是一个开源的系统虚拟化模块，自Linux 2.6.20之后集成在Linux的各个主要发行版本中;
   它使用Linux自身的调度器进行管理，所以相对于Xen，其核心源码很少，相对VMWare的管理方式来说是比较麻烦，但从性能上并不比VMWare差。;
4. Parallels Desktop. Parallels Desktop是由Parallels推出的一款为苹果电脑提供硬件虚拟化的软件，产品于2006年6月发布，它是第一款能在苹果-英特尔架构的苹果电脑上使用的虚拟化软件。如果你想在Mac上运行Windows系统，那么Parallels Desktop 会是你的最佳选择。它可以在Intel 或 Apple M 系列 Mac 计算机上无缝运行 Windows 应用，最大限度地解决了 MacOS 与 Windows 软件生态差距方面的问题。
5. QEMU.QEMU是一款由法布里斯·贝拉等人编写，可执行硬件虚拟化的（hardware virtualization）开源仿真器（Emulator）。QEMU与其他VM 解决方案不同的地方在于，它既是虚拟机，也是机器模拟器。QEMU可以通过动态的二进制转换，模拟CPU，并且提供一组设备模型，使它能够运行多种未修改的客户机OS。QEMU还可以通过与KVM一起使用，从而以接近真实电脑的速度来运行虚拟机。
6. Citrix Hypervisor.Citrix Hypervisor 是业界领先的平台，实现了经济高效的桌面、服务器和云虚拟化基础结构；Citrix Hypervisor 支持任意规模或类型的组织整合计算资源，以及将计算资源转换为虚拟工作负载，从而满足现今数据中心的要求。同时可以确保将工作负载无缝移动到云中。
7. Xen.XEN最初是剑桥大学Xensource的一个开源研究项目，2003年9月发布了首个版本Xen 1.0，2007年Xensource被Citrix公司收购，开源Xen转由http://www.xen.org继续推进，该组织成员包括个人和公司（如Citrix、Oracle等）。2014年03月11日，Xen发布4.4版本，更好地支持ARM架构；Xen是运行在裸机上的虚拟化管理程序（HyperVisor），是半虚拟化（Para-Virtualization）技术的典型代表。
8. 8.Microsoft Hyper-V. 无论你是软件开发人员、IT 专业人员还是技术爱好者，你们中的许多人都需要运行多个操作系统。 Hyper-V 让你可以在 Windows 上以虚拟机形式运行多个操作系统。
9. Multipass. Multipass 是一个轻量虚拟机管理器，是由 Ubuntu 运营公司 Canonical 所推出的开源项目。运行环境支持 Linux、Windows、macOS。在不同的操作系统上，使用的是不同的虚拟化技术。在 Linux 上使用的是 KVM、Window 上使用 Hyper-V、macOS 中使用 HyperKit 以最小开销运行VM，支持在笔记本模拟小型云。同时，Multipass 提供了一个命令行界面来启动和管理 Linux 实例。下载一个全新的镜像需要几秒钟的时间，并且在几分钟内就可以启动并运行 VM。


### Hyper-V

#### WSL-2 和 hyper-V 的区别
hyper-V 是完整的硬件有虚拟化, 可以拥有完整虚拟机. 实现完整的系统功能, 必须学习K8S中的node时, 就可以使用.

 WSL-2 则是内核兼容层面的虚拟化, 可以执行linux相关的命令, 但是无法作为独立的node节点去使用. 比如 windows docker 就是这样实现的. linux功能阉割的比较多.
### 启用 Hyper-V 功能
![](assets/Pasted%20image%2020240905011531.png)
也可以使用管理员命令启动.
![](assets/Pasted%20image%2020240905011541.png)

### 使用可视化界面配置虚拟机

你可以使用命令一步创建
```ps1
# Set VM Name, Switch Name, and Installation Media Path.
$VMName = 'TESTVM'
$Switch = 'External VM Switch'
$InstallMedia = 'C:\Users\Administrator\Desktop\en_windows_10_enterprise_x64_dvd_6851151.iso'

$VMName: 定义虚拟机的名称为 TESTVM。
$Switch: 指定 Hyper-V 使用的网络交换机名称为 External VM Switch，用于将虚拟机连接到外部网络。
$InstallMedia: 指定虚拟机安装操作系统的 ISO 文件路径。

# Create New Virtual Machine
New-VM -Name $VMName -MemoryStartupBytes 2147483648 -Generation 2 -NewVHDPath "D:\Virtual Machines\$VMName\$VMName.vhdx" -NewVHDSizeBytes 53687091200 -Path "D:\Virtual Machines\$VMName" -SwitchName $Switch

使用 New-VM 命令创建一个新的虚拟机，指定：
名称：$VMName。
分配内存：2147483648 字节 (2GB)。
虚拟机代数：Generation 2，表示使用较新的 Hyper-V 代数。
硬盘路径：NewVHDPath，创建一个新的虚拟硬盘文件 (VHDX) 存储在指定路径。
硬盘大小：53687091200 字节（50GB）。
网络交换机：使用上面定义的 $Switch。

# Add DVD Drive to Virtual Machine
Add-VMScsiController -VMName $VMName
Add-VMDvdDrive -VMName $VMName -ControllerNumber 1 -ControllerLocation 0 -Path $InstallMedia

Add-VMScsiController: 给虚拟机添加 SCSI 控制器。
Add-VMDvdDrive: 在虚拟机中添加 DVD 驱动器，并将其连接到指定的安装介质路径 $InstallMedia。

# Mount Installation Media
$DVDDrive = Get-VMDvdDrive -VMName $VMName

通过 Get-VMDvdDrive 获取并存储虚拟机的 DVD 驱动器信息。

# Configure Virtual Machine to Boot from DVD
Set-VMFirmware -VMName $VMName -FirstBootDevice $DVDDrive

Set-VMFirmware 命令用于将虚拟机的固件配置为从 DVD 驱动器启动操作系统安装。
```

- 直接下载 ubuntu镜像 https://ubuntu.com/download/desktop
![](assets/Pasted%20image%2020240905013606.png)
![](assets/Pasted%20image%2020240905014032.png)
链接虚拟机, 进行安装
![](assets/Pasted%20image%2020240905014420.png)
![](assets/Pasted%20image%2020240905203916.png)
这样依赖我们就创建了三个虚拟机. 

### 常用命令

```sh
# 1. 运行以下命令以显示适用于 Hyper-V PowerShell 模块的 PowerShell 命令的可搜索列表。
Get-Command -Module hyper-v | Out-GridView
# 若要了解有关特定 PowerShell 命令的详细信息，请使用 `Get-Help`。 例如，运行以下命令将返回有关 `Get-VM` Hyper-V 命令的信息。
Get-Help Get-VM
```

```sh
# 读取虚拟机列表

PS C:\Users\Administrator> Get-VM

Name      State   CPUUsage(%) MemoryAssigned(M) Uptime           Status   Version
----      -----   ----------- ----------------- ------           ------   -------
k8sMaster Running 0           6630              00:51:18.4560000 正常运行 11.0
k8sSlave1 Running 0           7148              00:51:48.7110000 正常运行 11.0
k8sSlave2 Running 0           6832              00:51:51.1230000 正常运行 11.0

# 返回已启动的虚拟机
Get-VM | where {$_.State -eq 'Running'}
# 关机的虚拟机
Get-VM | where {$_.State -eq 'Off'}
# 特定名字的虚拟机 启动
Start-VM -Name k8sMaster

# 启动所有已关机的虚拟机
Get-VM | where {$_.State -eq 'Off'} | Start-VM
# 关闭所有虚拟机
Get-VM | where {$_.State -eq 'Running'} | Stop-VM

# 创建虚拟机的快照
Get-VM -Name <VM Name> | Checkpoint-VM -SnapshotName <name for snapshot>
```
新建虚拟机
```sh
$VMName = "VMNAME"

$VM = @{
     Name = $VMName
     MemoryStartupBytes = 2147483648
     Generation = 2
     NewVHDPath = "C:\Virtual Machines\$VMName\$VMName.vhdx"
     NewVHDSizeBytes = 53687091200
     BootDevice = "VHD"
     Path = "C:\Virtual Machines\$VMName"
     SwitchName = (Get-VMSwitch).Name
 }

New-VM @VM
```
powershell 直连, 仅限windows主机之间. 不提供进入linux等其他虚拟机操作命令行的 command.
https://learn.microsoft.com/zh-cn/virtualization/hyper-v-on-windows/user-guide/powershell-direct

### 创建虚拟网关
推荐可视化配置
![](assets/Pasted%20image%2020240905234708.png)

以下是命令行配置嵌套虚拟机网络：
https://learn.microsoft.com/zh-cn/virtualization/hyper-v-on-windows/user-guide/enable-nested-virtualization

对于在虚拟机中创建虚拟机,  是可以的, 但不推荐啊. 资源被消耗的太快.

但是我们是要使用容器编排技术, 需要 启用嵌套虚拟化.

powershell 管理员模式
```powershell
# 关闭所有的 Runing 虚拟机
Get-VM | where {$_.State -eq 'Running'} | Stop-VM

# 所有的虚拟系统
PS C:\Users\Administrator> Get-VM

Name      State CPUUsage(%) MemoryAssigned(M) Uptime   Status   Version
----      ----- ----------- ----------------- ------   ------   -------
k8sMaster Off   0           0                 00:00:00 正常运行 11.0
k8sSlave1 Off   0           0                 00:00:00 正常运行 11.0
k8sSlave2 Off   0           0                 00:00:00 正常运行 11.0

# 开启所有的 虚拟机的嵌套虚拟化 
Get-VM | Where-Object { $_.State -eq 'Off' } | ForEach-Object { Set-VMProcessor -VMName $_.Name -ExposeVirtualizationExtensions $true }

# 启动所有的 Runing 虚拟机
Get-VM | where {$_.State -eq 'Running'} | Start-VM

# 关闭虚拟机的 嵌套虚拟化
Set-VMProcessor -VMName <VMName> -ExposeVirtualizationExtensions $false
```

这样虚拟机就可以再次开启虚拟机了.  
windows宿主机 -> Hyper-V ( Linux ) -> Linux  开启 docker kvm 

#### 配置虚拟交换机
- NAT网络地址转换: https://learn.microsoft.com/zh-cn/virtualization/hyper-v-on-windows/user-guide/setup-nat-network  选择内部虚拟交换机, 让虚拟机能够链接外网
- 外部, 本质就是虚拟机对外放出接口, 内部服务器反向代理, 一般不推荐
- 专用, 虚拟机之间相互通讯. 

![](assets/Pasted%20image%2020240905235440.png)

### 其他功能
用不太上, 毕竟windows虚拟机还是学习和测试用,  生产环境还是纯粹的 container容器 or linux裸机
https://learn.microsoft.com/zh-cn/virtualization/hyper-v-on-windows/user-guide/refactor-wmiv1-to-wmiv2


## 自学版

[我们快速过了一遍教程版的基础上，我们根据官方文档再次学习一遍](TutorialEdition.md)
### 主要名词解释:
- schedulet: 调度器, 为node分配任务
- api server 集群网关,.负载均衡
- etcd 键值数据库
	- v2 内存
	- v3 磁盘存储
	- 恢复还原
- replication controller 控制器
- node 服务器节点
	- kind | minikube 简易集群工具, 推荐使用 minikube
	- kubeadm 高级集群管理工具. 这个是生产级别的工具
	- kubectl 命令行工具, 必备
	- kubelet 配置集群管理工具时, 会自动为node安装
	- kube proxy 配置集群管理工具时, 会自动为node安装
	- container 容器: CRI 接口
	- dashboard: b/s访问体系
	- ingress controller: 七层代理, 代理网关.
	- federation: 跨集群中心多k8s统一管理功能
	- Prometheus: k8s集群监控
	- ELK: k8s日志统一接入平台


## k3s 轻量级k8s集群框架