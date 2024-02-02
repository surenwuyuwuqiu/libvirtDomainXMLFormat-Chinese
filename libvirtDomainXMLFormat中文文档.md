# 资料源：

https://libvirt.org/formatdomain.html#general-metadata

# 部分术语及libvirt目标

为避免所用术语的歧义，以下是一些 libvirt 文档中使用的特定概念：**(部分来源:redhat官方介绍https://www.redhat.com/zh/topics/virtualization)**

- **节点**(node)是单个物理机器
- **虚拟机监控程序**(hypervisor)是一层(种)软件，允许将节点虚拟化为一组虚拟机(创建并运行VM)，这些虚拟机可能具有与节点本身不同的配置。——也叫VMM(Virtual Machine Monitor, 虚拟机监控器)
- **域**(domain)是在由虚拟化监控程序提供的虚拟机上运行的网络实例(或在容器虚拟化情况下为子系统)。

注1：配备了虚拟机监控程序的物理硬件叫做**主机(Host)**，而使用其资源的虚拟机成为**客户机(Guest)**。这些虚拟客户机将计算资源(如 CPU、内存和[存储器](https://www.redhat.com/zh/topics/data-storage))视为一组可进行重新分配的资源。操作员可以控制 CPU、内存、存储器和其他资源的虚拟实例，以便虚拟客户机能在需要时收到所需资源。



libvirt 是目前使用最为广泛的对 KVM 虚拟机进行管理的工具和应用程序接口（API），而且一些常用的虚拟机管理工具（如virsh、virt-install、virt-manager等）和云计算框架平台（如 OpenStack、OpenNebula、Eucalyptus 等）都在底层使用 libvirt 的应用程序接口。

libvirt 是为了更方便地管理平台虚拟化技术而设计的开放源代码的**应用程序接口、守护进程和管理工具**，它不仅提供了对虚拟化客户机的管理，也提供了对虚拟化网络和存储的管理。尽管 libvirt 项目最初是为 Xen 设计的一套API，但是目前对KVM等其他 Hypervisor 的支持也非常的好。libvirt 支持多种虚拟化方案，既支持包括 KVM、QEMU、Xen、VMware、VirtualBox 等在内的平台虚拟化方案，又支持 OpenVZ、LXC 等 Linux 容器虚拟化系统，还支持用户态 Linux（UML）的虚拟化。

libvirt 作为**中间适配层**，让底层 Hypervisor 对上层用户空间的管理工具是可以做到**完全透明**的，因为 libvirt 屏蔽了底层各种 Hypervisor 的细节，为上层管理工具提供了一个**统一的、较稳定的接口（API）**。通过 libvirt，一些用户空间管理工具可以管理各种不同的 Hypervisor 和上面运行的客户机。

![libvirt-manage-hypervisors.jpg](https://opskumu.oss-cn-beijing.aliyuncs.com/images/libvirt-manage-hypervisors.jpg)

libvirt 的管理功能主要包含如下五个部分：

- **域的管理**：包括对节点上的域的各个生命周期的管理，如：启动、停止、暂停、保存、恢复和动态迁移。也包括对多种设备类型的热插拔操作，包括：磁盘、网卡、内存和 CPU，当然不同的 Hypervisor 上对这些热插拔的支持程度有所不同。

- **远程节点的管理**：只要物理节点上运行了 libvirtd 这个守护进程，远程的管理程序就可以连接到该节点进程管理操作，经过认证和授权之后，所有的 libvirt 功能都可以被访问和使用。libvirt 支持多种网络远程传输类型，如 SSH、TCP 套接字、Unix domain socket、支持 TLS 的加密传输等。假设使用最简单的 SSH，则不需要额外配置工作，比如：example.com 节点上运行了 libvirtd，而且允许 SSH 访问，在远程的某台管理机器上就可以用如下的命令行来连接到 example.com 上，从而管理其上的域。

  ```bash
  virsh -c qemu+ssh://root@example.com/system
  ```

- **存储的管理**：任何运行了 libvirtd 守护进程的主机，都可以通过 libvirt 来管理不同类型的存储，如：创建不同格式的客户机镜像（qcow2、raw、qde、vmdk等）、挂载 NFS 共享存储系统、查看现有的 LVM 卷组、创建新的 LVM 卷组和逻辑卷、对磁盘设备分区、挂载 iSCSI 共享存储，等等。当然 libvirt 中，对存储的管理也是支持远程管理的。

- **网络的管理**：任何运行了 libvirtd 守护进程的主机，都可以通过libvirt来管理物理的和逻辑的网络接口。包括：列出现有的网络接口卡，配置网络接口，创建虚拟网络接口，网络接口的桥接，VLAN 管理，NAT 网络设置，为客户机分配虚拟网络接口，等等。

- 提供一个稳定、可靠、高效的**应用程序接口（API）**以便可以完成前面的 4 个管理功能。

libvirt 主要由三个部分组成，它们分别是：**应用程序编程接口（API）库**、一个**守护进程（libvirtd）**和一个**默认命令行管理工具（virsh）**。应用程序接口（API）是为了其他虚拟机管理工具（如 virsh、virt-manager等）提供虚拟机管理的程序库支持。libvirtd 守护进程负责执行对节点上的域的管理工作，在用各种工具对虚拟机进行管理之时，这个守护进程一定要处于运行状态中，而且这个守护进程可以分为两种：一种是 root 权限的libvirtd，其权限较大，可以做所有支持的管理工作；一种是普通用户权限的 libvirtd，只能做比较受限的管理工作。virsh 是 libvirt 项目中默认的对虚拟机管理的一个命令行工具。

![Virtualization management](https://www.redhat.com/rhdc/managed-files/styles/wysiwyg_float/private/virtualization-mgmt-concept-560x478.png?itok=nZFYz0-b)

现在我们可以定义libvirt的目标：**提供一个通用且稳定的层足以安全地管理节点上的域**。

因此，libvirt 应该提供进行管理所需的所有 API，包括： 在虚拟机监控程序对这些操作的支持范围内，对域进行预配、创建、修改、监控、控制、迁移和停止。并非所有虚拟机监控程序都提供相同的操作；但如果某个操作对于至少一个特定虚拟机的域管理是有用的，那么在 libvirt 中提供这个操作是值得的。可以使用 libvirt 同时访问多个节点，但 API仅限于单个节点操作。节点资源操作是libvirt API范围的一部分，用于域的管理和预配，例如接口设置、防火墙规则、存储管理和通用预配API。Libvirt还将提供实现管理策略所需的状态监视API，不仅可以检查域状态，也可以公开本地节点的资源消耗。

这意味着以下子目标：

- 所有API都可以通过安全的API远程携带
- 虽然大多数 API 在虚拟机监控程序或主机操作系统方面都是通用的，但某些 API 可能针对单个虚拟化环境，只要语义对于从域管理角度进行操作是明确的
- API 应该允许高效、干净地执行管理节点上的域所需的所有操作，包括资源预置和设置
- API 不会尝试提供高级虚拟化策略或多节点管理功能，如负载平衡，但API应该足够支持在libvirt之上实现这些功能。
- API的稳定性是一个大问题，libvirt应该将应用程序与虚拟化框架较低级别预期的频繁更改隔离开来
- 被管理的节点可能位于与使用libvirt的hypervisor不同的物理机器上，因此libvirt支持远程访问，但只能通过使用安全协议来实现。
- libvirt将提供API来枚举、监视和使用托管节点上可用的资源，包括CPU、内存、存储、网络和NUMA分区。

因此，libvirt旨在成为更高级别管理工具和专注于单个节点虚拟化的应用程序的构建块(唯一的例外是涉及多个节点的节点之间的域迁移功能)。

# 域XML格式(Domain XML Format)

本节介绍了用于表示域的XML格式，根据运行的域的类型和用于启动域的选项，该格式有多种变体。有关hypervisor特定的详细信息，需参阅驱动程序文档:[driver docs](https://libvirt.org/drivers.html)

注1：域(Domain) = VMInstance = 虚拟机 = guest客户机(个人理解)



## 1. 元素和属性概述(Element and attribute overview)

所有虚拟机所需的根元素都命名为 `domain` 。它有两个属性，`type`指定用于运行域的hypervisor。允许的值是特定于驱动程序的，包括"xen"、"kvm"、"hvf"(自8.1.0和QEMU 2.12起)、"QEMU"和"lxc"。第二个属性是`id`，它是运行的客户机(GUEST MACHINE，物理机上运行的虚拟机实例)的唯一整数标识符(unique integer identifier， UID)。非活动计算机没有id值。



### 1.1 通用元数据(General metadata)

```xml
<domain type='kvm' id='1'>
  <name>MyGuest</name>
  <uuid>4dea22b3-1d52-d8f3-2516-782e98ab3fa0</uuid>
  <genid>43dc0cf8-809b-4adb-9bea-a9abb5f3d90e</genid>
  <title>A short description - title - of the domain</title>
  <description>Some human readable description</description>
  <metadata>
    <app1:foo xmlns:app1="http://app1.org/app1/">..</app1:foo>
    <app2:bar xmlns:app2="http://app1.org/app2/">..</app2:bar>
  </metadata>
  ...
```

`name`

​	`name`元素的内容提供虚拟机的短名称。该名称应仅由字母数字字符组成，并且要求在单个主机范围内是唯一的。它通常用于构成存储持久配置文件的文件名。 从0.0.1开始

`uuid`

​	`uuid`元素的内容为虚拟机提供全局唯一标识符(Universally Unique IDentifier)。该格式必须符合 RFC 4122规范，例如 3e3fce45-4f53-4fa7-bb32-11f34168b82b。如果在定义/创建新机器时省略，则会生成随机 UUID。还可以通过[SMBIOS 系统信息](https://libvirt.org/formatdomain.html#smbios-system-information)(1.3章节)规范提供 UUID。从0.0.1开始，sysinfo从0.8.7开始

`genid`

​	自从4.4.0版本，`genid`元素可以用来添加虚拟机生成ID，该ID使用与`uuid`相同的格式公开128位加密随机整数值标识符，称为全局唯一标识符(GUID)。该值用于帮助在虚拟机重新执行已经执行过的操作时，通知guest OS(虚拟机中的操作系统)，例如：

- VM starts executing a snapshot—VM 开始执行快照

- VM is recovered from backup—虚拟机已从备份中恢复

- VM is failover in a disaster recovery environment—

  VM 在灾难恢复环境中进行故障转移

- VM is imported， copied， or cloned—虚拟机已导入、复制或克隆

​	guest OS会注意到这种变化，然后根据需要采取适当的反应，例如将其分布式数据库的副本标记为脏数据，重新初始化其随机数生成器等。

​	libvirt XML解析器会接受提供的GUID值，或者只是使用 <genid/>，在这种情况下，将生成一个GUID并保存在XML中。对于上述的转换，libvirt会在重新执行之前更改GUID。

`titile`

​	可选的元素`title`为域的简短描述提供了空间。标题不应包含任何换行符。从0.9.10起。

`description`

​	`description`元素的内容提供了虚拟机的可读描述，libvirt 不会以任何方式使用此数据，它可以包含用户想要的任何信息。从0.7.2开始

`metadata`

​	应用程序可以使用`metadata`节点以XML节点/树的形式存储自定义元数据。应用程序必须在其XML节点/树上使用自定义名称空间，每个名称空间只有一个顶级元素(如果应用程序需要结构，则它们的命令空间元素应该具有子元素)。自0.9.10起



### 1.2 操作系统启动(Operating system booting)

启动虚拟机的方法有很多种，每种方法都有各自的优点和缺点



#### 1.2.1 BIOS引导加载程序(BIOS bootloader)

支持完全虚拟化的虚拟机监控程序可通过 BIOS 进行引导。在这种情况下，BIOS 有一个引导顺序优先级(软盘、硬盘、CDROM、网络)，用于确定在哪里获取/查找引导映像。

```xml
<!-- Xen with fullvirt loader -->
...
<os>
  <type>hvm</type>
  <loader>/usr/lib/xen/boot/hvmloader</loader>
  <boot dev='hd'/>
</os>
...

<!-- QEMU with default firmware， serial console and SMBIOS -->
...
<os>
  <type>hvm</type>
  <boot dev='cdrom'/>
  <bootmenu enable='yes' timeout='3000'/>
  <smbios mode='sysinfo'/>
  <bios useserial='yes' rebootTimeout='0'/>
</os>
...

<!-- QEMU with UEFI manual firmware and secure boot -->
...
<os>
  <type>hvm</type>
  <loader readonly='yes' secure='yes' type='pflash'>/usr/share/OVMF/OVMF_CODE.fd</loader>
  <nvram template='/usr/share/OVMF/OVMF_VARS.fd'>/var/lib/libvirt/nvram/guest_VARS.fd</nvram>
  <boot dev='hd'/>
</os>
...

<!-- QEMU with UEFI manual firmware， secure boot and with NVRAM type 'file'-->
...
<os>
  <type>hvm</type>
  <loader readonly='yes' secure='yes' type='pflash'>/usr/share/OVMF/OVMF_CODE.fd</loader>
  <nvram type='file' template='/usr/share/OVMF/OVMF_VARS.fd'>
    <source file='/var/lib/libvirt/nvram/guest_VARS.fd'/>
  </nvram>
  <boot dev='hd'/>
</os>
...

<!-- QEMU with UEFI manual firmware， secure boot and with network backed NVRAM'-->
...
<os>
  <type>hvm</type>
  <loader readonly='yes' secure='yes' type='pflash'>/usr/share/OVMF/OVMF_CODE.fd</loader>
  <nvram type='network'>
    <source protocol='iscsi' name='iqn.2013-07.com.example:iscsi-nopool/0'>
      <host name='example.com' port='6000'/>
      <auth username='myname'>
        <secret type='iscsi' usage='mycluster_myname'/>
      </auth>
    </source>
  </nvram>
  <boot dev='hd'/>
</os>
...

<!-- QEMU with automatic UEFI firmware and secure boot -->
...
<os firmware='efi'>
  <type>hvm</type>
  <loader secure='yes'/>
  <boot dev='hd'/>
</os>
...

<!-- QEMU with automatic UEFI stateless firmware for AMD SEV -->
...
<os firmware='efi'>
  <type>hvm</type>
  <loader stateless='yes'/>
  <boot dev='hd'/>
</os>
...
```

`firmware` 固件

​	`firmware`属性允许管理应用程序自动填充<loader/>和<nvram/>元素，并可能启用所选固件所需的一些功能。可接受的值为`bios`和`efi`。选择过程在指定位置扫描描述已安装固件映像的文件，并使用满足域要求的最特定的文件。按偏好顺序(从一般到最具体)的位置为：

```
/usr/share/qemu/firmware
/etc/qemu/firmware
$XDG_CONFIG_HOME/qemu/firmware
```

​	有关更多信息，请参阅QEMU存储库中`docs/interop/firmware.json`中描述的固件元数据规范。普通用户无需关心。自5.2.0(仅限QEMU和KVM)对于VMware客户，当客户使用UEFI时，此设置为`efi`，而在使用BIOS时未设置。自5.3.0起(VMware ESX和Workstation/Player)

注1：UEFI，全称为"统一的可扩展固件接口"(Unified Extensible Firmware Interface)，是一种用于计算机启动的开放标准。它取代了传统的BIOS(Basic Input/Output System)固件，作为现代计算机系统的引导方式。可以提供更强大的虚拟化能力，支持更高效的虚拟机管理和部署。

`type`

​	`type`元素的内容指定要在虚拟机中启动(引导)的操作系统的类型。`hvm`表示操作系统是设计为在裸金属上运行的，因此需要完全虚拟化。`linux`(命名不佳！)指的是一个支持Xen3 hypervisor(虚拟机监控程序)客户机ABI的操作系统。还有两个可选属性，`arch`指定要虚拟化的CPU架构，`machine`指的是机器类型。Capabilities XML提供了有关这些属性的允许值的详细信息。如果省略了`arch`，那么对于大多数hypervisor驱动程序，将选择主机本机架构。然而，对于`test`、`ESX`和`VMWare`虚拟机监控程序驱动程序，即使在`x86_64`主机上，也将始终选择`i686` 架构。自0.0.1起

注1：i686 架构是一种基于 Intel 80386 微处理器的 x86 架构的扩展。它是 x86 架构的一部分，通常被称为 "IA-32" 架构，是最常见的个人计算机和服务器体系结构之一。i686 架构支持 32 位处理器操作，包括各种指令集和功能，如浮点运算、多媒体扩展等。i686 架构是在原始的 8086 架构基础上发展起来的，通过多代处理器的改进和扩展而形成。这些改进包括增加寄存器、更高的时钟频率、更大的内存访问能力等。i686 架构也支持多任务处理和虚拟内存管理，使操作系统和应用程序能够更高效地运行。

`firmware`

​	仅自 7.2.0 QEMU/KVM 起

​	使用固件自动选择时，固件中会启用不同的功能。功能列表可用于限制应为虚拟机自动选择哪些固件。可以使用零个或多个`feature`元素指定功能列表。在选择固件时，Libvirt将只考虑列出的功能，而忽略其余功能。

​	`feature`

​		强制列表属性：

- `enabled`(可接受的值为`yes`和`no`)用于告知libvirt是否必须在自动选择的固件中启用该功能
- `name`功能的名称，功能列表：
  - `enrolled-keys`(已注册密钥)所选nvram模板是否已注册默认证书。具有安全引导(启动)功能但没有注册密钥的固件也将成功引导未签名的二进制文件。仅对具有安全启动功能的固件有效。
  - `secure-boot`固件是否实现UEFI安全启动功能

`loader`加载程序

​	可选的`loader`标记是指由绝对路径指定的固件 blob，用于协助域创建过程。它由 Xen 完全虚拟化域以及为 QEMU/KVM 域设置 QEMU BIOS 文件路径使用。Xen 自 0.1.0 起，QEMU/KVM 自 0.9.12 起。然后， 自 1.2.8 起，元素可以有两个可选属性：`readonly`(接受的值为yes和no)以反映镜像应该可写或只读。第二个属性`type`接受值`rom`和`pflash`。它告诉hypervisor应该映射到客户机内存中的哪个位置。例如，如果加载程序路径指向 UEFI 映像，则类型应为`pflash`。此外，某些固件可能会实现安全启动功能。属性`secure`可用于告诉hypervisor固件具有安全启动功能。它不能用于启用或禁用固件中的功能本身。 从 2.1.0 开始。如果加载程序被标记为只读，则使用 UEFI 时，假定将有一个可写的 NVRAM 可用。然而，在某些情况下，加载程序可能需要在没有任何 NVRAM 的情况下运行，从而在关闭时丢弃任何配置更改。`stateless`(无状态)标志(自 8.6.0) 可用于控制此行为，当设置为`yes`时，将永远不会创建 NVRAM。

​	启用固件自动选择后，`format`属性可用于告诉 libvirt 仅考虑特定格式的固件版本。支持的值为`raw`和`qcow2`。 自 9.2.0 起(仅限 QEMU)

`nvram`----非易失性随机存取存储器(Non-Volatile Random-Access Memory)断电后数据保持不变

​	一些 UEFI 固件可能想要使用非易失性存储器来存储一些变量。在主机中，这表示为文件，并且文件的绝对路径存储在该元素中。此外，当域启动时，libvirt 会复制`qemu.conf`中定义的所谓主 NVRAM 存储文件。如果需要，模板属性可用于配置文件中主 NVRAM 存储的每个域覆盖映射。请注意，对于瞬态域，如果 NVRAM 文件是由 libvirt 创建的，那么该文件将被保留下来，由管理应用程序负责保存和删除文件(如果需要持久化)。从1.2.8开始

​	从 8.5.0 开始，元素可以具有`type`属性(接受值`file`、`block`和`network` )，在这种情况下，NVRAM 存储由<source>子元素描述，其语法与`disk`的源相同 。请参阅1.20.1[Hard drives, floppy disks, CDROMs](https://libvirt.org/formatdomain.html#hard-drives-floppy-disks-cdroms)

​	**注意：** `network`支持的 NVRAM 变量不是从`template`实例化的，用户有责任提供有效的 NVRAM 映像。

​	该元素支持`format`属性，该属性与<loader>元素的同名属性具有相同的语义。 自 9.2.0 起(仅限 QEMU)

​	如果加载器被标记为无状态，则提供此元素是无效的。

**注1：**瞬态域(Transient Domain)是指在虚拟化环境中创建的一种临时性的虚拟机实例。与持久性域(Persistent Domain)不同，瞬态域在虚拟机关闭后通常不会保留其状态或数据，而是在关闭后会被删除或重置。主要用于临时性任务和测试目的。

`boot` 启动/引导

​	`dev`属性采用值"fd"、"hd"、"cdrom"或"network"之一，用于指定要考虑的下一个引导设备。`boot`元素可以重复多次，以设置启动设备的优先级列表，依次尝试不同的引导设备。相同类型的多个设备根据其目标进行排序，同时保留总线的顺序。定义域后，libvirt(通过 virDomainGetXMLDesc)返回的 XML 配置按排序顺序列出设备。排序后，第一个设备将被标记为可启动。因此，例如，一个配置为从"hd"引导的域，分配了vdb、hda、vda和hdc磁盘给它，将从vda引导(按照排序后的列表顺序为vda、vdb、hda、hdc)。具有 hdc、vda、vdb 和 hda 磁盘的类似域将从 hda 启动(排序的磁盘为：hda、hdc、vda、vdb)。以所需的方式进行配置可能很棘手，这就是为什么引入了每个设备的引导元素(请参阅1.20.1 [Hard drives, floppy disks, CDROMs](https://libvirt.org/formatdomain.html#hard-drives-floppy-disks-cdroms), [Network interfaces](https://libvirt.org/formatdomain.html#network-interfaces), 以及下面的1.20.8 [Host device assignment](https://libvirt.org/formatdomain.html#host-device-assignment) 部分)，它们是提供完全控制引导顺序的首选方式。`boot`元素和每个设备引导元素是互斥的 。从 0.1.3 开始，从 0.8.8 开始 per-device boot

`smbios`

​	如何填充客户端中可见的 SMBIOS 信息。必须指定`mode`属性，并且可以是"emulate" "模拟"(让hypervisor虚拟机监控程序生成所有值)、"host"(除了 UUID， 从主机的 SMBIOS 值复制所有块 0 和块 1；[virConnectGetSysinfo](https://libvirt.org/html/libvirt-libvirt-host.html#virConnectGetSysinfo) 调用 可以用于查看复制了哪些值)或"sysinfo"(使用 [SMBIOS System Information](https://libvirt.org/formatdomain.html#smbios-system-information)元素中的值)。如果未指定，则使用虚拟机监控程序默认值。从0.8.7开始

到目前为止，BIOS/UEFI 配置选项已经足够通用，可以被大多数(如果不是全部)固件实现。然而，从现在开始，并非每个设置对所有固件都有意义。例如， `rebootTimeout`对于 UEFI 没有意义，`useserial`可能无法在不向串行线路上产生任何输出的 BIOS 固件上使用，等等。此外，固件通常不会为 libvirt(或用户)公开其功能以供检查。并且其功能集合可能会随着每个新版本的发布而改变。因此，建议用户在生产环境之前先尝试他们使用的设置，以确保他们可靠。

`bootmenu`

​	是否在虚拟机启动时启用交互式启动菜单提示。`enable`属性可以是"yes"或"no"。如果未指定，将使用hypervisor的默认值。自0.8.3版本起，还增加了`timeout`属性，它表示启动菜单等待超时的毫秒数。允许的值是范围在[0， 65535]内的数字，除非`enable`设置为"yes"，否则将被忽略。自1.2.8版本起。

`bios`

​	该元素具有属性`userserial`，其可能值为`yes`或 `no`。它启用或禁用串行图形适配器，允许用户在串行端口上查看 BIOS 消息。因此，需要定义[串行端口](https://libvirt.org/formatdomain.html#serial-port) 。从 0.9.4 开始。从 0.10.2(仅限 QEMU)开始，还有另一个属性，`rebootTimeout`用于控制在启动失败的情况下客户机是否应重新启动以及在多长时间后重新启动(根据 BIOS)。该值以毫秒为单位，最大值为65535，特殊值-1将禁用重新启动。



#### 1.2.2 主机引导加载程序(Host bootloader)

采用半虚拟化的Hypervisor通常不会模拟BIOS，而是由主机负责启动操作系统。这可以使用主机中的伪引导加载程序来提供为客户机选择内核的接口。一个例子是使用Xen的`pygrub`。Bhyve hypervisor还使用主机引导加载程序，可以是`bhyveload`或`grub Bhyve`。

```xml
...
<bootloader>/usr/bin/pygrub</bootloader>
<bootloader_args>--append single</bootloader_args>
...
```

`bootloader`引导加载程序

​	`bootloader`元素的内容为在主机OS中可执行的引导加载程序提供了完全限定的路径。将运行此引导加载程序来选择要引导的内核。引导加载程序所需的输出取决于所使用的hypervisor。自0.1.0

`bootloader_args`

​	可选的`bootloader_args`元素允许将命令行参数传递给bootloader。自0.2.3



#### 1.2.3 直接内核引导(Direct kernel boot)

在安装新的guest OS时，直接从主机操作系统中存储的内核和initrd(初始化内存盘)启动通常很有用，这样可以将命令行参数直接传递给安装程序。此功能通常可用于半虚拟化和完全虚拟化的guest OS。

```xml
...
<os>
  <type>hvm</type>
  <loader>/usr/lib/xen/boot/hvmloader</loader>
  <kernel>/root/f8-i386-vmlinuz</kernel>
  <initrd>/root/f8-i386-initrd</initrd>
  <cmdline>console=ttyS0 ks=http://example.com/f8-i386/os/</cmdline>
  <dtb>/root/ppc.dtb</dtb>
  <acpi>
    <table type='slic'>/path/to/slic.dat</table>
  </acpi>
</os>
...
```

`type`

​	该元素具有与前面 [BIOS bootloader](https://libvirt.org/formatdomain.html#bios-bootloader)部分中描述的语义相同的语义。

`loader`

​	该元素具有与前面 [BIOS bootloader](https://libvirt.org/formatdomain.html#bios-bootloader)部分中描述的语义相同的语义。

`kernel`

​	此元素的内容指定主机操作系统中内核镜像的完全限定路径。

`initrd`

​	此元素的内容指定主机操作系统中(可选)ramdisk镜像的完全限定路径

`cmdline`

​	此元素的内容指定在启动时传递给内核(或安装程序)的参数。这通常用于指定备用主控制台(例如串行端口)或安装媒体源/kickstart文件

`dtb`

​	此元素的内容指定主机操作系统中(可选)设备树二进制(dtb，device table binary)映像的完全限定路径。自1.0.4

`acpi`

​	`table`元素包含ACPI表的完全限定路径。type属性包含ACPI表类型(目前只支持`slic`)自1.3.5(QEMU)自5.9.0(Xen)



#### 1.2.4 容器引导(Container boot)

当使用基于容器的虚拟化而不是内核/引导映像来引导域时，需要使用`init`元素的init二进制文件路径。默认情况下，这将在没有参数的情况下启动。要指定初始argv，请使用`initarg`元素，根据需要重复多次。`cmdline`元素(如果设置)将用于提供与`/proc/cmdline`等效的值，但不会影响init argv。

要设置环境变量，请使用`initenv`元素，每个变量一个。

要为init设置自定义工作目录，请使用`initdir`元素。

要以给定用户或组的身份运行init命令，请分别使用`inituser`或`initgroup`元素。这两个元素都可以提供用户(resp.group)id或名称。在用户或组id前面加一个`+`将强制将其视为一个数值。如果没有这一点，它将首先作为用户名或组名进行尝试。

```xml
<os>
  <type arch='x86_64'>exe</type>
  <init>/bin/systemd</init>
  <initarg>--unit</initarg>
  <initarg>emergency.service</initarg>
  <initenv name='MYENV'>some value</initenv>
  <initdir>/my/custom/cwd</initdir>
  <inituser>tester</inituser>
  <initgroup>1000</initgroup>
</os>
```

如果要启用用户命名空间，请设置`idmap`元素。`uid`和`gid`元素有三个属性：

`start`

​	容器中的第一个用户ID。它必须是"0"。

`target`

​	容器中的第一个用户ID将映射到主机中的此目标用户ID。

`count`

​	容器中允许映射到主机用户的用户数。

```xml
<idmap>
  <uid start='0' target='1000' count='10'/>
  <gid start='0' target='1000' count='10'/>
</idmap>
```



### 1.3 SMBIOS系统信息(SMBIOS  System Information)

一些hypervisor允许控制向虚拟机呈现什么系统信息(例如，hypervisor可以填充SMBIOS字段，并通过虚拟机内的`dmidecode`命令进行检查)。可选的 `sysinfo `元素涵盖了所有这些信息类别。自0.8.7

```xml
...
<os>
  <smbios mode='sysinfo'/>
  ...
</os>
<sysinfo type='smbios'>
  <bios>
    <entry name='vendor'>LENOVO</entry>
  </bios>
  <system>
    <entry name='manufacturer'>Fedora</entry>
    <entry name='product'>Virt-Manager</entry>
    <entry name='version'>0.9.4</entry>
  </system>
  <baseBoard>
    <entry name='manufacturer'>LENOVO</entry>
    <entry name='product'>20BE0061MC</entry>
    <entry name='version'>0B98401 Pro</entry>
    <entry name='serial'>W1KS427111E</entry>
  </baseBoard>
  <chassis>
    <entry name='manufacturer'>Dell Inc.</entry>
    <entry name='version'>2.12</entry>
    <entry name='serial'>65X0XF2</entry>
    <entry name='asset'>40000101</entry>
    <entry name='sku'>Type3Sku1</entry>
  </chassis>
  <oemStrings>
    <entry>myappname:some arbitrary data</entry>
    <entry>otherappname:more arbitrary data</entry>
  </oemStrings>
</sysinfo>
<sysinfo type='fwcfg'>
  <entry name='opt/com.example/name'>example value</entry>
  <entry name='opt/com.coreos/config' file='/tmp/provision.ign'/>
</sysinfo>
...
```

`sysinfo`元素有一个强制属性类型，用于确定子元素的布局，支持的值为：

​	`smbios`

​	子元素指定特定的SMBIOS值，如果与`os`元素的`smbios`子元素(请参见1.2[Operating system booting](https://libvirt.org/formatdomain.html#operating-system-booting))结合使用，将影响虚拟机。`sysinfo`的每个子元素命名一个SMBIOS块，在这些元素内部可以是一个`entry`元素列表，描述块内的字段。以下块block和条目entry识别为：

​		`bios`

​			这是 SMBIOS的块0，条目名称来自：

​			`vendor` 

​				BIOS供应商名称

​			`version`

​				BIOS版本

​			`date`

​				BIOS发布日期。如果提供，则为mm/dd/yy或mm/dd/yyyy格式。如果字符串的年份部分是两位数字，则假定年份为19yy。

​			`release`

​				系统BIOS主要和次要版本号值连接在一起，作为一个由句点分隔的字符串，例如10.22。

​	`system`

​		这是SMBIOS块1，条目名称来自：

​		`manufacturer`

​			BIOS制造商

​		`product`

​			产品名称

​		`version`

​			产品版本

​		`serial`

​			序列号

​		`uuid`

​			通用唯一ID号。如果此条目与顶级uuid元素一起提供(请参见 1.1[General metadata](https://libvirt.org/formatdomain.html#general-metadata))，则这两个值必须匹配。

​		`sku`

​			识特定配置的SKU编号。

​		`family`

​			识别特定计算机所属的系列。

​	`baseBoard`

​		这是SMBIOS的块2.这个元素可以重复多次来描述所有的基板；然而，并不是所有的hypervisor都必须支持重复。该元素可以具有以下子元素：

​		`manufacturer`

​			BIOS制造商

​		`product`

​			产品名称

​		`version`

​			产品的版本

​		`serial`

​			序列号

​		`asset`

​			资产标签

​		`location`

​			机箱中的位置

​		注意：错误提供的bios、系统或基板块条目将被忽略，不会出现错误。除了uuid验证和日期格式检查之外，所有值都作为字符串传递给hypervisor驱动程序。

​	`chassis`

​		自4.1.0起，这是SMBIOS的块3，条目名称来自：

​		`manufacturer`

​			Chassis机箱(底盘)制造商

​		`version`

​			Chassis的版本

​		`serial`

​			序列号

​		`asset`

​			资产标签

​		`SKU`

​			SKU编号

​	`oemStrings`

​		这是SMBIOS的块11。这个元素应该出现一次，并且可以有多个`entry`子元素，每个子元素都提供任意的字符串数据。条目中可以提供的数据没有限制，但是，如果数据打算由客户机的应用程序使用，建议将应用程序名称用作字符串中的前缀。(自4.1.0起)

`fwcfg`

​	一些hypervisor提供了统一的方式来调整固件如何配置自己，或者可能包含要为guest OS安装的表，例如引导顺序、ACPI、SMBIOS等。

​	它甚至允许用户定义自己的配置Blobs。在QEMU的情况下，这些会出现在域的sysfs下(如果客户机内核启用了FW_CFG_sysfs配置选项)，在`/sys/ffirmware/qemufw_CFG`下。请注意，无论<os/>下的<smbios/>模式如何，这些值都适用。自6.5.0

​	**请注意，由于数据槽数量有限，强烈建议使用fwcfg，而应使用<oemStrings/>。**

```xml
<sysinfo type='fwcfg'>
  <entry name='opt/com.example/name'>example value</entry>
  <entry name='opt/com.example/config' file='/tmp/provision.ign'/>
</sysinfo>
```

​	`sysinfo`元素可以有多个子元素。然后，每个元素都有强制的`name`属性，该属性定义blob的名称，并且必须以`opt/`开头，为了避免与其他名称冲突，建议采用`opt/$RFQDN/$name`的形式，其中`$RFQDN`是 控制的反向完全限定域名。然后，元素可以包含值(直接设置blob值)，也可以包含`file`属性(从文件中设置blob的值)。



### 1.4 CPU分配(CPU Allocation)

```xml
<domain>
  ...
  <vcpu placement='static' cpuset="1-4，^3，6" current="1">2</vcpu>
  <vcpus>
    <vcpu id='0' enabled='yes' hotpluggable='no' order='1'/>
    <vcpu id='1' enabled='no' hotpluggable='yes'/>
  </vcpus>
  ...
</domain>
```

`vcpu`

​	这个元素的内容定义了为guest OS分配的虚拟 CPU 的最大数量，必须介于 1 到 hypervisor 支持的最大数量之间。

​	`cpuset`

​		可选属性`cpuset`是一个以逗号分隔的物理CPU编号列表，默认情况下域进程和虚拟CPU可以固定到该列表。(注意：域进程和虚拟CPU的固定策略可以由`cputune`单独指定。如果指定了`cputune`的属性`emulatorpin`，则此处`vcpu`指定的`cpuset`将被忽略。同样，对于指定了`vcpupin`的虚拟CPU，此处`cpuset`指定的`cpuset`将被忽视。对于没有指定`vcpupin`此处由`cpuset`指定的物理CPU)。该列表中的每个元素要么是一个CPU编号，要么是一系列CPU编号，或者是一个插入符号，后跟一个要从以前的范围中排除的CPU编号。自0.4.4起

​	`current`

​		可选属性`current`可以用来指定是否应启用少于最大数量的虚拟CPU。自0.8.5起

​	`placement`

​		可选属性`placement`可以用于指示域进程的CPU放置模式。该值可以是"static"或"auto"，但如果指定了`cpuset`，则默认`numatune`或"static"的`placement`方式)。使用"auto"表示域进程将通过查询numad固定到查询节点集，并且如果指定了属性`cpuset`的值，则会忽略该值。如果没有指定`cpuset`和`placement`，或者`placement`是"static"，但没有指定`cpuset`，则域进程将固定到所有可用的物理CPU。自0.9.11起(仅限QEMU和KVM)

`vcpus`

​	`vcpus`元素允许控制各个vCPU的状态。`id`属性指定libvirt在其他地方使用的vCPU id，如vCPU固定、调度程序信息和NUMA分配。请注意，在某些情况下，guest中看到的vCPU ID可能与libvirt ID不同。有效ID是从0到vCPU元素设置的最大vCPU计数减去1。`enabled`属性允许控制vCPU的状态。有效值为`yes`和`no`。`hotpluggable`热插拔控制在启动时启用CPU的情况下，是否可以热插拔给定的vCPU。请注意，所有禁用的vCPU都必须是可热插拔的。有效值为`yes`和`no`。`order`允许指定添加在线vCPU的顺序。对于需要同时插入多个vCPU的hypervisor/平台，顺序可能会在需要同时启用的所有vCPU中重复。不需要指定顺序，然后以任意顺序添加vCPU。如果使用排序信息，则必须将其用于所有在线vCPU。Hypervisor可以在某些操作期间清除或更新排序信息，以确保配置有效。请注意，hypervisor可能会创建与启动vCPU不同的可热插拔vCPU，因此可能需要进行特殊初始化。Hypervisor可能要求在启动时启用的不可热插拔的vCPU从ID 0开始群集。还可能要求vCPU 0始终存在并且不可热插拔。请注意，为单个CPU提供状态可能是支持可寻址vCPU热插拔所必需的，并且并非所有hypervisor都支持此功能。对于QEMU，需要以下条件。vCPU 0需要启用且不可热插拔。在PPC64上，同一核心中的vCPU也需要启用。启动时出现的所有不可热插拔CPU都需要在vCPU 0之后进行分组。自2.2.0起(仅限QEMU)



### 	1.5 IOTreads分配(IOTreads Allocation)

IOThreads是受支持磁盘设备的专用事件循环线程，用于执行块I/O请求，以提高可扩展性，尤其是在具有许多LUN的SMP主机/客户机上。自1.2.8起(仅限QEMU)

注1：SMP(Symmetric Multiprocessing)主机是指具有多个处理器核心(CPU核心)的计算机系统，这些核心可以同时运行多个任务或线程。在SMP系统中，每个CPU核心都可以访问相同的内存和设备，并且它们共享相同的系统总线和其他资源。这种设计允许多个CPU核心共同处理任务，提高系统的性能和吞吐量。SMP主机通常用于高性能计算、服务器和虚拟化环境中。

```xml
<domain>
  ...
  <iothreads>4</iothreads>
  ...
</domain>
```

```xml
<domain>
  ...
  <iothreadids>
    <iothread id="2"/>
    <iothread id="4"/>
    <iothread id="6"/>
    <iothread id="8" thread_pool_min="2" thread_pool_max="32">
      <poll max='123' grow='456' shrink='789'/>
    </iothread>
  </iothreadids>
  <defaultiothread thread_pool_min="8" thread_pool_max="16"/>
  ...
</domain>
```

`iothreads`

​	这个可选元素的内容定义了分配给域的IOThreads数量，供支持的目标存储设备使用。每个主机CPU应该只有1或2个IOThreads。可能有多个受支持的设备分配给每个IOThread。自1.2.8起

`iothreadids`

​	可选的`iothreadids`元素提供了专门定义域的IOThread ID的功能。默认情况下，IOThread ID按顺序编号，从1开始到为域定义的`iothreads`数量。`id`属性用于定义IOThread id。`id`属性必须是大于0的正整数。如果定义的`iothreadids`少于为域定义的`iothreads`，那么libvirt将从1开始顺序填充`iothreadids`，避免任何预定义的id。1.2.15起元素有两个可选属性`thread_pool_min`和`thread_pool _max`，这两个属性允许为给定IOThread设置工作线程数的下限和上限。前者可以是零值，而后者则不能。自8.5.0起自9.4.0起，在iothread切换回事件之前，可以使用可选的子元素`pool`轮询覆盖iothread的hypervisor默认轮询间隔。可选属性`max`设置了轮询应使用的最长时间(以纳秒为单位)。将`max`设置为0将禁用轮询。属性`grow`和`shrink`覆盖(override)(或当设置为0时，如果设置的间隔被认为不足或太大，则禁用增加/减少轮询间隔的默认步骤)

`defaultiothread`

​	此元素表示hypervisor中的默认事件循环，其中处理来自未分配给特定IOThread的设备的I/O请求。然后，元素可以具有`thread_pool_min`和/或`thread_pool _max`属性，这些属性控制默认事件循环的工作线程数的下限和上限。Emulator可能是多线程的，并根据需要生成所谓的工作线程。通常，这两个属性都不应该设置(让模拟器使用自己的默认值)，除非模拟器在实时工作负载中运行，因此无法承受生成新工作线程所需的不可预测时间。自8.5.0



### 1.6 CPU调整/优(CPU Tuning)

```xml
<domain>
  ...
  <cputune>
    <vcpupin vcpu="0" cpuset="1-4，^2"/>
    <vcpupin vcpu="1" cpuset="0，1"/>
    <vcpupin vcpu="2" cpuset="2，3"/>
    <vcpupin vcpu="3" cpuset="0，4"/>
    <emulatorpin cpuset="1-3"/>
    <iothreadpin iothread="1" cpuset="5，6"/>
    <iothreadpin iothread="2" cpuset="7，8"/>
    <shares>2048</shares>
    <period>1000000</period>
    <quota>-1</quota>
    <global_period>1000000</global_period>
    <global_quota>-1</global_quota>
    <emulator_period>1000000</emulator_period>
    <emulator_quota>-1</emulator_quota>
    <iothread_period>1000000</iothread_period>
    <iothread_quota>-1</iothread_quota>
    <vcpusched vcpus='0-4，^3' scheduler='fifo' priority='1'/>
    <iothreadsched iothreads='2' scheduler='batch'/>
    <cachetune vcpus='0-3'>
      <cache id='0' level='3' type='both' size='3' unit='MiB'/>
      <cache id='1' level='3' type='both' size='3' unit='MiB'/>
      <monitor level='3' vcpus='1'/>
      <monitor level='3' vcpus='0-3'/>
    </cachetune>
    <cachetune vcpus='4-5'>
      <monitor level='3' vcpus='4'/>
      <monitor level='3' vcpus='5'/>
    </cachetune>
    <memorytune vcpus='0-3'>
      <node id='0' bandwidth='60'/>
    </memorytune>

  </cputune>
  ...
</domain>
```

`cputune`

​	可选的`cputune`元素提供有关域的CPU可调参数的详细信息。注意：对于qemu驱动程序，启动模拟器后将遵守可选的`vcpupin`和`emulatorpin`固定设置，并考虑NUMA约束。这意味着预计在此期间将使用主机的其他物理CPU，这将反映在`virsh cpu-stats`的输出中。

`vcpuin`

​	可选的`vcpupin`元素指定域vCPU将被固定到主机的哪个物理CPU上。如果省略此项，并且没有指定元素`vcpu`的属性`cpuset`，则默认情况下，vCPU将固定到所有物理CPU。它包含两个必需的属性，属性`vcpu`指定vCPU id，属性`cpuset`与元素`vcpu`的属性`cpuset`相同。自0.9.0起支持QEMU驱动程序，自0.9.1起支持Xen驱动程序

`emulatorpin`

​	可选的`emulatorpin`元素指定将"模拟器"(不包括vCPU或iothreads的域的子集)固定到哪一个主机物理CPU。如果省略此项，并且没有指定元素`vcpu`的属性`cpuset`，则默认情况下，"模拟器"固定到所有物理CPU。它包含一个必需的属性`cpuset`，用于指定要固定到的物理CPU。

`iothreadpin`

​	可选的`iothreadpin`元素指定IOThreads固定在哪一个物理机CPU上。如果省略此项并且属性`cpuset`的元素`vcpu`没有指定，则默认情况下，IOThreads将固定到所有物理CPU上。包含两个必须属性，属性`iothread`指定了 IOThead的ID，属性`cpuset`·指定要固定到的物理CPU。请参阅记录iothread有效值的IOThreads分配部分。自1.2.9

`shares`

​	可选`share`元素指定域的比例加权份额。如果忽略此项，则默认为操作系统提供的默认值。注意，该值没有单位，它是基于其他虚拟机设置的相对度量，例如，配置为值2048的虚拟机将获得两倍于配置为值1024的虚拟机的CPU时间。使用cgroups v1时，该值应在[2,262144]的范围内，使用cgroups v2时，应在[1,10000]的范围中。自0.9.0

`period`

​	可选的`period`元素指定强制间隔(单位：微秒)。在此期间内，域的每个vCPU将不允许消耗超过配额的运行时。该值应在[1000,1000000]的范围内。值为0的句点表示没有值。自0.9.4起仅支持QEMU驱动程序，自0.9.10起仅支持LXC

`quota`  配额

​	可选的`quota`元素指定允许的最大带宽(单位：微秒)。`quota`为负值的域表示该域对vCPU线程具有无限带宽，这意味着它不受带宽控制。该值应在[1000,17592186044415]范围内或小于0。值为0的配额表示没有值。 可以使用此功能来确保所有vCPU以相同的速度运行。自0.9.4起仅支持QEMU驱动程序，自0.9.10起仅支持LXC

`global_period`

​	可选的`global_period`元素指定整个域的强制CFS调度程序间隔(单位：微秒)，而`period`则强制每个vCPU的间隔。该值应在[1000,1000000]之间。值为0的`global_period`表示没有值。自1.3.3起仅支持QEMU驱动程序

`global_quota`

​	可选的`global_period`元素指定整个域的强制CFS调度程序间隔(单位：微秒)，而period则强制每个vCPU的间隔。该值应在[1000、1000000之间]。值为0的`global_period`表示没有值。自1.3.3起仅支持QEMU驱动程序

`emulator_period`

​	可选的`emulator_period`元素指定强制间隔(单位：微秒)。在`emulator_period`内，域的模拟器线程(不包括vCPU)将不允许消耗超过`emulator_quota`的运行时。该值应在[1000,1000000]的范围内。值为0的句点表示没有值。0.10.0之后仅支持QEMU驱动程序

`emulator_quota`

​	可选的`emulator_quota`元素指定域的仿真器线程(不包括vCPU的线程)允许的最大带宽(单位：微秒)。emulator_quota为任何负值的域表示该域对仿真器线程(不包括vCPU的线程)具有无限带宽，这意味着它不受带宽控制。该值应在[1000, 17592186044415]范围内或小于0。值为0的配额表示没有值。0.10.0之后仅支持QEMU驱动程序

`iothread_period`

​	可选的`iothread_period`元素指定 IOThread 的执行间隔(单位：微秒)。在`iothread_period`内，域中的每个 IOThread 将不允许消耗超过`iothread_quota `的运行时间。该值应在 [1000， 1000000] 范围内。值为 0 的` iothread_period `表示没有值。自 2.1.0 起仅支持 QEMU 驱动程序

`iothread_quota` 

​	可选的`iothread_quota`元素指定 IOThread 允许的最大带宽(单位：微秒)。`iothread_quota`为任何负值的域 表示该域 IOThreads 具有无限带宽，这意味着它不受带宽控制。该值应在 [1000， 17592186044415] 范围内或小于 0。 值为 0 的`iothread_quota`表示没有值。 可以使用此功能来确保所有 IOThread 以相同的速度运行。自 2.1.0 起仅支持 QEMU 驱动程序

`vcpuched， iothreadsched` and `emulatorsched`

​	可选的`vcpusched、iothreadsched`和`emulatorsched`元素分别指定特定 vCPU、IOThread 和模拟器线程的调度程序类型(值`batch、idle、fifo、rr `)。对于`vcpusched` 和`iothreadsched `，属性`vcpus`和`iothreads`选择此设置适用于哪些 vCPU/IOThread，将它们保留为默认值。元素`emulatorsched`没有该属性。有效的vCPU 值从 0 开始，到比为域定义的 vCPU 数量减一为止。有效的iothreads值在[IOThreads Allocation](https://libvirt.org/formatdomain.html#iothreads-allocation)部分中描述 。如果未定义`iothreadids `，则 libvirt 会将 IOThread 编号从 1 到 可用于域的`iothread`数量。对于实时调度程序(`fifo、rr`)，还必须指定优先级(对于非实时调度程序则忽略优先级)。优先级的取值范围取决于主机内核(通常为1-99)。 自 1.2.13 起 `emulatorsched` 自 5.3.0 起

`cachetune` Since 4.1.0

​	可选的`cachetune`元素可以使用主机上的resctrl控制CPU缓存的分配。是否支持这一点可以从报告了一些限制(如最小size和所需粒度)的功能中收集。所需的属性`vcpus`指定此分配应用于哪些vCPU。vCPU只能是一个`cachetune`元素分配的成员。cachetune指定的vCPU可以与memorytune中的vCPU相同，但不允许重叠。可选的、仅输出的id属性唯一标识缓存。支持的子元素有：

​	`cache`

​		此可选元素控制CPU缓存的分配，并具有以下属性：

​		`level`

​			要分配的主机缓存级别。

​		`id`

​			要从中分配的主机缓存id。

​		`type`

​			分配类型。可以是代码(指令)的 `code`，数据的 `data`，或者代码和数据都包含的 `both`(统一)。目前，分配只能与主机支持的相同类型一起进行，这意味着 无法请求`both`在启用了 CDP(代码/数据优先级)的主机上同时分配两者。

​		`size`

​			要分配的区域的大小。默认情况下，值以字节为单位，但单位属性可用于缩放值。

​		`unit`(optional)

​			如果指定，则是指定大小的单位，如KiB、MiB、GiB或TiB(在内存分配的`memory`元素中描述)，`size`默认为字节。

​	`monitor` Since4.10.0

​		可选元素`monitor`监视器为当前缓存分配创建缓存监视器，并具有以下必需属性：

​		`level`

​			监视器所属的主机缓存级别。

​		`vcpus`

​			监视器应用的vCPU列表。监视器的vCPU名单只能是关联分配的vCPU清单的成员。默认监视器具有与关联分配相同的vCPU列表。对于非默认监视器，不允许重叠vCPU。

`memorytune` Since4.7.0

​	可选的`memorytune`元素可以使用主机上的resctrl控制内存带宽的分配。是否支持这一点可以从报告了一些限制(如最小带宽和所需粒度)的功能中收集。所需的属性`vcpus`指定此分配应用于哪些vCPU。vCPU只能是一个内存调整元素分配的成员。`memorytune`指定的vcpus可以与`cachetune`指定相同。但是，它们不允许相互重叠。支持的子元素有：

​	`node`

​		此元素控制CPU内存带宽的分配，并具有以下属性：

​		`id`

​			从中分配内存带宽的主机节点id。

​		`bandwidth`

​			要从此节点分配的内存带宽。默认情况下，该值以百分比为单位。



### 1.7 内存分配(Memory Allocation)

```xml
<domain>
  ...
  <maxMemory slots='16' unit='KiB'>1524288</maxMemory>
  <memory unit='KiB'>524288</memory>
  <currentMemory unit='KiB'>524288</currentMemory>
  ...
</domain>
```

`memory`

​	在引导时为虚拟机分配的最大内存。内存分配包括在启动时指定的可能的附加内存设备，或后来的热插拔。该值的单位由可选属性`unit` 决定，默认为 "KiB"(千字节，210 或 1024 字节的块)。有效的单位包括 "b" 或 "bytes" 代表字节，"KB" 代表千字节(10^3 或 1，000 字节)，"k" 或 "KiB" 代表 kibibytes(1024 字节)，"MB" 代表兆字节(10^6 或 1,000,000 字节)，"M" 或 "MiB" 代表 mebibytes(2^20 或 1,048,576 字节)，"GB" 代表千兆字节(10^9 或 1,000,000,000 字节)，"G" 或 "GiB" 代表 gibibytes(2^30 或 1,073,741,824 字节)，"TB" 代表太字节(10^12 或 1,000,000,000,000 字节)，或 "T" 或 "TiB" 代表 tebibytes(2^40 或 1,099,511,627,776 字节)。但是，libvirt 会将该值舍入到最接近的 kibibyte，并且可能会进一步舍入到hypervisor支持的粒度。一些hypervisor还会强制实施最小内存分配，如 4000KiB。如果为guest配置了 NUMA(请参阅  1.14 [CPU model and topology](https://libvirt.org/formatdomain.html#cpu-model-and-topology))，则可以省略`memory`内存元素。在崩溃的情况下，可以使用可选属性 `dumpCore` 来控制是否在生成的核心转储文件中包括guest内存(值为 "on" 或 "off")。`unit` 自 0.9.11 版本引入，`dumpCore` 自 0.10.2 版本引入(仅适用于 QEMU)。

`maxMemory`

​	guest运行时最大内存分配。<memory>元素或NUMA单元大小配置指定的初始内存可以通过将内存热插拔到该元素指定的限制来增加。`unit`属性的行为与<memory>相同。`slots`属性指定可用于向guest添加内存的插槽数。界限是特定于hypervisor的。请注意，由于通过内存热插拔添加的内存块的对齐，可能无法实现此元素指定的完整大小分配。自1.2.14由QEMU驱动程序支持。

`currentMemory`

​	guest的实际内存分配。该值可以小于最大分配，以便在运行中增加guest内存。如果忽略此项，则默认为与`memory`内存元素相同的值。`unit`属性的行为与`memory`相同。



### 1.8 内存备份(Memory Backing)

```xml
<domain>
  ...
  <memoryBacking>
    <hugepages>
      <page size="1" unit="G" nodeset="0-3，5"/>
      <page size="2" unit="M" nodeset="4"/>
    </hugepages>
    <nosharepages/>
    <locked/>
    <source type="file|anonymous|memfd"/>
    <access mode="shared|private"/>
    <allocation mode="immediate|ondemand" threads='8'/>
    <discard/>
  </memoryBacking>
  ...
</domain>
```

可选元素`memoryBacking` 可包含影响主机页如何支持虚拟存储器页的几个元素。

`hugepages`

​	这告诉虚拟机监控程序应该使用大页而不是正常的本机页面大小来分配guest内存。从 1.2.5 开始，可以为每个 numa 节点更具体地设置大页。引入`page`页面元素。它有一个强制属性 `size`，指定应使用哪些大页(在支持不同大小的大页的系统上特别有用)。`size`属性的默认单位是千字节(1024 的倍数)。如果 想使用不同的单位，请使用可选的`unit`属性。对于具有 NUMA 的系统，可选的`nodeset`节点集属性可能会很方便，因为它将给定guest的 NUMA 节点与某些大页面大小联系起来。从示例代码片段来看，除 4 号节点外，每个 NUMA 节点都使用 1 GB 的大页。有关正确的语法，请参阅 1.10 [NUMA Node Tuning](https://libvirt.org/formatdomain.html#numa-node-tuning)。

`nosharepages`

​	指示系统监控程序禁用此域的共享页面(内存合并，KSM)。自1.0.6起

`locked`

​	当系统监控程序设置并支持时，属于域的内存页将被锁定在主机的内存中，并且主机将不允许将它们交换出去，这对于某些工作负载(如实时工作负载)可能是必需的。对于QEMU/KVM的 guests，QEMU进程本身使用的内存也将被锁定：与guest内存不同，这是libvirt无法提前计算出的数量，因此它必须完全取消对锁定内存的限制。因此，启用此选项会带来潜在的安全风险：当内存耗尽时，主机将无法从客户机收回锁定的内存，这意味着恶意客户机分配大量锁定内存可能会对主机造成拒绝服务攻击。因此，除非 的工作负载需要，否则不鼓励使用此选项；即便如此，强烈建议在内存分配上同时设置一个适用于特定环境的`hard_limit`(请参阅Memory Tuning )，以减轻上述风险。自1.0.6起

`source`

​	使用`type`属性，可以提供"file"以利用文件内存备份或保持默认的"匿名"。从4.10.0开始， 可以选择"memfd"备份。(仅限QEMU/KVM)

`access`

​	使用`mode`属性，指定内存是"shared"还是"private"。`memAccess`可以覆盖每个numa节点。

`allocation`

​	使用可选`mode`模式属性，通过提供"immediate"立即或"ondemand"按需指定何时分配内存。自8.2.0起，可以通过`thread`属性设置系统监控程序用于分配内存的线程数。为了加快分配过程，当固定仿真线程时，建议包括来自所需NUMA节点的CPU，以便分配线程可以设置其关联性。

`discard`

​	当系统监控程序设置并支持时，内存内容将在guest关闭前(或拔下DIMM模块时)。请注意，这只是一种优化，并不能保证在所有情况下都能工作(例如，当系统监控程序崩溃时)。自4.4.0起(仅限QEMU/KVM)



### 1.9 内存调整/优(Memory Tuning)

```xml
<domain>
  ...
  <memtune>
    <hard_limit unit='G'>1</hard_limit>
    <soft_limit unit='M'>128</soft_limit>
    <swap_hard_limit unit='G'>2</swap_hard_limit>
    <min_guarantee unit='bytes'>67108864</min_guarantee>
  </memtune>
  ...
</domain>
```

`memtune`

​	可选元素`memtune`提供了有关域的内存可调参数的详细信息。如果忽略此项，则默认为操作系统提供的默认值。对于QEMU/KVM，参数作为一个整体应用于QEMU进程。因此，在计算它们时，需要将guest的RAM、guest视频RAM和QEMU本身的一些内存开销相加。最后一块很难确定，所以需要猜测和尝试。对于每个可调，可以使用与<memory>相同的值来指定数字在输入时的单位。为了向后兼容，输出始终使用KiB。`unit`自0.9.11所有 *_limit参数的可能值在0到VIR_DOMAIN_MEMORY_PARAM_UNLIMITED的范围内。

`hard_limit`

​	可选元素`hard_limit`是guest可以使用的最大内存。该值的单位为kibiytes(即1024字节的块)。强烈建议QEMU和KVM的用户不要设置此限制，因为如果猜测值太低，域可能会被内核杀死，并且确定进程运行所需的内存是一个无法确定的问题；也就是说，如果由于工作负载的要求， 已经在Memory Backing内存备份中设置了`locked`，那么 必须考虑部署的具体情况，并计算出`hard_limit`的值，该值足够大到可以支持guest的内存需求，但又要足够小到可以保护主机免受恶意guest锁定所有内存的影响。

`soft_limit`	

​	可选元素`soft_limit`是在内存争用期间强制执行的内存限制。该值的单位为kibiytes(即1024字节的块)

`swap_hard_limit`

​	可选元素`swap_hard_limit`是guest可以使用的最大内存加交换。该值的单位为kibiytes(即1024字节的块)。这必须大于所提供的hard_limit值

`min_guarantee`

​	可选元素`min_guarantee`是为客户机保证的最小内存分配。该值的单位为kibiytes(即1024字节的块)。此元素仅受VMware ESX和OpenVZ驱动程序的支持。



### 1.10 NUMA 节点调优(NUMA Node Tuning)

注1：NUMA(Non-Uniform Memory Access，非一致性内存访问)是一种计算机体系结构，用于处理多处理器系统中的内存访问性能问题。在 NUMA 架构中，系统中的处理器核心和内存被分成多个节点，每个节点具有自己的本地内存。不同节点之间的内存访问速度可能会不同，这就是所谓的"非一致性"。

在虚拟化环境中，虚拟机(VM)通常运行在物理主机上，而主机可能具有多个 NUMA 节点。NUMA 节点之间的内存访问延迟和带宽可能不同，这会影响虚拟机的性能。因此，NUMA 节点调优(NUMA Node Tuning)是一种优化措施，用于确保虚拟机在 NUMA 架构下获得更好的性能。

```xml
<domain>
  ...
  <numatune>
    <memory mode="strict" nodeset="1-4，^3"/>
    <memnode cellid="0" mode="strict" nodeset="1"/>
    <memnode cellid="2" mode="preferred" nodeset="2"/>
  </numatune>
  ...
</domain>
```

`numatune`

​	可选元素`numatune`提供了如何通过控制域进程的NUMA策略来调整NUMA主机性能的详细信息。注意，仅由QEMU驱动程序支持。自0.9.3起

`memory`

​	可选元素`memory`指定如何为NUMA主机上的域进程分配内存。它包含几个可选属性。属性`mode`模式为"interleave"、"strict"、"preferred"或"restrictive"，默认为"strict)。值"restrictive"指定使用系统默认策略，并且仅使用cgroups来限制内存节点，并且它要求在`memnode`元素中将模式设置为"restricted"(请参阅下面的quirk)。这仅仅是为了能够使用`virsh-numatune`或`virDomainSetNumaParameters`请求为正在运行的域移动此类内存，并且不能保证会发生这种情况。属性`nodeset`指定NUMA节点，使用与元素`vcpu`的属性`cpuset`相同的语法。属性`placement`(自0.9.12起)可用于指示域进程的内存放置模式，其值可以是"静态"或"自动"，默认为`vcpu`的`placement`，如果指定了`nodeset`节点集，则为"静态"。"auto"表示域进程将只从查询numad返回的咨询节点集中分配内存，如果指定了属性节点集的值，则会忽略该值。如果vcpu的位置为"auto"，并且未指定numatune，则将隐式添加位置为"autom"且模式为"strict"的默认numatune。自0.9.3起，请参阅[virDomainSetNumaParameters](https://libvirt.org/html/libvirt-libvirt-domain.html#virDomainSetNumaParameters)了解有关此元素更新的更多信息。

`memnod`

​	可选元素`memnode`可以为每个guest的NUMA节点指定内存分配策略。对于那些没有相应`memnode`元素的节点，将使用`memory`元素中的默认值。属性`cellid`指定了应用设置的客户端 NUMA 节点的地址。属性`mode`和`nodeset`具有与`memory`元素相同的含义和语法。此设置与自动放置不兼容。请注意，对于`memnode`，这将仅引导vCPU线程或类似机制的内存访问，并且在很大程度上取决于虚拟化监控程序的特定实现(hypervisor-specific)。这并不能保证节点内存分配的位置。为了进行适当的限制，应使用其他方式(例如不同的模式、预先分配的hugepages)。QEMU自1.2.7起



### 1.11 块I/O 调整/优(Block I/O Tuning)

```xml
<domain>
  ...
  <blkiotune>
    <weight>800</weight>
    <device>
      <path>/dev/sda</path>
      <weight>1000</weight>
    </device>
    <device>
      <path>/dev/sdb</path>
      <weight>500</weight>
      <read_bytes_sec>10000</read_bytes_sec>
      <write_bytes_sec>10000</write_bytes_sec>
      <read_iops_sec>20000</read_iops_sec>
      <write_iops_sec>20000</write_iops_sec>
    </device>
  </blkiotune>
  ...
</domain>
```

`blkotune`

​	可选元素`blkiotune`提供了为域调整Blkio-cgroup可调参数的能力。如果忽略此项，则默认为操作系统提供的默认值。自0.8.8

`weight`

​	可选元素`weight`是guest的总体I/O权重。该值应在[100， 1000]的范围内。在内核2.6.39之后，该值可能在[10， 1000]的范围内。

`device`

​	域可以具有多个`device`元素，这些设备元素进一步调整域使用的每个主机块设备的权重。请注意，如果多个磁盘(请参阅 [Hard drives, floppy disks, CDROMs](https://libvirt.org/formatdomain.html#hard-drives-floppy-disks-cdroms)—1.20.1节)由同一主机文件系统中的文件备份，则它们可以共享一个主机块设备，这就是为什么这个调整参数是全局域级别的，而不是与每个客户磁盘设备相关联(与磁盘定义的<iotune>元素形成对比(请参阅 [Hard drives, floppy disks, CDROMs](https://libvirt.org/formatdomain.html#hard-drives-floppy-disks-cdroms)—1.20.1节)，后者可以应用于单个磁盘)。每个`device`元素都有两个强制的子元素，一个是描述设备绝对路径的`path`，另一个是给出该设备相对权重的`weight`，范围在[100， 1000]。在内核2.6.39之后，该值可能在[10， 1000]的范围内。自0.9.8，此外，可以使用以下可选子元素：

​	`read_bytes_sec`

​		读吞吐量限制(以字节每秒为单位)。自1.2.2

​	`write_bytes_sec`

​		写吞吐量限制(以字节每秒为单位)。自1.2.2

​	`read_iops_sec`

​		每秒读取I/O操作数限制。自1.2.2

​	`write_iops_sec`

​		每秒写入I/O操作数限制。自1.2.2



### 1.12 资源分区(Resource partitioning)

Hypervisor可以允许将虚拟机放置在资源分区中，可能会嵌套这些分区。`resource`元素将与资源分区相关的配置组合在一起。目前支持一个名为`partition`的子元素，其内容定义了要将该域放置在哪个资源分区的绝对路径中。如果没有列出分区，那么该域将被放置在默认分区中。在启动客户机之前，应用程序/管理员有责任确保分区存在。默认情况下，只能假定(特定hypervisor)默认分区存在。

```xml
...
<resource>
  <partition>/virtualmachines/production</partition>
</resource>
...
```

资源分区目前由QEMU和LXC驱动程序支持，它们将分区路径映射到所有安装的控制器中的cgroups目录。自1.0.5



### 1.13 光纤通道VMID(Fibre Channel VMID)

FC SAN(Fibre Channel Storage Area Network)可以根据虚拟机标识(VMID)提供各种不同的服务质量(QoS)级别和访问控制。它还可以在每个虚拟机的级别收集遥测数据，这些数据可以用来提升虚拟机的IO性能。这可以通过使用光纤通道元素的`appid`属性进行配置。该属性包含一个字符串(最多128字节)，内核会使用它来创建VMID(Virtual Machine Identifier).

```xml
...
<resource>
  <fibrechannel appid='userProvidedID'/>
</resource>
...
```

使用此功能需要支持光纤通道的硬件、使用`CONFIG_BLK_CGROUP_FC_APPID`编译的内核和加载`nvme_FC`的内核模块。自7.7.0



### 1.14 (CPU model and topology)

可以使用以下元素集合指定CPU模型、其功能和拓扑结构的要求。自0.7.5

```xml
...
<cpu match='exact'>
  <model fallback='allow'>core2duo</model>
  <vendor>Intel</vendor>
  <topology sockets='1' dies='1' cores='2' threads='1'/>
  <cache level='3' mode='emulate'/>
  <maxphysaddr mode='emulate' bits='42'/>
  <feature policy='disable' name='lahf_lm'/>
</cpu>
...
```

```xml
<cpu mode='host-model'>
  <model fallback='forbid'/>
  <topology sockets='1' dies='1' cores='2' threads='1'/>
</cpu>
...
```

```xml
<cpu mode='host-passthrough' migratable='off'>
  <cache mode='passthrough'/>
  <maxphysaddr mode='passthrough' limit='39'/>
  <feature policy='disable' name='lahf_lm'/>
...
```

```xml
<cpu mode='maximum' migratable='off'>
  <cache mode='passthrough'/>
  <feature policy='disable' name='lahf_lm'/>
...
```

如果不需要对CPU模型及其功能进行限制，可以使用更简单的CPU元素。自0.7.6

```xml
...
<cpu>
  <topology sockets='1' dies='1' cores='2' threads='1'/>
</cpu>
...
```

`cpu`

​	`cpu`元素是用于描述guest cpu需求的主要容器。它的`match`属性指定提供给guest的虚拟CPU与这些要求的匹配程度。自0.7.6，如果`topology`是`cpu`中唯一的元素，则可以省略`match`属性。`match`属性的可能值为：

​	`minimum`

​		指定的CPU型号和功能描述了请求的最小CPU。如果可以在当前主机上使用请求的虚拟机监控程序，将为客户机提供更好的CPU。这是一种受约束的`host-model `模型模式；如果提供的虚拟CPU不满足要求，则不会创建域。

​	`exact`

​		提供给guest的虚拟CPU应该与规范完全匹配。如果不支持这样的CPU，libvirt将拒绝启动域。

​	`strict`

​		除非主机CPU与规范完全匹配，否则不会创建域。这在实践中不是很有用，只有在有真正原因的情况下才应该使用。

0.8.5起，`match`属性可以省略，默认为`exact`。有时系统监控程序无法创建与libvirt传递的规范完全匹配的虚拟CPU。从3.2.0开始，可选的`check`属性可用于请求检查虚拟CPU是否符合规范的特定方式。在启动域时，通常可以安全地省略此属性，并使用默认值。一旦域启动，libvirt将自动将`check`属性更改为支持的最佳值，以确保域迁移到另一台主机时虚拟CPU不会发生更改。可以使用以下值：

​	`none`

​		Libvirt不进行检查，如果无法提供请求的CPU，则由hypervisor拒绝启动域。对于QEMU，这意味着根本不进行检查，因为QEMU的默认行为是发出警告，但无论如何都要启动域。

​	`partial`

​		Libvirt将在启动域之前检查guest的CPU规范，但其余内容将保留在hypervisor上。它仍然可以提供不同的虚拟CPU。

​	`full`

​		hypervisor创建的虚拟CPU将根据CPU规范进行检查，除非两个CPU匹配，否则不会启动域。

0.9.10起，可以使用可选的`mode`属性，以便更容易地将客户CPU配置为尽可能接近主机CPU。模式属性的可能值为：

​	`custom`

​		在这种模式下，`cpu`元素描述了应该提供给guest的cpu。当未指定`mode`属性时，这是默认设置。这种模式使得持久guest无论在哪个主机上启动都能看到相同的硬件。

​	`host-model`

​		`host-model`模式本质上是将主机CPU定义从capabilities XML复制到域XML的快捷方式。由于CPU定义是在启动域之前复制的，因此完全相同的XML可以在不同的主机上使用，同时仍然提供每个主机支持的最佳客户CPU。`match`属性不能在此模式中使用。也不支持指定CPU型号，但仍可使用`model`的`fallback`属性。使用`feature`元素，除了host 模型之外，还可以专门启用或禁用特定标志。这可以用于调整可以模拟的功能。(从1.1.1开始).Libvirt并没有对每个CPU的各个方面进行建模，因此客户机CPU不会与主机CPU完全匹配。另一方面，提供给guest的ABI是可复制的。在迁移过程中，完整的CPU模型定义会转移到目标主机，因此即使目标主机包含更多功能的CPU或更新的内核，迁移的客户机也会看到与客户机运行实例完全相同的CPU模型；但是关闭和重新启动guest可以根据新主机的能力向guest呈现不同的硬件。在libvirt 3.2.0和QEMU 2.9.0之前，不支持通过QEMU检测主机CPU模型。因此，使用主机模型创建的CPU配置可能无法按预期工作。自3.2.0和QEMU 2.9.0，该模式按照设计方式工作，并且在 [domain capabilities XML](https://libvirt.org/formatdomaincaps.html#cpu-configuration)中公布的主机模型CPU定义中，`fallback`属性设置为`forbid`。当`fallback`属性设置为`allow`域功能XML时，建议仅使用host capabilities XML中的CPU模型使用`custom`模式。自1.2.11 PowerISA允许处理器在二进制兼容模式下运行VM，支持旧版本的ISA。PowerPC体系结构上的Libvirt使用`host-model`来表示在二进制兼容模式下运行的guest模式CPU。示例：当用户需要power7虚拟机在Power8主机上以兼容模式运行时，可以用XML描述如下：

```xml
<cpu mode='host-model'>
  <model>power7</model>
</cpu>
...
```

注1：ABI(Application Binary Interface)定义了不同部分(通常是软件和硬件之间)之间的二进制接口标准，以确保不同程序和模块能够互相协同工作。

注2：PowerISA(Power Instruction Set Architecture)(Power指令集架构)微处理器架构

​	`host-passthrough` 物理机透传

​		在这种模式下，提供给虚拟机的CPU应该与主机CPU完全相同，即使在libvirt无法理解的方面也是如此。然而，这种模式的缺点是虚拟机环境无法在不同的硬件上重现。因此，如果遇到任何问题，你需要自己解决。可以使用`feature`元素来进一步更改虚拟CPU的详细信息。如果源主机和目标主机在硬件、QEMU版本、 microcode版本和配置方面不完全相同，那么使用host-passthrough迁移虚拟机是危险的。如果尝试这样的迁移，那么虚拟机可能会在在目标主机上恢复执行时挂起或崩溃。根据hypervisor版本，虚拟CPU可能会包含阻止迁移的功能，即使迁移到一个完全相同的主机也可能如此。自6.5.0起，可选的`migratable`属性可用于显式请求将这些功能从虚拟CPU中删除(on)或保留在其中(off)。这个属性并不会使迁移到另一台主机更安全：即使使用`migratable='on'`，除非两台主机如上所述完全相同，否则迁移仍然是危险的。

​	`maximum`

​		当使用硬件虚拟化运行客户机时，此CPU型号在功能上与主机直通相同，因此请参阅上面的文档。

​		当使用CPU仿真运行客户机时，此CPU模型将启用仿真引擎能够支持的最大功能集。请注意，即使migrateable='on'，迁移也会很危险，除非两个主机都运行相同版本的模拟代码。

​		自7.1.0起使用QEMU驱动程序。

当域可以直接在主机CPU上运行时，`host-mode`和`host-passthrough`模式都是有意义的(例如，类型为`kvm`或`hvf`的域)。实际主机CPU与具有模拟虚拟CPU的域(例如`qemu`类型的域)无关。然而，为了向后兼容，即使对于在模拟CPU上运行的域，也可以实现`host-model`，在这种情况下，可以使用hypervisor能够模拟的最佳CPU，而不是试图模拟主机CPU模型。

如果应用程序不关心特定的CPU，只想要最好的功能集而不需要迁移兼容性，那么在可用的hypervisor上，`maximum`模型是一个不错的选择。

`model`

​	`model`元素的内容指定guest请求的CPU模型。可用CPU型号及其定义的列表可以在目录`cpu_map`中找到，该目录安装在libvirt的数据目录中。如果hypervisor无法使用确切的CPU模型，那么libvirt会自动返回到hypervisor支持的最接近的模型，同时维护CPU功能列表。自0.9.10以来，可以使用可选的`fallback`属性来禁止这种行为，在这种情况下，启动请求不支持的CPU模型的域的尝试将失败。`fallback`属性支持的值为：`allow`(这是默认值)和`forbid`。可选的`vendor_id`属性(0.10.0起)可用于设置guest看到的供应商id。它必须正好有12个字符长。如果未设置，则使用主机的供应商id。典型的可能值是"AuthenticAMD"和"GenuineIntel"。

`vendor`-cpu供应商-amd / intel / hygon

​	0.8.3起，`vendor`元素的内容指定了guest请求的CPU vendor。如果缺少此元素，则无论供应商如何，guest都可以在与给定功能匹配的CPU上运行。支持的供应商列表可以在`cpu_map/*_vendors.xml`中找到。

`topology`拓扑

​	`topology`元素指定提供给客户的虚拟CPU的请求拓扑。`sockets`、`dies`、`cores`和`threads`这四个属性接受非零正整数值。它们分别指CPU sockets(插槽)的总数、每个sockets的die(裸片数，晶圆)、每个die的内核数和每个内核的线程数。`dies`属性是可选的，如果省略，则默认为1，而其他属性都是必需的。Hypervisor可能要求`cpus`元素指定的最大vCPU数量等于拓扑结构产生的vCPUs数量。

注1：海光Hygon CPU 架构图

![cpu架构补充理解dies](D:\File Storage\TyporaStorage\image-20230906112940539.png)

`feature`

​	`cpu`元素可以包含零个或多个用于调整由所选cpu模型提供的特征的`feature`元素。已知功能名称的列表可以在与CPU型号相同的文件中找到。每个`feature`元素的含义取决于其`policy`属性，该属性必须设置为以下值之一：

​	`force`

​		无论主机CPU是否支持该功能，虚拟CPU都会声明该功能得到支持。

​	`require`

​		除非主机CPU支持该功能或hypervisor能够模拟该功能，否则guest创建将失败。

​	`optional`

​		仅当主机CPU支持该功能时，虚拟CPU才会支持该功能。

​	`disable`

​		虚拟CPU不支持该功能。

​	`forbid`

​		如果主机CPU支持该功能，则创建客户端将失败。

从0.8.5开始，`policy`属性可以省略，默认为`require`。

单个CPU功能名称作为`name`属性的一部分指定。例如，要显式指定英特尔IvyBridge CPU模型的"pcid"功能:

```xml
...
<cpu match='exact'>
  <model fallback='forbid'>IvyBridge</model>
  <vendor>Intel</vendor>
  <feature policy='require' name='pcid'/>
</cpu>
...
```

`cache`

​	自3.3.0开始，`cache`元素描述了虚拟CPU缓存。如果该元素不存在，hypervisor将使用一个合理的默认值。

​	`level`

​		此可选属性指定元素描述的缓存级别。缺少属性意味着元素同时描述所有CPU缓存级别。禁止混合使用带`level`属性集的`cache`元素和不带`level`属性集的缓存元素。

​	`mode`

​		支持以下取值:

​		`emulate`

​			hypervisor将提供一个假的CPU缓存数据。

​		`passthrough` 透传

​			主机CPU上的真实CPU cache数据会被传递给虚拟CPU。

​		`disable`

​			虚拟CPU将报告没有指定级别的CPU缓存(或者如果缺少级别属性则根本没有缓存)。

`maxphysaddr`

​	8.7.0起，`maxphysaddr`元素以位为单位描述虚拟CPU地址的大小。如果缺少该元素，则使用hypervisor默认值。

​	`mode`

​		这个强制属性指定地址大小的显示方式。支持以下模式:

​		`passthrough`

​			主机CPU报告的物理地址位数将传递给虚拟CPU

​		`emulate`

​			hypervisor将通过`bits`属性为物理地址的位数定义一个特定的值(自9.2.0是可选的)，位数不能超过hyperviosr支持的物理地址位数。

​	`bits`

​		如果`mode`属性设置为`emulation`，并且指定虚拟CPU地址大小(以位为单位)，则`bits`属性是必选的。

​	`limit`

​		`limit`属性可用于限制`passthrough`模式的地址位的最大值，即，如果主机CPU报告的位超过该值，则使用`limit`。自9.3.0

可以使用`numa`元素指定guest的NUMA拓扑。

```xml
...
<cpu>
  ...
  <numa>
    <cell id='0' cpus='0-3' memory='512000' unit='KiB' discard='yes'/>
    <cell id='1' cpus='4-7' memory='512000' unit='KiB' memAccess='shared'/>
  </numa>
  ...
</cpu>
...
```

每个`cell`元素指定一个NUMA cell或NUMA节点。`cpus`指定节点的CPU或CPUs范围。从6.5.0开始，对于qemu驱动程序，如果模拟器二进制支持每个`cell`使用不相交`cpus`范围，则每个`cell`中声明的所有cpu的总和将与`vcpu`元素中声明的虚拟cpu的最大数量相匹配。这是通过将任何剩余的CPU填充到第一个NUMA `cell`来完成的。鼓励用户提供完整的NUMA拓扑，其中NUMA CPUs的总数与在`vcpus`中声明的最大虚拟CPU数量相匹配，以确保在qemu和libvirt的各个版本中，域的配置保持一致。`memory`指定了以kibibytes(即1024字节的块)为单位的节点内存。自6.6.0以来，`cpus`属性是可选的，如果省略，则会创建一个没有CPU的NUMA节点。自1.2.11以来，可以使用额外的`unit`属性(请参阅内存分配 1.7[Memory Allocation](https://libvirt.org/formatdomain.html#memory-allocation))，以定义`memory`的单位。自1.2.7起，所有的单元都应该具有`id`属性，以便在代码中引用某些单元，否则这些单元将按从0开始的递增顺序分配id。不建议混合具有`id`属性和没有`id`属性的单元，因为这可能导致意外的行为。自1.2.9起，`memAccess`是一个可选属性，可以控制内存是否映射为"shared"或"private"。这仅适用于hugepages支持的内存和nvdimm模块。每个`cell`元素都可以具有一个可选的`discard`属性，用于调整给定NUMA节点的丢弃功能，如在" [Memory Backing](https://libvirt.org/formatdomain.html#memory-backing)."下所述。可接受的值为`yes`和`no`。自4.4.0起

这个guest的NUMA规范目前仅适用于QEMU/KVM和Xen。

NUMA硬件架构支持NUMA单元之间距离的概念。从3.10.0开始，可以使用NUMA `cell`描述中的`distances`元素来定义NUMA单元之间的距离。`sibling`子元素用于指定兄弟NUMA单元格之间的距离值。要了解更多细节，请参阅ACPI(Advanced Configuration and Power Interface，高级配置和电源接口)规范中解释系统的SLIT(System Locality Information Table，系统位置信息表)的章节。

```xml
...
<cpu>
  ...
  <numa>
    <cell id='0' cpus='0，4-7' memory='512000' unit='KiB'>
      <distances>
        <sibling id='0' value='10'/>
        <sibling id='1' value='21'/>
        <sibling id='2' value='31'/>
        <sibling id='3' value='41'/>
      </distances>
    </cell>
    <cell id='1' cpus='1，8-10，12-15' memory='512000' unit='KiB' memAccess='shared'>
      <distances>
        <sibling id='0' value='21'/>
        <sibling id='1' value='10'/>
        <sibling id='2' value='21'/>
        <sibling id='3' value='31'/>
      </distances>
    </cell>
    <cell id='2' cpus='2，11' memory='512000' unit='KiB' memAccess='shared'>
      <distances>
        <sibling id='0' value='31'/>
        <sibling id='1' value='21'/>
        <sibling id='2' value='10'/>
        <sibling id='3' value='21'/>
      </distances>
    </cell>
    <cell id='3' cpus='3' memory='512000' unit='KiB'>
      <distances>
        <sibling id='0' value='41'/>
        <sibling id='1' value='31'/>
        <sibling id='2' value='21'/>
        <sibling id='3' value='10'/>
      </distances>
    </cell>
  </numa>
  ...
</cpu>
...
```

描述NUMA单元之间的距离目前仅由Xen和QEMU支持。如果没有给出`distances`来描述不同小区之间的SLIT数据，则默认使用10表示本地距离和20表示远程距离的方案。

注1：SLIT(System Locality Information Table)是一种在NUMA(Non-Uniform Memory Access)体系结构中使用的数据结构，它提供了关于系统中不同本地性域之间的本地性信息。SLIT 数据通常以表格或矩阵的形式表示，其中每个条目表示一个本地性域对的距离。较小的值表示更接近的距离，而较大的值表示更远的距离。这种信息可以帮助操作系统更有效地将任务和数据分配到本地性域，以减少延迟和提高系统性能。

#### 	ACPIHeterogeneous Memory Attribute Table

```xml
...
<cpu>
  ...
  <numa>
    <cell id='0' cpus='0-3' memory='2097152' unit='KiB' discard='yes'>
      <cache level='1' associativity='direct' policy='writeback'>
        <size value='10' unit='KiB'/>
        <line value='8' unit='B'/>
      </cache>
    </cell>
    <cell id='1' cpus='4-7' memory='512000' unit='KiB' memAccess='shared'/>
    <interconnects>
      <latency initiator='0' target='0' type='access' value='5'/>
      <latency initiator='0' target='0' cache='1' type='access' value='10'/>
      <bandwidth initiator='0' target='0' type='access' value='204800' unit='KiB'/>
    </interconnects>
  </numa>
  ...
</cpu>
...
```

从6.6.0开始，`cell`元素可以有一个`cache`子元素，它描述内存邻近域的内存侧缓存。`cache`元素具有描述缓存级别的`level`属性，因此该元素可以被重复多次以描述缓存的不同级别。

然后，`cache`元素具有以下强制属性：

​	`level`

​		描述所涉及的缓存级别。

​	`assocaitivity`

​		描述缓存关联性(可接受的值为`none`、`direct`和`full`)。

​	`policy`

​		描述缓存写关联性(可接受的值为`none`、`writeback`和`writethrough`)。

`cache`元素有两个强制性的子元素:`size`和`line`，它们描述缓存大小和缓存行大小。两个元素都接受两个属性:`value`和`unit`，它们设置相应缓存属性的值.

NUMA描述有一个可选的`interconnects`元素，用于描述启动邻近域(处理器或I/O)和目标邻近域(内存)之间的规范化内存读/写延迟，读/写带宽。

`interconnects`元素可以具有零个或多个`latency`子元素来描述两个内存节点之间的延迟，以及零个或多个`bandwith`子元素来描述两个内存节点之间的带宽。它们都有以下强制属性:

​	`initiator`

​		指的是源NUMA节点

​	`target`

​		指的是目的NUMA节点

​	`type`

​		访问的类型。取值范围:`access`、`read`、`write`

​	`value`

​		实际值。对于延迟，该值以纳秒为单位，对于带宽，该值以千字节每秒为单位。使用附加的`unit`属性来改变单位。

为了描述从一个NUMA节点到另一个NUMA节点的缓存的延迟，`latency`元素具有可选的`cache`属性，该属性与`target`属性相结合，创建对远程NUMA节点缓存级别的完整引用。例如，`target='0' cache='1'`表示NUMA节点0的第一级缓存。



### 1.15 事件配置(Envents configuration)

有时需要覆盖对各种事件采取的默认操作。并非所有hypervisor都支持所有事件和操作。这些操作可能是调用libvirt api virDomainReboot、virDomainShutdown或virDomainShutdownFlags的结果。使用`virsh reboot`或`virsh shutdown`也会触发该事件。

```xml
...
<on_poweroff>destroy</on_poweroff>
<on_reboot>restart</on_reboot>
<on_crash>restart</on_crash>
<on_lockfailure>poweroff</on_lockfailure>
...
```

以下元素集合允许在guest操作系统触发生命周期操作时指定操作。一个常见的用例是在进行初始操作系统安装时强制将重新启动视为关机。这允许在安装后的第一次启动时重新配置VM。

`on_poweroff`
	此元素的内容指定guest请求关机时要采取的操作。
`on_reboot`
	此元素的内容指定guest请求重新启动时要采取的操作。
`on_crash`
	此元素的内容指定guest崩溃时要采取的操作。

这些状态中的每一个都允许相同的四种可能的操作。
`destroy`
	该域名将被完全终止，所有资源将被释放。
`restart`
	该域将被终止，然后使用相同的配置重新启动。
`preserve`
	域将被终止，其资源将被保留以允许分析。
`rename-restart`
	该域将被终止，然后用一个新名称重新启动。(仅支持libxl hypervisor驱动程序。)

QEMU/KVM/HVF支持`on_poweroff`和`on_reboot`事件处理`destroy`和`restart`动作，但是设置为 `restart`的`on_poweroff` 和设置为`destroy`的`on_reboot` 的组合是禁止的。

从0.8.4开始，`on_crash`事件支持这些附加操作。

​	`coredump-destroy`

​		崩溃的域的核心将被转储，然后该域将完全终止并释放所有资源

​	`coredump-restart`

​		崩溃的域的核心将被转储，然后该域将使用相同的配置重新启动

从3.9.0起，可以通过virDomainSetLifecycleAction API配置生命周期事件。

`on_lockfailure`元素(自1.0.0起)可用于配置当锁管理器丢失资源锁时应采取的操作。libvirt可以识别以下操作，尽管并非所有操作都需要单独的锁管理器支持。如果未指定任何操作，则每个锁管理器都将执行其默认操作。

​	`poweroff`

​		域将被强制关闭电源。

​	`restart`

​		域将关闭电源并重新启动以重新获取其锁定

​	`pause`

​		域将暂停，以便在锁问题解决后可以手动恢复。

​	`ignore`

​		保持域像什么都没发生一样运行。



### 1.16 (Power Management)

0.10.2起，可以强制启用或禁用到guestOS的BIOS广告。(注意：仅支持qemu驱动程序)

```xml
...
<pm>
  <suspend-to-disk enabled='no'/>
  <suspend-to-mem enabled='yes'/>
</pm>
...
```

`pm`

​		这些元素启用("是")或禁用("否")BIOS对S3(暂停到内存)和S4(暂停到磁盘)ACPI睡眠状态的支持。如果没有指定任何内容，那么hypervisor将保留其默认值。注意：此设置不能阻止guest操作系统执行挂起，因为guest操作系统本身可以选择绕过睡眠状态的不可用性(例如，通过完全关闭S4)。



### 1.17 (Hypervisor features)

Hypervisor可以允许打开/关闭某些CPU/机器功能。

```xml
...
<features>
  <pae/>
  <acpi/>
  <apic/>
  <hap/>
  <privnet/>
  <hyperv mode='custom'>
    <relaxed state='on'/>
    <vapic state='on'/>
    <spinlocks state='on' retries='4096'/>
    <vpindex state='on'/>
    <runtime state='on'/>
    <synic state='on'/>
    <stimer state='on'>
      <direct state='on'/>
    </stimer>
    <reset state='on'/>
    <vendor_id state='on' value='KVM Hv'/>
    <frequencies state='on'/>
    <reenlightenment state='on'/>
    <tlbflush state='on'/>
    <ipi state='on'/>
    <evmcs state='on'/>
  </hyperv>
  <kvm>
    <hidden state='on'/>
    <hint-dedicated state='on'/>
    <poll-control state='on'/>
    <pv-ipi state='off'/>
    <dirty-ring state='on' size='4096'/>
  </kvm>
  <xen>
    <e820_host state='on'/>
    <passthrough state='on' mode='share_pt'/>
  </xen>
  <pvspinlock state='on'/>
  <gic version='2'/>
  <ioapic driver='qemu'/>
  <hpt resizing='required'>
    <maxpagesize unit='MiB'>16</maxpagesize>
  </hpt>
  <vmcoreinfo state='on'/>
  <smm state='on'>
    <tseg unit='MiB'>48</tseg>
  </smm>
  <htm state='on'/>
  <ccf-assist state='on'/>
  <msrs unknown='ignore'/>
  <cfpc value='workaround'/>
  <sbbc value='workaround'/>
  <ibs value='fixed-na'/>
  <tcg>
    <tb-cache unit='MiB'>128</tb-cache>
  </tcg>
  <async-teardown enabled='yes'/>
</features>
...
```

所有功能都列在`features`元素中，省略一个可切换的功能标签会将其关闭。通过询问功能XML和域功能XML可以找到可用的功能，但完全虚拟化域的常见集合是：

​	`pae`

​		物理地址扩展模式允许32位guests寻址超过4GB的内存。

​	`acpi`

​		ACPI对于电源管理非常有用，例如，对于KVM或HVF客户，它是正常关机工作所必需的。

​	`apic`

​		APIC允许使用可编程的IRQ管理。0.10.2(仅限QEMU)，有一个可选属性`eoi`，其值为`on`和`off`，用于切换guest的eoi(中断结束)的可用性。

​	`hap`

​		根据`state`属性(值`on`、`off`)启用或禁用硬件辅助分页。如果hyperviosr检测到硬件辅助分页可用，默认设置为on。

​	`viridian`

​		启用Viridian虚拟化扩展，用于对半虚拟化guest OS进行虚拟化。

​	`privnet`

​		始终创建专用网络命名空间。如果定义了任何接口设备，则会自动设置。此功能仅与基于容器的虚拟化驱动程序(如LXC)相关。

​	`hyperv`

​		启用各种功能，改善运行Microsoft Windows的guest的行为。

​	![image-20230906195702475](D:\File Storage\TyporaStorage\image-20230906195702475.png)

8.0.0起，可以通过将`mode`属性设置为以下值之一来进一步配置hyperviosr：

​	`custom`

​		精确设置指定的功能。

​	`passthrough`

​		启用hypervisor当前支持的所有功能，即使是libvirt无法理解的功能。使用passthrough迁移虚拟机在源主机和目标主机的硬件、QEMU版本、微码版本和配置不完全相同的情况下是危险的。如果尝试这样的迁移，那么在目标主机上恢复执行时，虚拟机可能会挂起或崩溃。根据hypervisor的版本，虚拟CPU可能会包含可能阻止迁移到即使是相同主机的功能。

​	`mode`属性可以省略，并且将默认为`custom`自定义。

`pvspinlock`

​	通知guest主机支持准虚拟自旋锁，例如通过公开pvticketlocks机制。可以使用`state='off'`属性显式禁用此功能。

`kvm`

​	用于更改KVM虚拟机监控程序行为的各种功能。

![image-20230906205006218](D:\File Storage\TyporaStorage\image-20230906205006218.png)

`xen`

​	用于于更改Xen虚拟机监控程序行为的各种功能。

![image-20230906205050197](D:\File Storage\TyporaStorage\image-20230906205050197.png)

`pmu`

​	根据`state`属性(值打开、关闭、默认打开)启用或禁用客户机的性能监视单元。自1.2.12

`vmport`

​	根据`state`属性(值`on`、`off`、默认为`on`)，为vmmouse等启用或禁用VMware IO端口的模拟。自1.2.16

`gic`

​	使用常规中断控制器而不是APIC启用架构，以处理中断。例如，"aarch64"体系结构使用`gic`而不是`apic`。可选属性`version`指定GIC版本；但是，可能并非所有hypervisor都支持它。可接受的值为2、3和`host`。自1.2.16起

`smm`

​	根据`state`属性(值`on，off`，默认值`on`)启用或禁用系统管理模式。自2.1.0

​	可选的子元素`tseg`可用于指定专用于SMM扩展TSEG的内存量。除现有的选项(1个MIB，2 MIB和8 MIB)外，它提供了第四个选件大小，可供guest OS选择(或更确切地说是装载机)。可以将大小指定为该元素的值，可选属性`unit`可用于指定上述值的单元(默认为" MiB")。如果设置为0，则不会公布扩展大小，并且只有默认大小(见上文)可用。

​	**如果VM正在启动，则应该不理会此选项，除非 非常确定自己知道自己在做什么。**

​	该值是可配置的，因为无法在保证其正确工作的情况下正确进行计算。在QEMU中，用户可配置的扩展TSEG功能在`pc-q35-2.9`之前(包括该版本)不可用。从`pc-q35-2.10`开始，该功能可用，默认大小为16 MiB。这应该足以满足大约272个vCPU、总共5 GiB客户机RAM、无热插拔内存范围和32 GiB 64位PCI MMIO孔径的需求。或者适用于48个vCPU，具有1TB客户机RAM、无热插拔DIMM范围和32GB 64位PCI MMIO孔径。这些值还可以根据VM正在使用的加载器而变化。

​	可能需要额外的大小来显著提高vCPU计数或增加地址空间(可以是内存、maxMemory、64位PCI MMIO孔径大小；每1 TiB地址空间大约8 MiB TSEG)，也可以四舍五入。

​	由于此设置的性质类似于"客户应该有多少RAM"，建议用户查阅客户机操作系统或加载程序的文档(如果有的话)，或者通过反复更改值来进行测试，直到虚拟机成功引导。对于用户来说，另一个指导价值可能是48 MiB对于相当大的客户机(240 vCPU和4TB客户机RAM)应该足够了，但这是故意不设置为默认值，因为48 MiB的不可用RAM对于小客户机来说可能太多了(例如512 MiB的RAM)。

​	有关单位属性的更多详细信息，请参阅内存分配1.7 [Memory Allocation](https://libvirt.org/formatdomain.html#memory-allocation)。自4.5.0起(仅限QEMU)

`ioapic`

​	调优I/O APIC。驱动程序属性的可能值是:`kvm `(kvm域的默认值)和`qemu`，它将I/O APIC放在用户空间中，也称为拆分I/O APIC模式。3.4.0以后(仅QEMU/KVM)

`hpt`

​	配置pSeries客户机的HPT(Hash Page Table哈希页表)。启用了`resizing`属性的可能值，如果客户机和主机都支持HPT调整大小，则启用HPT调整大小;`disabled`，这将导致禁用HPT调整大小，而不管客户和主机支持;`required`，这将阻止客户机启动，除非客户机和主机都支持HPT调整大小。如果未定义该属性，则将使用hypervisor默认值。自3.10.0(仅QEMU/KVM)。

​	可选的`maxpagesize`子元素可用于限制HPT客户机可用的页面大小。常用值为64KiB、16MiB和16GiB;如果没有指定，将使用hyperviosr默认值。自4.5.0(仅QEMU/KVM)。

`vmcoreinfo`

​	启用QEMU vmcoreinfo设备，让客户内核保存调试细节。从4.4.0开始(仅限QEMU)

`htm`

​	为pSeries客户机配置HTM(硬件转换内存)可用性。`state`属性的可能值是`on`和`off`。如果未定义该属性，则将使用hypervisor默认值。从4.6.0开始(仅QEMU/KVM)

`nested-hv`

​	为pSeries客户机配置嵌套HV可用性。这需要从主机(L0)启用才能有效;如果计划在(L1)客户机中运行嵌套的(L2)客户机，那么在(L1)客户机中提供HV支持是非常可取的，因为这将使那些嵌套的客户机具有比使用KVM PR或TCG时更好的性能。`state`属性的可能值是`on`和`off`。如果未定义该属性，则将使用hypervisor默认值。4.10.0起(仅QEMU/KVM)

`msrs`

​	有些客户机可能需要忽略未知的模型特定寄存器(Model Specific Registers， MSR)读写。可以通过将`msrs`的未知属性设置为`ignore`来切换这种情况。如果未定义该属性或将其设置为`fault`，则不会忽略未知的读写。自5.1.0以来(仅限bhyve)

`ccf-assist`

​	为pSeries客户机配置ccf-assist (Count Cache Flush Assist)可用性。`state`属性的可能值是`on`和`off`。如果未定义该属性，则将使用hypervisor默认值。自5.9.0(仅QEMU/KVM)

`cfpc`

​	为pSeries客户配置cfpc(特权更改时缓存刷新)可用性。value属性的可能值是坏的(没有保护)、工作区(可用的软件工作区)和固定的(在硬件中固定的)。如果未定义该属性，则将使用hypervisor默认值。自6.3.0(仅QEMU/KVM)以来，

`sbbc`

​	为pSeries客户配置了sbbc(Speculation Barrier Bounds Checking， 推测栅栏边界检查)可用性。`value`属性的可能值是`broken`(没有保护)、`workaround`(可用的软件工作区)和`fixed`(在硬件中固定的)。如果未定义该属性，则将使用hypervisor默认值。从6.3.0开始(仅QEMU/KVM) 

`ibs`

​	为pSeries客户配置ibs(Indirect Branch Speculation， 间接分支推测)可用性。`value`属性的可能值`broken`(没有保护)、`workaround`(计数缓存刷新)、`fixed-ibs`(通过序列化间接分支修复)、`fixed-ccd`(通过禁用缓存计数修复)和`fixed-na`(在硬件上修复-不再适用)。如果未定义该属性，则将使用hypervisor默认值。从6.3.0(仅QEMU/KVM)起，

`tcg`

​	各种特性来改变tcg加速器的行为。

![image-20230926114324882](D:\File Storage\TyporaStorage\image-20230926114324882.png)

 `async-teardown`

​	根据`enabled`属性(值`yes`或`no`)启用或禁用QEMU异步拆解以改善客户机上的内存回收。自9.6.0起(仅限QEMU)



### 1.18 (Time keeping)

客户机时钟通常是从主机时钟初始化的。大多数操作系统期望硬件时钟保持在UTC，这是默认的。然而，Windows期望它是在所谓的"本地时间"。

```xml
...
<clock offset='localtime'>
  <timer name='rtc' tickpolicy='catchup' track='guest'>
    <catchup threshold='123' slew='120' limit='10000'/>
  </timer>
  <timer name='pit' tickpolicy='delay'/>
</clock>
...
```

`clock`

​	`offset`属性有四个可能的值，允许对如何将客户机时钟同步到主机进行细粒度控制。注意，并非所有hypervisor都支持所有模式。

​	`utc`

​		当启动时，客户时钟将始终同步到utc。从0.9.11开始，` utc `模式可以转换为` variable `模式，这可以通过使用调整属性来控制。如果值为'reset'，则永远不会完成转换(不是所有hypervisor都可以在每次启动时同步UTC;在这些hypervisor上使用` reset `将导致错误)。数值强制转换为` variable `模式，使用该值作为初始调整。默认的调整是特定于hypervisor的。

​	`localtime`

​		客户时钟将在启动时同步到主机配置的时区(如果有的话)。从0.9.11开始，调整属性的行为与在` utc `模式下相同。

​	`timezone`

​		使用`timezone`属性将客户时钟同步到请求的时区。自0.7.7

​	`variable`

​		客户时钟将根据`basis`属性，相对于UTC或本地时间应用任意偏移。相对于UTC(或本地时间)的增量是使用`adjustment`属性以秒为单位指定的。客户机可以随时调整RTC，并期望在下次重新启动时兑现。这与"utc"和"localtime"模式(可选属性adjustment="重置")形成对比，后者在每次重新启动时都会丢失RTC调整。自0.7.7自0.9.11以来，`basis`属性可以是"utc"(默认值)或"localtime"。

​	`absolute`

​		客户时钟在域启动时将始终设置为`start`属性的值。`start`属性的参数是epoch时间戳。8.4.0起。

一个`clock`可以有零个或多个定时器子元素。自0.8.0起

 `timer`

​	每个定时器元素都需要一个`name`属性，并具有其他依赖于指定`name`的可选属性。各种hypervisor支持不同的属性组合。

​	`name`

​		`name`属性选择要修改的计时器，可以是"platform"(当前不支持)、"hpet"(xen，qemu，lxc)、"kvmclock"(qemu)、"pit"(qemo)、"rtc"(qemu， lxc)"tsc"(xen，qemu-自3.2.0起)、"hyperclock"(qemu，自1.2.2起)或"armvtimer"(qemcu，自6.1.0起)。`hypervclock`计时器为运行Microsoft Windows操作系统的客户机添加了对iTSC功能的参考时间计数器和参考页面的支持。

​	`track`

​		`track`属性指定计时器跟踪的内容，可以是"boot"、"guest"、"wall"或"realtime"。仅对name="rtc"或name="platform"有效。

​	`tickpolicy`

​		`tickpolicy`属性确定当QEMU错过向客户机注入tick的最后期限时会发生什么。例如，这可能是因为客户机被暂停。

​		`delay`

​			继续以正常速率发送tick。guest操作系统不会注意到任何问题，因为从它的角度来看，时间将继续正常流动。guest的时间现在应该正好落后于host的时间，刚好是错过ticks的时间。

​		`catchup`

​			以更高的速率发送tick，以赶上错过的tick。guest OS不会注意到任何问题，因为从它的角度来看，时间将继续正常流动。一旦计时器设法赶上了所有丢失的tick，guest和host中的时间应该匹配。

​		`merge`

​			将遗漏的tick合并为一个tick并注入。guest时间可能会延迟，具体取决于操作系统对tick合并的响应

​		`discard`

​			扔掉错过的tick，继续正常的未来注入。guest OS将看到计时器一次提前一个潜在的相当大的量，就好像干预的时间块根本不存在一样；更不用说，这样的突然跳跃很容易混淆没有专门准备处理它的guest OS。假设guest OS能够正确处理时间跳跃，那么guest和host中的时间现在应该匹配。

​	如果策略是"catchup"，`catchup`子元素中可以有更多详细信息

​		`catchup`

​			`catchup`元素有三个可选属性，每个属性都是正整数。属性包括`threshold，slew`和`limit`。

​	请注意，hypervisors不需要支持所有时间源中的所有策略		

​	`frequency`

​		`frequency`属性是一个无符号整数，指定`name="tsc"`运行的频率。

​	`mode`

​		`mode`属性控制`name="tsc"`计时器的管理方式，可以是"auto"、"native"、"emulate"、"paravirt"或"smpsafe"。总是模拟其他计时器。

​	`present`

​		`present`属性可以是"yes"或"no"，以指定特定计时器是否可用于guest。



### 1.19 (Performance monitoring events)

一些平台允许监控虚拟机的性能和内部执行的代码。若要启用性能监视事件，可以在`perf`元素中指定它们，也可以通过`virDomainSetPerfEvents` API启用它们。然后使用virConnectGetAllDomainStats API检索性能值。自2.0.0起.

```xml
...
<perf>
  <event name='cmt' enabled='yes'/>
  <event name='mbmt' enabled='no'/>
  <event name='mbml' enabled='yes'/>
  <event name='cpu_cycles' enabled='no'/>
  <event name='instructions' enabled='yes'/>
  <event name='cache_references' enabled='no'/>
  <event name='cache_misses' enabled='no'/>
  <event name='branch_instructions' enabled='no'/>
  <event name='branch_misses' enabled='no'/>
  <event name='bus_cycles' enabled='no'/>
  <event name='stalled_cycles_frontend' enabled='no'/>
  <event name='stalled_cycles_backend' enabled='no'/>
  <event name='ref_cpu_cycles' enabled='no'/>
  <event name='cpu_clock' enabled='no'/>
  <event name='task_clock' enabled='no'/>
  <event name='page_faults' enabled='no'/>
  <event name='context_switches' enabled='no'/>
  <event name='cpu_migrations' enabled='no'/>
  <event name='page_faults_min' enabled='no'/>
  <event name='page_faults_maj' enabled='no'/>
  <event name='alignment_faults' enabled='no'/>
  <event name='emulation_faults' enabled='no'/>
</perf>
...
```

![image-20230926141923776](D:\File Storage\TyporaStorage\image-20230926141923776.png)



### 1.20 (Devices)

最后一组XML元素都用于描述提供给客户域的设备。所有设备都作为主`device`元素的子元素出现。自0.1.3起

```xml
...
<devices>
  <emulator>/usr/lib/xen/bin/qemu-dm</emulator>
</devices>
...
```

`emulator`

​	`emulator`元素的内容指定设备模型模拟器二进制文件的完全限定路径。 [capabilities XML](https://libvirt.org/formatcaps.html) 指定了建议用于每个特定域类型/体系结构组合的默认仿真器。

为了帮助用户识别他们关心的设备，每个设备都可以有直接的子别名`alias`元素，然后该元素具有`name`属性，用户可以在其中存储设备的标识符。标识符必须有"ua-"前缀，并且在域中必须是唯一的。此外，标识符必须仅由以下字符组成：`[a-zA-Z0-9_-]`。自3.9.0

```xml
<devices>
  <disk type='file'>
    <alias name='ua-myDisk'/>
  </disk>
  <interface type='network' trustGuestRxFilters='yes'>
    <alias name='ua-myNIC'/>
  </interface>
  ...
</devices>
```



#### 	1.20.1 (Hard drives， floppy disks， CDROMs)

任何看起来像磁盘的设备，无论是软盘、硬盘、cdrom还是半虚拟化驱动程序，都是通过`disk`元素指定的。

```xml
...
<devices>
  <disk type='file' snapshot='external'>
    <driver name="tap" type="aio" cache="default"/>
    <source file='/var/lib/xen/images/fv0' startupPolicy='optional'>
      <seclabel relabel='no'/>
    </source>
    <target dev='hda' bus='ide'/>
    <iotune>
      <total_bytes_sec>10000000</total_bytes_sec>
      <read_iops_sec>400000</read_iops_sec>
      <write_iops_sec>100000</write_iops_sec>
    </iotune>
    <boot order='2'/>
    <encryption type='...'>
      ...
    </encryption>
    <shareable/>
    <serial>
      ...
    </serial>
  </disk>
    ...
  <disk type='network'>
    <driver name="qemu" type="raw" io="threads" ioeventfd="on" event_idx="off"/>
    <source protocol="sheepdog" name="image_name">
      <host name="hostname" port="7000"/>
    </source>
    <target dev="hdb" bus="ide"/>
    <boot order='1'/>
    <transient/>
    <address type='drive' controller='0' bus='1' unit='0'/>
  </disk>
  <disk type='network'>
    <driver name="qemu" type="raw"/>
    <source protocol="rbd" name="image_name2">
      <host name="hostname" port="7000"/>
      <snapshot name="snapname"/>
      <config file="/path/to/file"/>
      <auth username='myuser'>
        <secret type='ceph' usage='mypassid'/>
      </auth>
    </source>
    <target dev="hdc" bus="ide"/>
  </disk>
  <disk type='block' device='cdrom'>
    <driver name='qemu' type='raw'/>
    <target dev='hdd' bus='ide' tray='open'/>
    <readonly/>
  </disk>
  <disk type='network' device='cdrom'>
    <driver name='qemu' type='raw'/>
    <source protocol="http" name="url_path" query="foo=bar&amp;baz=flurb>
      <host name="hostname" port="80"/>
      <cookies>
        <cookie name="test">somevalue</cookie>
      </cookies>
      <readahead size='65536'/>
      <timeout seconds='6'/>
    </source>
    <target dev='hde' bus='ide' tray='open'/>
    <readonly/>
  </disk>
  <disk type='network' device='cdrom'>
    <driver name='qemu' type='raw'/>
    <source protocol="https" name="url_path">
      <host name="hostname" port="443"/>
      <ssl verify="no"/>
    </source>
    <target dev='hdf' bus='ide' tray='open'/>
    <readonly/>
  </disk>
  <disk type='network' device='cdrom'>
    <driver name='qemu' type='raw'/>
    <source protocol="ftp" name="url_path">
      <host name="hostname" port="21"/>
    </source>
    <target dev='hdg' bus='ide' tray='open'/>
    <readonly/>
  </disk>
  <disk type='network' device='cdrom'>
    <driver name='qemu' type='raw'/>
    <source protocol="ftps" name="url_path">
      <host name="hostname" port="990"/>
    </source>
    <target dev='hdh' bus='ide' tray='open'/>
    <readonly/>
  </disk>
  <disk type='network' device='cdrom'>
    <driver name='qemu' type='raw'/>
    <source protocol="tftp" name="url_path">
      <host name="hostname" port="69"/>
    </source>
    <target dev='hdi' bus='ide' tray='open' rotation_rate='7200'/>
    <readonly/>
  </disk>
  <disk type='block' device='lun'>
    <driver name='qemu' type='raw'/>
    <source dev='/dev/sda'>
      <slices>
        <slice type='storage' offset='12345' size='123'/>
      </slices>
      <reservations managed='no'>
        <source type='unix' path='/path/to/qemu-pr-helper' mode='client'/>
      </reservations>
    </source>
    <target dev='sda' bus='scsi' rotation_rate='1'/>
    <address type='drive' controller='0' bus='0' target='3' unit='0'/>
  </disk>
  <disk type='block' device='disk'>
    <driver name='qemu' type='raw'/>
    <source dev='/dev/sda'/>
    <geometry cyls='16383' heads='16' secs='63' trans='lba'/>
    <blockio logical_block_size='512' physical_block_size='4096' discard_granularity='4096'/>
    <target dev='hdj' bus='ide'/>
  </disk>
  <disk type='volume' device='disk'>
    <driver name='qemu' type='raw'/>
    <source pool='blk-pool0' volume='blk-pool0-vol0'/>
    <target dev='hdk' bus='ide'/>
  </disk>
  <disk type='network' device='disk'>
    <driver name='qemu' type='raw'/>
    <source protocol='iscsi' name='iqn.2013-07.com.example:iscsi-nopool/2'>
      <host name='example.com' port='3260'/>
      <auth username='myuser'>
        <secret type='iscsi' usage='libvirtiscsi'/>
      </auth>
    </source>
    <target dev='vda' bus='virtio'/>
  </disk>
  <disk type='network' device='lun'>
    <driver name='qemu' type='raw'/>
    <source protocol='iscsi' name='iqn.2013-07.com.example:iscsi-nopool/1'>
      <host name='example.com' port='3260'/>
      <auth username='myuser'>
        <secret type='iscsi' usage='libvirtiscsi'/>
      </auth>
    </source>
    <target dev='sdb' bus='scsi'/>
  </disk>
  <disk type='network' device='disk'>
    <driver name='qemu' type='raw'/>
    <source protocol='nfs' name='PATH'>
      <host name='example.com'/>
      <identity user='USER' group='GROUP'/>
    </source>
    <target dev='vda' bus='virtio'/>
  </disk>
  <disk type='network' device='lun'>
    <driver name='qemu' type='raw'/>
    <source protocol='iscsi' name='iqn.2013-07.com.example:iscsi-nopool/0'>
      <host name='example.com' port='3260'/>
      <initiator>
        <iqn name='iqn.2013-07.com.example:client'/>
      </initiator>
    </source>
    <target dev='sdb' bus='scsi'/>
  </disk>
  <disk type='dir' device='floppy'>
    <driver name='qemu' type='fat'/>
    <source dir='/var/somefiles'>
    <target dev='fda'/>
    <readonly/>
  </disk>
  <disk type='volume' device='disk'>
    <driver name='qemu' type='raw'/>
    <source pool='iscsi-pool' volume='unit:0:0:1' mode='host'/>
    <target dev='vdb' bus='virtio'/>
  </disk>
  <disk type='volume' device='disk'>
    <driver name='qemu' type='raw'/>
    <source pool='iscsi-pool' volume='unit:0:0:2' mode='direct'/>
    <target dev='vdc' bus='virtio'/>
  </disk>
  <disk type='file' device='disk'>
    <driver name='qemu' type='qcow2' queues='4' queue_size='256' />
    <source file='/var/lib/libvirt/images/domain.qcow'/>
    <backingStore type='file'>
      <format type='qcow2'/>
      <source file='/var/lib/libvirt/images/snapshot.qcow'/>
      <backingStore type='block'>
        <format type='raw'/>
        <source dev='/dev/mapper/base'/>
        <backingStore/>
      </backingStore>
    </backingStore>
    <target dev='vdd' bus='virtio'/>
  </disk>
  <disk type='nvme' device='disk'>
    <driver name='qemu' type='raw'/>
    <source type='pci' managed='yes' namespace='1'>
      <address domain='0x0000' bus='0x01' slot='0x00' function='0x0'/>
    </source>
    <target dev='vde' bus='virtio'/>
  </disk>
  <disk type='vhostuser' device='disk'>
    <driver name='qemu' type='raw'/>
    <source type='unix' path='/tmp/vhost-blk.sock'>
      <reconnect enabled='yes' timeout='10'/>
    </source>
    <target dev='vdf' bus='virtio'/>
  </disk>
</devices>
...
```

`disk`

​	`disk`元素是描述磁盘的主要容器，支持以下属性：

​	`type`

​		有效值为"file"、"block"、"dir"(自0.7.5起)、"network"(自0.8.7起)、或"volume"(从1.0.5起)、或者"nvme"(自6.0.0起)或"vhostuser"(自7.1.0起)，并指代磁盘的底层源。自0.0.3起

​	`device`

​		指示如何向guest操作系统公开磁盘。此属性的可能值为"floppy"、"disk"、"cdrom"和"lun"，默认值为"disk"。

​		使用"lun"(自0.9.10起)仅在协议="scsi"的类型为"块"或"网络"时有效，或者在使用iscsi源池作为模式"主机"或作为使用光纤通道存储池的NPIV虚拟主机总线适配器(vHBA)时类型为"卷"时有效。以这种方式配置，LUN的行为与"磁盘"相同，只是接受来自guest的通用SCSI命令并将其传递到物理设备。还要注意的是，device='lun'只会被实际的原始设备识别，而不会被单独的分区或LVM分区识别(在这种情况下，内核会拒绝通用SCSI命令，使其与device='disk'相同)。自0.1.4

​	`model`

​		指示磁盘的模拟设备型号。通常，这只由总线属性表示，但对于总线"virtio"，可以用"virtiotransitional"、"virtionon-transitional"或"virtio"进一步指定模型。有关更多详细信息，请参阅Virtio过渡设备。自5.2.0起

​	`rawio`

​		指示磁盘是否需要rawio功能。有效设置为"是"或"否"(默认为"否")。如果域中的任何一个磁盘的rawio="yes"，则将为域中的所有磁盘启用rawio功能(因为在QEMU的情况下，只能根据每个进程设置此功能)。此属性仅在设备为"lun"时有效。注意，rawio打算限制每个设备的能力，然而，当前的QEMU实现为域进程提供了比这更广泛的能力(基于每个进程，影响所有域磁盘)。为了在这个阶段尽可能地限制QEMU驱动程序的功能，建议使用`sgio`，它比`rawio`更安全。自0.9.10

​	`sgio`

​		如果hypervisor和操作系统支持，则指示是否为磁盘筛选无特权的SG_IO命令。有效设置为"filtered"已筛选或"unfiltered"未筛选，默认设置为"filtered"。仅当`device`为"lun"时可用。自1.0.2起

​	`snapshot`

​		指示磁盘快照过程中磁盘的默认行为：`internal`需要一种文件格式，如qcow2，可以存储快照和自快照以来的数据更改；`external`将快照与实时数据分离；否表示磁盘不会参与快照；和手动允许通过非托管存储提供程序执行快照。只读磁盘默认为`no`，而其他磁盘的默认值取决于hypervisor的功能。在 [domain snapshot creation](https://libvirt.org/formatsnapshot.html)的过程中，某些hypervisor还允许按快照进行选择。并非所有快照模式都受支持；例如，使用临时磁盘启用快照通常没有意义。自0.9.5

`source`

​	磁盘`source`的表示取决于磁盘类型属性值，如下所示：

​	`file`

​		`file`属性指定存放磁盘的文件的完全限定路径。自0.0.3起

​		9.0.0起，可以添加一个新的可选属性`fdgroup`，指示通过`virDomainFDAssociate()`API通过与域对象关联的文件描述符访问磁盘，而不是打开文件。libvirt不一定要通过文件系统访问这些文件。在执行块操作时，通过`file`传递的文件名仍然可以用于生成写入镜像元数据的路径，但libvirt不会以本机方式访问这些路径。

​	`block`

​		`dev`属性指定要用作磁盘的主机设备的完全限定路径。自0.0.3起

​	`dir`

​		`dir`属性指定要用作磁盘的目录的完全限定路径。自0.7.5

​		请注意，大多数支持目录磁盘的hypervisor都是通过公开模拟块设备来实现这一点的，其中模拟文件系统填充了配置目录的内容。由于客户机操作系统可能会缓存文件系统元数据，因此对目录的外部更改可能不会出现在客户机中，和/或可能导致可从VM观察到损坏的数据。

​		模拟文件系统的格式由<driver>元素的`format`属性控制。目前只支持`fat`格式。Hypervisor可能只支持<readonly/>模式。

​	`network`

​		`protocol`属性指定访问请求的映像的协议。可能的值有"nbd"、"iscsi"、"rbd"、"sheepdog"、"gluster"、"vxhs"、"nfs、"http"、"https"、"ftp"、"ftps"或"tftp"。

​		对于`nbd`以外的任何协议`protocol`，必须使用附加属性`name`来指定将使用的卷/映像。

​		对于"nbd"，`name`属性是可选的。可以通过将`tls`属性设置为`yes`来启用NBD的TLS传输。对于QEMU hypervisor，TLS环境的使用也可以通过/etc/libvirt/QEMU.conf中的`nbd_TLS`和`nbd_TLS_x509_cert_dir`在主机上进行全局控制。(自4.5.0起为'TLS')自8.2.0起，可选属性`tlsHostname`可用于覆盖用于TLS证书验证的nbd服务器的预期主机名。

​		对于`http`和`https`协议，可选的属性`query`指定查询字符串。(自6.2.0起)

​		对于"iscsi"(自1.0.4起)，`name`属性可能包括一个逻辑单元号，用斜线与目标名称隔开(例如，iqn.2013-07.com.example:iscsi pool/1)。如果未指定，则默认LUN为零。

​		对于"vxhs"(自3.8.0起)，`name`是由HyperScale服务器分配的卷的UUID。此外，可选属性`tls`(仅限QEMU)可用于控制VxHS块设备是否会利用hypervisor配置的tls X.509证书环境来加密数据通道。对于QEMU虚拟机监控程序，TLS环境的使用也可以通过文件`/etc/libvirt/QEMU.conf`中的 `vxhs_TLS` 和 `vxhs_TLS_x509_cert_dir` 或`default_TLS_x509 _cert_dr`设置在主机上进行全局控制。如果启用了`vxhs_TLS`，则除非域`TLS`属性设置为"no"，否则libvirt将使用主机配置的TLS环境。如果`tls`属性设置为"yes"，则无论qemu.conf设置如何，都将尝试tls身份验证。

​		自0.8.7

​	`volume`

​		底层磁盘源由属性`pool`和`volume`表示。属性`pool`指定磁盘源所在的存储池(由libvirt管理)的名称。属性`volume`指定用作磁盘源的存储卷(由libvirt管理)的名称。`volume`属性的值将是`virsh-vol-list[pool Name]`命令的“Name”列的输出。

​		使用属性`mode`(自1.1.1起)指示如何将LUN表示为磁盘源。有效值为“direct”和“host”。如果未指定`mode`，则默认使用“主机”。使用“direct”作为`mode`值表示使用 [storage pool's](https://libvirt.org/formatstorage.html)的`source`元素`host`属性作为磁盘源来生成libscsi URI(例如“file”=iscsi://example.com:3260/iqn.2013-07.com.example:iscsi pool/1')。使用“host”作为`mode`值表示使用LUN在主机上显示的路径(例如，'file=/dev/disk/by-path/ip-example.com:3260 iscsi iqn.2013-07.com.example:iscsi-pol-LUN-1')。使用iscsi资源池中的LUN提供了与使用`type`为“block”或“network”配置的磁盘相同的功能，并且`device`为“lun”并且可以由guest使用。自1.0.5起

​	`nvme`

​		要为NVMe磁盘指定磁盘源，源元素具有以下属性：

​		`type`

​			在`address`子元素中指定的地址类型。目前，只接受`pci`值。

​		`managed`

​			此属性指示libvirt在域启动时自动分离NVMe控制器(`yes`)，或者期望系统管理员分离控制器(`no`)。

​		`namespace`

​			应分配给域的命名空间ID。根据NVMe标准，命名空间编号从1开始，包括。

​	`<disk-type='nvme'>`和`<hostdev/>`的区别在于，后者是具有所有限制的普通主机设备分配(例如，没有实时迁移)，而前者使hypervisor通过hypervisor的块层运行nvme磁盘，从而启用该层提供的所有功能(例如，快照、域迁移等)。此外，由于nvme磁盘与其PCI驱动程序没有绑定，主机内核存储堆栈不涉及(与通过`<disk-type='block'>`传递say `/dev/nvme0n1`相比)，因此可以实现更低的延迟。

​	`vhostuser`

​		使hypervisor能够使用vhost用户协议连接到另一个进程。需要为VM配置共享内存，有关更多详细信息，请参阅`memoryBacking`元素的`access`模式(请参阅1.8[Memory Backing](https://libvirt.org/formatdomain.html#memory-backing))。

​	`source`元素具有以下强制属性：

​		`type`

​			字符设备的类型。目前只支持unix类型。

​		`path`

​			要用作磁盘源的unixsocket的路径。

请注意，vhost服务器同时替换了磁盘前端和后端，因此几乎所有的磁盘属性都无法通过<disk>XML对此磁盘类型进行配置。此外，此磁盘类型不支持块作业、增量备份和快照等功能。

对于“file”、“block”和“volume”，可以使用一个或多个可选的子元素`seclabel`(请参阅1.21[Security label](https://libvirt.org/formatdomain.html#security-label))来覆盖仅针对该源文件的域安全标签策略。(注意，对于“卷”类型的磁盘，`seclabel`仅在指定的存储卷为“文件”或“块”类型时有效)。自0.9.9

`source`元素也可以具有与backingStore的`index`属性相同语义的`index`属性。
`source`元素可以包含以下子元素:

​	`host`

​		当磁盘`type`为“network”时，`source`可以有零个或多个`host`子元素，用于指定要连接的主机。`host`元素支持4个属性，即:"name"、"port"、"transport"和"socket"，分别指定主机名、端口号、传输类型和到socket的路径。此元素的含义和元素的数量取决于协议属性。

![image-20230927094805622](D:\File Storage\TyporaStorage\image-20230927094805622.png)

​		Gluster支持“tcp”、“rdma”、“unix”作为传输属性的有效值。NBD支持“tcp”和“unix”。其他仅支持“tcp”。如果没有指定，则假定为“tcp”。如果传输是“unix”，socket属性指定AF_UNIXsocket的路径。NFS只支持使用“tcp”传输，根本不支持使用端口，因此必须省略它。

​	`snapshot`

​		`snapshot`元素的`name`属性可以指定一个内部快照名称，作为存储协议的源。从1.2.11开始支持` rbd `(仅QEMU)。	

​	`config`

​		元素的`file`属性提供了指向配置文件的完全限定路径，该配置文件将作为参数提供给网络存储协议的客户端。从1.2.11开始支持` rbd `(仅QEMU)。

​	`auth`

​		从libvirt 3.9.0开始，`auth`元素支持磁盘`type`“network”，该类型使用带有协议属性“rbd”或“iscsi”的`source`元素。如果存在`auth`元素，则提供访问源所需的身份验证凭据。它包括一个强制属性`username`，用于标识在身份验证期间使用的用户名，以及一个带有强制属性`type`的子元素`secret`，用于绑定到一个持有实际密码或其他凭据的 [libvirt secret object](https://libvirt.org/formatsecret.html) (域XML故意不公开密码，只公开对管理密码的对象的引用)。已知的秘密类型是“ceph”用于ceph RBD网络源，“iscsi”用于iscsi目标的CHAP认证。两者都需要与秘密对象的uuid相匹配的`uuid`属性，或者与秘密对象中指定的键相匹配的`usage`属性。

​	`encryption`

​		从libvirt 3.9.0开始，对于加密的存储源，`encryption`可以是`source`元素的子元素。如果存在，则指定如何加密存储源，请参阅[Storage Encryption](https://libvirt.org/formatstorageencryption.html) 页面以获取更多信息。请注意，加密的` qcow `格式已被破坏，因此不再支持与磁盘映像一起使用。(从libvirt 4.5.0开始)

​	`reservations`

​		从libvirt 4.4.0开始，`reservations`可以是用于存储源(仅QEMU驱动程序)的`source`元素的子元素。如果存在，则支持对基于SCSI的磁盘进行持久预留。元素有一个强制属性`managed`，使用接受的值`yes`和`no`进行管理。如果启用了`managed`，  libvirt准备和管理所需的任何资源。在不管理持久预留时，hypervisor充当客户机，必须在子元素`source`中提供到服务器socket的路径，该子元素源当前仅接受以下属性:`type`(一个值`unix`)、`path`(到socket的路径)和`mode`(接受一个值`client`(指定hypervisor的角色)。建议允许libvirt管理持久预留。

​	`initiator`启动器

​		从libvirt 4.7.0开始，`initiator`元素支持磁盘`type`“network”，它使用带有协议属性“iscsi”的`source`元素。如果存在，initiator元素提供了通过强制属性`name`访问源所需的启动器IQN。

​	`address`

​		对于nvme类型的磁盘，此元素指定主机nvme控制器的PCI地址。6.0.0起

​	`slices`

​		使用`slice`子元素对`slices`元素进行切片，因此允许配置镜像格式(`slice type='storage'`)在存储源中的位置偏移量和大小，或镜像格式容器中的guest数据(未来扩展)。`offset`和`size`值的单位是字节。自6.1.0起

​	 `ssl`

​		对于`https`和`ftps`访问的存储，可以使用此元素调整SSL传输参数。`verify`属性允许打开或关闭SSL证书验证。支持的取值为`yes`和`no`。自6.2.0

​	`cookies`

​		针对`http`和`https`访问的存储可以传递一个或多个cookie。cookie的名称和值必须符合HTTP规范。自6.2.0

​	`readahead`

​		`readahead`成员有一个`size`属性，用于指定支持预读缓冲区的字节大小。注意，` 0 `被认为是没有提供值。从6.2.0 开始

​	`timeout`

​		`timeout`元素有一个`seconds`属性，用于指定支持的协议的连接超时时间，单位为秒。注意，` 0 `被认为是没有提供值。自6.2.0

​	`identity`

​		当使用`nfs`协议时，它用于提供有关用户和组的配置信息。元素有两个属性，`user`和`group`。用户可以将这些元素提供为用户或组字符串，或者直接提供为用户和组ID号，如果字符串在ID号的开头使用“+”进行格式化的话。如果省略这些属性中的任何一个，则假定该字段是当前系统的默认值。如果`user`和`group`都是默认值，则可以省略整个元素。

​	`reconnect`

​		对于磁盘类型vhostuser，如果连接丢失，则配置重新连接超时。这是在启用两个强制属性和超时的情况下设置的。对于磁盘类型网络和协议nbd， QEMU nbd重新连接延迟可以通过属性delay设置:

​		`enabled`
​			如果启用了重新连接功能，则接受`yes`和`no`
​		`timeout`
​			hypervisor尝试重新连接的秒数。
​		`delay`
​			仅适用于NBD主机。所有请求暂停并在成功重新连接后重新运行的秒数。在此之后，任何延迟的请求以及成功重新连接之前的所有未来请求都将立即失败。如果不设置，QEMU的默认值为0。

​	 对于代表cdrom或软盘(`device`属性)的“file”或“volume”磁盘类型，可以定义在源文件不可访问时如何处理磁盘的策略。(注意，`startupPolicy`对卷volume类型的磁盘无效，除非指定的存储卷是文件file类型的)。这是由`startupPolicy`属性完成的(从0.9.7开始)，接受这些值:

![image-20230927101632746](D:\File Storage\TyporaStorage\image-20230927101632746.png)

​	从1.1.2开始，`startupPolicy`扩展到支持除cdrom和软盘floppy之外的硬盘。在客户端冷启动时，如果某个磁盘不可访问或其磁盘链被破坏，使用startupPolicy 'optional'，客户端将丢弃该磁盘。此功能目前不支持迁移。

`backingStore`

​	该成员描述由同级`source`元素指定的磁盘使用的后备存储器。自1.2.4起，如果hypervisor驱动程序不支持[backingStoreInput](https://libvirt.org/formatdomaincaps.html#backingstoreinput)(自5.10.0起)域特性，则在输入时将忽略`backingStore`，仅用于输出，以描述检测到的运行域的后备链。如果支持`backingStoreInput`，则`backingStore`用作`source`或其他`backingStore`的后备映像，后备镜像元数据中记录的任何后备映像信息。空的`backingStore`元素意味着兄弟源是自包含的，不基于任何后备存储器。为了使检测到的后备链信息准确，后备链的格式必须正确地指定在链的每个文件的元数据中(libvirt创建的文件满足此属性，但使用现有的外部文件进行快照或块复制操作需要最终用户正确地预先创建文件)。`backingStore`支持以下属性:

​	`type`

​		`type`属性表示后备存储使用的磁盘类型，有关详细信息和可能的值，请参阅上面的磁盘类型属性。

​	`index`

​		该属性仅在输出时有效(在输入时被忽略)，并且在执行块操作(例如通过`virDomainBlockRebase` API)时，可以使用它来引用磁盘链的特定部分。例如，`vda[2]`指具有vda目标的磁盘的`index='2'`的后备存储。

此外，`backingStore`还支持以下子元素:

​	`format`

​		`format`元素包含`type`属性，该属性指定后备存储的内部格式，如`raw`或`qcow2`。

​		`format`元素可以包含`metadata_cache`子元素，它与磁盘驱动程序的同名子元素具有相同的语义。

​	`source`

​		该元素与`disk`中的`source`元素具有相同的结构。它指定哪个文件、设备或网络位置包含所描述的后备存储的数据。

​	`backingStore`

​		如果后备存储不是自包含的，则链中的下一个元素由嵌套的`backingStore`元素描述。

`mirror`

​	如果hypervisor已经启动了一个长时间运行的块作业操作，则会出现该元素，其中`source`子元素中的镜像位置最终将具有与源相同的内容，并且文件格式为子元素`format`(可能与源的格式不同)。`source`子元素的细节由镜像的`type`属性决定，类似于对整个`disk`设备元素所做的工作。`job`属性提到哪个API开始了操作(对于`virDomainBlockRebase` API是“copy”，对于`virDomainBlockCommit` API是“active-commit”)，从1.2.7开始。属性`ready`(如果存在)用于跟踪作业的进度:`yes`(如果磁盘已经准备好进行pivot操作)，或者从1.2.7开始，`abort`或`pivot`(如果作业正在完成过程中)。如果没有`ready`，则磁盘可能仍在复制。目前，这个元素只在输出中有效;它在输入时被忽略。`source`子元素适用于从1.2.6开始的所有两阶段作业。旧的libvirt只支持块复制到文件，从0.9.12开始;为了与老客户端兼容，这样的作业包括属性`file`中的冗余信息和`mirror`元素中的`format`。

 `target`

​	`target`元素控制总线/设备，在总线/设备 bus / driver下，磁盘可以暴露给guest OS。`dev`属性表示“逻辑”设备名。不保证指定的实际设备名称映射到guest OS中的设备名称。将其视为设备排序提示。可选的`bus`属性指定要仿真的磁盘设备的类型。可能的值是特定于驱动程序的，典型值是"ide"， "scsi"， "virtio"， "xen"， "usb"， "sata"，或"sd" ，"sd"从1.1.2开始。如果省略，总线类型将从设备名称的样式推断(例如，名为“sda”的设备通常将使用SCSI总线导出)。可选属性`tray`表示可移动磁盘(即光盘或软盘)的托盘状态，取值为"open"或"closed"，默认值为"closed"。注意，`tray`的值可以在域运行时更新。可选属性`removable`用于设置USB或SCSI磁盘的可移动标志，取值为`on`或`off`，默认值为`off`。可选属性`rotation_rate`设置SCSI、IDE或SATA总线上磁盘的存储旋转速率。1025 ~ 65534为介质转速，单位为转/分。值1用于表示固态或非旋转存储。这些值不需要匹配底层主机存储的值。0.0.3起;`bus`属性自0.0.3 ;`tray`属性自0.9.11; “usb”属性值0.4.4之后; “sata”属性值从0.9.7开始; “removable”属性值自1.1.3; "rotation_rate"属性从7.3.0开始的值

`iotune`

​	可选的`iotune`元素提供了提供额外的每设备I/O调整的能力，其值可能因每个设备而异(与`blkiotune`元素形成对比(请参阅1.11 [Block I/O Tuning](https://libvirt.org/formatdomain.html#block-i-o-tuning))，后者全局应用于域)。目前，唯一可用的调优是qemu的块I/O节流。该元素具有可选的子元素；任何未指定或给定值为0的子元素都意味着没有限制。自0.9.8

​	`total_bytes_sec`

​		可选的`total_bytes_sec`元素是以字节每秒为单位的总吞吐量限制。这不能与`read_bytes_sec`或`write_bytes_sec`一起出现。

​	`read_bytes_sec`

​		可选的`read_bytes_sec`元素是以字节每秒为单位的读取吞吐量限制。

​	`write_bytes_sec`

​		可选的`write_bytes_sec`元素是以字节每秒为单位的写入吞吐量限制。

​	`total_iops_sec`

​		可选的`total_iops_sec`元素是每秒的I/O操作总数。这不能与`read_iops_sec`或`write_iops_sec`一起出现。

​	`read_iops_sec`

​		可选的`read_iops_sec`元素是每秒的读取I/O操作。

​	`write_iops_sec`

​		可选的`write_iops_sec`元素是每秒的写I/O操作。

​	`total_bytes_sec_max`

​		可选的`total_bytes_sec_max`元素是以字节每秒为单位的最大总吞吐量限制。这不能与`read_bytes_sec_max`或	`write_bytes_se_max`一起出现。

​	`read_bytes_sec_max`

​		可选的`read_bytes_sec_max`元素是以字节每秒为单位的最大读取吞吐量限制。

​	`write_bytes_sec_max`

​		可选的`write_bytes_sec_max`元素是以字节每秒为单位的最大写入吞吐量限制。

​	`total_iops_sec_max`

​		可选的`total_iops_sec_max`元素是每秒I/O操作总数的最大值。这不能与`read_iops_sec_max`或`write_iops_sec_max`一起出现。

​	`read_iops_sec_max`

​		可选的`read_iops_sec_max`元素是每秒最大读取I/O操作数。

​	`write_iops_sec_max`

​		可选的`write_iops_sec_max`元素是每秒最大写入I/O操作数。

​	`size_iops_sec`

​		可选的`size_iops_sec`元素是每秒I/O操作的大小。

​	1.2.11和QEMU 1.7之后的吞吐量限制

​	`group_name`

​		可选的`group_name`提供了在多个驱动器之间共享I/O限制配额的功能。这可以防止最终用户通过将1个大驱动器拆分为N个小驱动器并获得正常限制配额的N倍来规避主机提供商的限制策略。可以使用任何名称。

​		group_name自3.0.0和QEMU 2.4起

​	`total_bytes_sec_max_length`

​		可选的`total_bytes_sec_max_length`元素是`total_betes_sec_max`突发周期的最大持续时间(以秒为单位)。仅在设置了`total_bytes_sec_max`时有效。

​	`read_bytes_sec_max_length`

​		可选的`read_bytes_sec_max_length`元素是`read_bytes_sec_max`突发周期的最大持续时间(以秒为单位)。仅当设置了`read_bytes_sec_max`时有效。

​	`write_bytes_sec_max`

​		可选的`write_bytes_sec_max_length`元素是`write_bytes_sec_max`突发周期的最大持续时间(以秒为单位)。仅在设置了`write_bytes_sec_max`时有效。

​	`total_iop_sec_max_length`

​		可选的`total_iops_sec_max_length`元素是`total_iops_sec_max`突发周期的最大持续时间(以秒为单位)。仅当设置了total_iops_sec_max时有效。

​	`read_iop_sec_max_length`

​		可选的`read_iops_sec_max_length`元素是`read_iop_sec_max`突发周期的最大持续时间(以秒为单位)。仅在设置了read_iops_sec_max时有效。

​	`write_iops_sec_max`

​		可选的`write_iops_sec_max_length`元素是`write_iop_sec_max`突发周期的最大持续时间(以秒为单位)。仅在设置了`write_iops_sec_max`时有效。

​	吞吐量长度z自2.4.0和QEMU 2.6起

`driver`

​	可选的驱动程序元素允许指定与用于提供磁盘的hypervisor驱动程序相关的进一步细节。自0.1.8

​	· 如果hypervisor支持多个后端驱动程序，则name属性选择主后端驱动程序名称，而可选`type`属性提供子类型。例如，xen支持名称“tap”、“tap2”、“phy”或“file”，类型为“aio”，而qemu只支持名称“qemu”，但支持多种类型，包括“原始”、“bochs”、“qcow2”和“qed”。

​	· 可选的缓存属性控制缓存机制，可能的值为“default”、“none”、“writethrough”、“回写”、“directsync”(类似于“writethroughs”，但它绕过了主机页缓存)和“unsafe”(主机可以缓存所有磁盘io，忽略来自guest的同步请求)。自0.6.0起，“directsync”自0.9.5起，“unsafe”自0.9.7起

​	· 可选的`error_policy`属性控制hyperivisor在磁盘读写错误时的行为，可能的值为“stop”、“report”、“ignore”和“enospace”。自0.8.0起，自0.9.7起“report”默认值由hypervisor自行决定。还有一个可选的`rerror_policy`，它仅控制读取错误的行为。自0.9.7起。如果没有给出错误策略，则错误策略用于读取和写入错误。如果给定了`rerror_policy`，它将覆盖读取错误的error_policy。还要注意的是，“enospace”不是读取错误的有效策略，因此，如果`error_policy`设置为“inospace”，并且没有给出`rerror_policy`，则读取错误策略将保留为默认值。

​	· 可选的`io`属性控制I/O上的特定策略；qemu-guests支持“线程”和“本机”自0.8.8，io_uring自6.3.0(qemu 5.0)。

​	· 可选的`ioeventfd` 属性允许用户设置磁盘设备的域I/O异步处理。默认值由hypervisor自行决定。接受的值为"on"和"off"。启用此功能允许QEMU在单独的线程处理I/O时执行虚拟机。通常，正在进行I/O操作的虚拟机会受益于此功能，因为它可以降低系统CPU利用率。但在负载过重的主机上，可能会增加虚拟机的I/O延迟。自0.9.3版(仅适用于QEMU和KVM)。一般情况下，除非 非常确定自己知道在做什么，否则应该不要触碰此选项。

​	· 可选的`event_idx`属性控制设备事件处理的某些方面。该值可以是"on"或"off" - 如果为"on"，将减少虚拟机的中断和退出次数。默认值由QEMU确定；通常情况下，如果支持该功能，则默认值为"on"。如果存在使此行为不太理想的情况，此属性提供了一种强制关闭该功能的方法。自0.9.5(仅适用于QEMU和KVM)。一般情况下，除非 非常确定自己知道在做什么，否则应该不要触碰此选项。

​	· 可选的`copy_on_read`属性控制是否将读取的后备文件复制到镜像文件中。该值可以是"on"或"off"。在后备文件位于较慢网络上时，通过复制读取可避免重复访问相同的后备文件扇区，因此这在一般情况下默认关闭。自0.9.10(仅适用于QEMU和KVM)。

​	· 可选的`discard`属性控制是否忽略或传递丢弃请求(也称为"trim"或"unmap")。该值可以是"unmap"(允许传递丢弃请求)或"ignore"(忽略丢弃请求)。自1.0.6(仅适用于QEMU和KVM)。

​	· 可选的`detect_zeroes`属性控制是否检测零写入请求。该值可以是"off"、"on"或"unmap"。前两个值分别关闭和开启检测。第三个值("unmap")开启检测，并根据上述丢弃选项的值尝试从Image `discard`这些区域(如果`discard`设置为"ignore"，它将作为"on"运行)。请注意，启用检测是一个计算密集型操作，但在慢速媒体上可以节省文件空间和/或时间。自2.0.0。

​	· 可选的`iothread`属性将磁盘分配给域`iothreads`值范围定义的IOThread(请参阅1.5 [IOThreads Allocation](https://libvirt.org/formatdomain.html#iothreads-allocation))。可以将多个磁盘分配给相同的IOThread，并从1编号到域iothreads值。仅适用于配置为使用"virtio" `bus`和"pci"或"ccw"`address`类型的磁盘设备`target`。自1.2.8(QEMU 2.1)。

​	· 可选的`queues`属性指定virtio-blk(自3.9.0)或vhost-user-blk(自7.1.0)的虚拟队列数量。

​	· 可选的`queue_size`属性指定virtio-blk或vhost-user-blk的每个虚拟队列大小(自7.8.0)。

​	· 对于virtio磁盘，也可以设置1.20.4[Virtio-related options](https://libvirt.org/formatdomain.html#virtio-related-options)(自3.5.0)。

​	· 可选的`metadata_cache`子元素控制与存储镜像元数据的特定缓存相关的方面。请注意，此设置仅适用于顶级镜像；`backingStore`的`format`元素的同名子元素可用于指定备份镜像的缓存设置。

​	自7.0.0版开始，可以通过`max_size`子元素来控制`qemu`虚拟机的`qcow2`格式驱动程序的元数据缓存的`max_size`(请参见下面的示例)。

​	可选的`discard_no_unref`属性可以设置为控制`qemu` hypervisor如何处理qcow2镜像中的客户机丢弃命令。启用后，来自客户机的丢弃请求将将qcow2群集标记为零，但将保留该群集的引用/偏移量。但它仍然会将丢弃传递到较低的层。这将解决qcow2镜像中的碎片问题。自9.5.0版(QEMU 8.1)。

​	在大多数情况下，hypervisor使用的默认配置足够了，因此不需要修改此设置。有关`qemu`虚拟机中`qcow2`元数据缓存行为的具体信息，请参阅`qemu` [qcow2 cache docs](https://git.qemu.org/?p=qemu.git;a=blob;f=docs/qcow2-cache.txt)。

​	Example:

```xml
<disk type='file' device='disk'>
  <driver name='qemu' type='qcow2'>
    <metadata_cache>
      <max_size unit='bytes'>1234</max_size>
    </metadata_cache>
  </driver>
  <source file='/var/lib/libvirt/images/domain.qcow'/>
  <backingStore type='file'>
    <format type='qcow2'>
      <metadata_cache>
        <max_size unit='bytes'>1234</max_size>
      </metadata_cache>
    </format>
    <source file='/var/lib/libvirt/images/snapshot.qcow'/>
    <backingStore/>
  </backingStore>
  <target dev='vdd' bus='virtio'/>
</disk>
```

`backenddomain`

​	可选的`backenddomain`元素允许指定承载磁盘的后端域(也称为驱动程序域)。使用`name`属性可以指定后端域名。自1.2.13起(仅限Xen)

`boot`

​	指定磁盘是可引导的。`order`属性确定在引导序列中尝试设备的顺序。在S390体系结构上，仅使用第一个引导设备。可选的`loadparm`属性是一个8个字符的字符串，guest可以在S390上通过sclp或diag 308查询该字符串。S390上的Linux客户机可以使用`loadparm`来选择boot entry。自3.5.0在BIOS引导加载程序部分，每个设备的`boot`元素不能与通用引导元素一起使用 [BIOS bootloader](https://libvirt.org/formatdomain.html#bios-bootloader)。自0.8.8

`encryption`

​	从libvirt 3.9.0开始，首选将encryption元素作为source元素的子元素。如果存在，它指定如何使用"qcow"加密卷。有关更多信息，请参阅[Storage Encryption](https://libvirt.org/formatstorageencryption.html)。

`readonly`

​	如果存在，表示设备不能由客户机修改。目前，对于具有属性`device='cdrom'`的磁盘，这是默认设置。

`shareable`

​	如果存在，表示设备预计在域之间共享(假设hypervisor和操作系统支持此功能)，这意味着该设备的缓存应该被停用。

`transient`

​	 如果存在，表示应在客户机退出时自动还原对设备内容的更改。对于某些hypervisor，将磁盘标记为transient可以阻止该域参与迁移、快照或块作业。仅支持vmx hypervisor(自0.9.5)和qemu hypervisor(自6.9.0)。

​	在源映像的<transient/>磁盘被假定为在多个同时运行的VM之间共享的情况下，可选的`shareBacking`属性应设置为是。请注意，hypervisor驱动程序可能需要热插拔此磁盘，因此仅适用于支持热插拔的配置。自7.4.0版开始。

`serial`

​	 如果存在，指定虚拟硬盘的序列号。例如，它可能看起来像<serial>WD-WMAP9A966149</serial>。不支持scsi-block设备，即那些使用磁盘类型'block'，使用`device`'lun'在`bus` 'scsi'上的设备。自0.7.1。

​	请注意，根据hypervisor和设备类型，序列号可能会被默默截断。IDE/SATA设备通常限制为20个字符。根据hypervisor版本的不同，SCSI设备可能限制为20、36或247个字符。

​	将来，hypervisor可能开始拒绝过长的序列号，而不是截断它们，因此建议通过测试所需的设备和hypervisor组合的所需序列号长度范围来避免隐式截断。

`wwn`

​	 如果存在，此元素指定虚拟硬盘或CD-ROM驱动器的WWN(World Wide Name, 全局唯一名称)。它必须由16个十六进制数字组成。自0.10.1。

`vendor`

​	 如果存在，此元素指定虚拟硬盘或CD-ROM设备的制造商。其长度不能超过8个可打印字符。自1.0.1。

`product`

​	 如果存在，此元素指定虚拟硬盘或CD-ROM设备的产品。其长度不能超过16个可打印字符。自1.0.1。

`address`

​	 如果存在，`address`元素将磁盘绑定到控制器的特定插槽(通常可以由libvirt推断实际的<controller>设备，尽管可以明确指定)。`type`属性是强制性的，通常为"pci"或"drive"。对于"pci"控制器，必须存在额外的属性`bus`、`slot`和`function`，以及可选的`domain`和`multifunction`。`multifunction`默认为'off'；任何其他值都需要QEMU 0.1.3和libvirt 0.9.7。对于"drive"控制器，可以使用额外的属性`controller、bus、target`(libvirt 0.9.11)和`unit`，每个属性默认为0。

`auth`

​	 从libvirt 3.9.0开始，首选将`auth`元素作为`source`元素的子元素。该元素仍然被视为磁盘的子元素并进行管理。将`auth`同时用作`disk`和`source`的子元素是无效的。`auth`元素在libvirt 0.9.7中作为磁盘的子元素引入。

`geometry`

​	可选的`geometry`元素提供了替代几何图元设置的功能。这对于S390 DASD磁盘或较旧的DOS磁盘非常有用。0.10.0

​	`cyls`

​		`cyls`属性是圆柱体cylinders的数量。

​	`heads`

​		heads属性是头的数量。

​	`secs`

​		`secs`属性是每个磁道的扇区数。

​	`trans`

​		可选`trans`属性是BIOS Translation Modus(none， lba 或 auto)

`blockio`

​	如果存在，`blockio`元素允许覆盖下面列出的任何块设备属性。自0.10.2(QEMU和KVM)

​	`logical_block_size`

​		磁盘将向guest OS报告的逻辑块大小。对于Linux，这将是BLKSSZGET ioctl返回的值，并描述磁盘I/O的最小单元。

​	`physical_block_size`

​		磁盘将向guest OS报告的物理块大小。对于Linux，这将是BLKPBSZGET ioctl返回的值，并描述磁盘的硬件扇区大小，该大小可能与磁盘数据的对齐有关。

​	`discard_gramularity`

​		在单个操作中可以丢弃的最小数据量。它会影响取消映射操作，并且它必须是logical_block_size的倍数。这通常由hypervisor正确配置。



#### 	1.20.2 (Filesystems)

```xml
...
<devices>
  <filesystem type='template'>
    <source name='my-vm-template'/>
    <target dir='/'/>
  </filesystem>
  <filesystem type='mount' accessmode='passthrough' multidevs='remap'>
    <driver type='path' wrpolicy='immediate'/>
    <source dir='/export/to/guest'/>
    <target dir='/import/from/host'/>
    <readonly/>
  </filesystem>
  <filesystem type='mount' accessmode='mapped' fmode='644' dmode='755'>
    <driver type='path'/>
    <source dir='/export/to/guest'/>
    <target dir='/import/from/host'/>
    <readonly/>
  </filesystem>
  <filesystem type='file' accessmode='passthrough'>
    <driver type='loop' format='raw'/>
    <source file='/export/to/guest.img'/>
    <target dir='/import/from/host'/>
    <readonly/>
  </filesystem>
  <filesystem type='mount' accessmode='passthrough'>
      <driver type='virtiofs' queue='1024'/>
      <binary path='/usr/libexec/virtiofsd' xattr='on'>
         <cache mode='always'/>
         <sandbox mode='namespace'/>
         <lock posix='on' flock='on'/>
         <thread_pool size='16'/>
      </binary>
      <source dir='/path'/>
      <target dir='mount_tag'/>
  </filesystem>
  <filesystem type='mount'>
      <driver type='virtiofs' queue='1024'/>
      <source socket='/tmp/sock'/>
      <target dir='tag'/>
  </filesystem>
  ...
</devices>
...
```

`filesystem`

​	filesystem属性`type`指定`source`的类型。可能的值包括：

​	`mount`

​		 在客户机中挂载的主机目录。用于LXC、OpenVZ(自0.6.2起)和QEMU/KVM(自0.8.5起)。如果未指定类型，则默认为此类型。此模式还具有一个可选的子元素`driver`，带有属性`type='path'`或`type='handle'`(自0.9.7起)。driver块具有可选的`wrpolicy`属性，用于进一步控制与主机页面缓存的交互；省略属性会给出默认行为，而值`immediate`表示在客户机文件写操作期间触发所有受影响页面的主机回写(自0.9.10起)。自6.2.0起，也支持`type='virtiofs'`。使用virtiofs需要设置共享内存，参见指南：[Virtiofs](https://libvirt.org/kbase/virtiofs.html)

​	`template`

​		OpenVZ文件系统模板。仅由OpenVZ驱动程序使用。

​	`file`

​		主机文件将被视为镜像并挂载到客户机中。将自动检测文件系统格式。仅由LXC驱动程序使用。

​	`block`

​		要在客户机中挂载的主机块设备。将自动检测文件系统格式。仅由LXC驱动程序使用(自0.9.5起)。

​	`ram`

​		使用来自主机操作系统的内存的内存文件系统。源元素具有一个名为usage的属性，该属性指定以KiB为单位的内存使用限制，除非通过units属性指定单位。仅由LXC驱动程序使用(自0.9.13起)。

​	`bind`

​		客户机内的一个目录将绑定到客户机内的另一个目录。仅由LXC驱动程序使用(自0.9.13起)。

​	`filesystem`

​		元素具有一个可选的`accessmode`属性，该属性指定访问源的安全模式(自0.8.5起)。目前，这仅在QEMU/KVM驱动程序的`type='mount'`模式下工作。对于驱动程序类型virtiofs，仅支持`passthrough`。对于其他驱动程序类型，可能的值包括：

​	`passthrough`

​		以客户机内的用户权限访问源。如果未指定，这是`accessmode`默认值。[More info](https://lists.gnu.org/archive/html/qemu-devel/2010-05/msg02673.html)

​	`mapped`

​		以hypervisor(QEMU进程)的权限访问`source`。[More info](https://lists.gnu.org/archive/html/qemu-devel/2010-05/msg02673.html)

​	`squash`

​		类似于'passthrough'，不同之处在于忽略特权操作(如'chown')的失败。这使得类似于passthrough的模式可供将hypervisor作为非root用户运行的人使用。[More info](https://lists.gnu.org/archive/html/qemu-devel/2010-09/msg00121.html)

自5.2.0版以来，filesystem元素具有一个可选的`model`属性，支持的值为"virtio-transitional"、"virtio-non-transitional"或"virtio"。有关更多详细信息，请参见1.20.5[Virtio transitional devices](https://libvirt.org/formatdomain.html#virtio-transitional-devices) 

filesystem元素具有可选的`fmode`和`dmode`属性。这两个属性控制在`accessmode`的`mapped`值(自6.10.0起，需要QEMU 2.10)下用于文件和目录的创建模式。如果未指定，QEMU将使用模式`600`创建文件，使用模式`700`创建目录。不支持setuid、setgid和粘性位。

filesystem元素具有一个可选的`multidevs`属性，用于指定如何处理包含多个设备的文件系统导出，以避免在使用9pfs时在客户机上出现文件ID冲突(自6.3.0起，需要QEMU 4.2)。此属性不适用于virtiofs。可能的值包括：

​	`default`

​		使用QEMU的默认设置(当前为警告)。

​	`remap`

​		此设置允许客户机访问一个导出中的多个设备，而不会发生行为不当。来自主机的inode编号会自动重新映射到客户机上，以主动防止如果客户机访问包含多个设备的导出时出现文件ID冲突。

​	`forbid`

​		只允许客户机访问导出中的一个设备。尝试访问同一导出中的其他设备将导致客户机上的个别文件系统访问失败，并在主机上记录错误(一次)。

​	`warn`

​		此设置类似于QEMU 4.2之前的9pfs行为，即不会执行任何操作来防止导出包含多个设备时的潜在文件ID冲突，唯一的例外是：现在在主机上记录了警告(一次)。如果一个导出中导出了多个设备，这可能会导致客户机一侧的不当行为，因为这可能会导致客户机一侧的文件ID冲突。

`driver `

​		可选的`driver`元素允许指定与提供文件系统的hypervisor驱动程序相关的进一步细节(自1.0.6起)。

​		如果hypervisor支持多个后端驱动程序，则`type`属性选择主要后端驱动程序名称，而format属性提供格式类型。例如，LXC支持"type"为"loop"，格式为"raw"或"nbd"(可以使用任何格式)。QEMU支持"type"为"path"或"handle"，但不支持格式。Virtuozzo驱动程序支持"type"为"ploop"，格式为"ploop"。

​		对于支持virtio的设备，还可以设置[Virtio-related options](https://libvirt.org/formatdomain.html#virtio-related-options)(自3.5.0起)。

​		对于`virtiofs`，可以使用`queue`属性指定队列大小(即队列可以容纳多少请求)(自6.2.0起)。

`binary`

​	可选的`binary`元素可以调整virtiofsd的选项。以下所有属性和元素都是可选的。属性`path`可用于覆盖守护程序的路径。属性`xattr`启用了文件系统扩展属性的使用。缓存可以通过`cache`元素进行调整，可能的模式值为`none`和`always`。锁定可以通过`lock`元素进行控制 - 属性`posix`和`flock`都接受`on`或`off`的值(自6.2.0起)。virtiofsd使用的沙箱方法可以通过`sandbox`元素进行配置，可能的模式值为`namespace`和`chroot`，请参阅 [virtiofsd documentation](https://qemu.readthedocs.io/en/latest/tools/virtiofsd.html) 以获取更多详细信息(自7.2.0起)。元素`thread_pool`接受一个名为`size`的属性，该属性定义了最大线程池大小。值"0"禁用了线程池。线程池有助于在与具有较高延迟的存储一起使用时增加在运行中的请求数量。但是，它会增加开销，因此对于快速、低延迟的文件系统，最好关闭它(自8.5.0起)。

`source` 

​	在主机上正在访问的客户机中的资源。必须在`type='template'`的情况下使用`name`属性，并且在`type='mount'`的情况下使用`dir`属性。对于virtiofs，可以使用`socket`属性连接到在libvirt之外启动的virtiofsd守护程序。在这种情况下，`target`元素不适用，大多数与virtiofs相关的选项也不适用，因为它们由virtiofsd而不是libvirtd控制。使用`type='ram'`时，使用`usage`属性设置以KiB为单位的内存限制，除非通过`units`属性指定单位。

`target `

​	在客户机中可以访问`source`的位置。对于大多数驱动程序，这是一个自动挂载点，但对于QEMU/KVM来说，这只是一个导出到客户机的任意字符串标记，用作提示在客户机上挂载的位置。

`readonly`

​		 启用将文件系统导出为只读挂载，如果未指定，默认为读写访问(目前仅适用于QEMU/KVM驱动程序)。

`space_hard_limit` 

​		该客户机文件系统的最大可用空间(自0.9.13起)。

`space_soft_limit`

​	 该客户机文件系统的最大可用空间。容器被允许在柔性限制期间超出其限制一段时间。之后，强制执行硬性限制(自0.9.13起)。



#### 	1.20.3 (Device Addresses)

许多设备都有一个可选的`<address>`子元素来描述设备在呈现给客户的虚拟总线上的位置。如果在输入时省略了一个地址(或地址中的任何可选属性)，libvirt将生成一个适当的地址；但是如果需要对布局进行更多控制，则需要显式地址。有关包括地址元素的设备示例，请参见下文。

​	每个地址都有一个强制性的属性`type`，用于描述设备所在的总线。为给定设备选择哪个地址在一定程度上受到设备和guest体系结构的限制。例如，<disk>设备使用`type='drive'`，而<console>设备在i686或x86_64guest上使用`type='ci'`，或在PowerPC64 pseriesguest上使用`type='papr-vio'`。每种地址类型都有其他可选属性，用于控制设备在总线上的位置：

​	`pci`

​		PCI地址具有以下附加属性：`domain`(2字节的十六进制整数，qemu当前未使用)、`bus`(0到0xff之间的十六进制值，包括0和0xff)、`slot`插槽(0x0到0x1f之间的十六进位值，包括0x0和0x1f)和`function`(0到7之间的值，包括7)。`multifunction`属性也可用，它控制开启PCI控制寄存器中特定插槽/功能的多功能位(自0.9.7，需要QEMU 0.13)。multifunction默认为“off”，但对于将使用多个功能的插槽的功能0，应设置为“打开”。(自4.10.0起)，支持PCI地址扩展，具体取决于体系结构。例如，S390guest的PCI地址将有一个zpci子元素，具有两个属性：`uid`(介于0x0001和0xffff之间的十六进制值，包括)和`fid`(介于0x00000000和0xffffff之间，包括)，供S390上的PCI设备用于用户定义标识符和功能标识符。自1.3.5，一些hypervisor驱动程序可能会接受一个没有其他属性的`<address type＝'pci'/>`元素作为为设备分配pci地址的显式请求，而不是可能适用于同一设备的其他类型的地址(例如virtio mmio)。域XML中配置的PCI地址与guest OS看到的PCI地址之间的关系有时可能会令人困惑：单独的文档更详细地描述了[how PCI addresses work](https://libvirt.org/pci-addresses.html)。

​	`drive`

​		驱动器地址具有 `controller`(控制器编号)、`bus`(总线编号)、`target`(目标编号) 和 `unit`(单元编号)等属性。这些属性用于指定虚拟机中的驱动器地址。

​	`virtio-serial`

​		Virtio-Serial 设备地址具有 `controller`(控制器编号)、`bus`(总线编号)和 `slot`(槽编号)等属性。这用于虚拟机中的 Virtio-Serial 设备。

​	`ccid`(智能卡)

​		CCID 设备地址用于智能卡设备，具有 `bus`(总线编号)和 `slot`(槽编号)等属性。

​	`usb`

​		USB 设备地址具有 `bus`(总线编号)和 `port`(端口编号)等属性，用于表示虚拟机中的 USB 设备。

​	`spapr-vio`

​		对于 PowerPC pseries 虚拟机，设备可以分配到 SPAPR-VIO 总线。它具有一个平面的 32 位地址空间，通常按照 0x00001000 的非零倍数分配地址，但也可以使用其他地址。每个地址具有 `reg`(起始寄存器地址的十六进制值)属性。

​	`ccw`

​		S390 虚拟机(使用 s390-ccw-virtio 机器类型)使用本机 CCW 总线进行 I/O 设备。CCW 总线地址具有 `cssid`(十六进制值，范围在 0 到 0xfe 之间)、`ssid`(值在 0 到 3 之间)和 `devno`(十六进制值，范围在 0 到 0xffff 之间)等属性。

​	`virtio-mmio`

​		这将设备放置在 virtio-MMIO 传输上，目前仅适用于某些 armv7l 和 aarch64 虚拟机。Virtio-MMIO 地址没有任何附加属性。

​	`isa`

​		ISA 地址具有 `iobase` 和 `irq` 等属性。

​	`unassigned`

​		对于 PCI 主机设备，`<address type='unassigned'/>` 允许管理员在域 XML 定义中包含 PCI 主机设备，而无需让虚拟机访问它。这允许Libvirt将设备作为常规PCI hostdev进行管理的配置，而不管guest是否有权访问它。`<address type＝'unasigned'/>`对于所有其他设备类型都是无效的地址类型。自6.0.0



#### 	1.20.4 (Virtio-related options)

QEMU的virtio设备在`driver`元素下有一些与virtio传输相关的属性：`iommu`属性允许设备使用模拟iommu。属性`ats`控制PCIe设备的地址转换服务支持。这是利用IOTLB支持所必需的(请参阅1.20.27 [IOMMU devices](https://libvirt.org/formatdomain.html#iommu-devices))。可能的值为`on`或`off`。自3.5.0起

属性`packed`控制QEMU是否应该尝试使用压缩virtqueues。与常规的拆分(spilt)队列相比，压缩(packed)队列仅由一个描述符环组成，用于替换可用和已用的环、索引和描述符缓冲区。这样可以提高缓存利用率和性能。是否实际使用压缩的virtqueues取决于QEMU、vhost后端和guest驱动程序之间的feature negotiation。可能的值为`on`或`off`。自6.3.0(仅限QEMU和KVM)

此可选属性`page_per_vq`控制向guest公开的通知功能的布局。启用后，每个virtio队列将在设备BAR上有一个专用页面，向guest公开。建议在hypervisor上启用vDPA时使用它，因为它可以将通知区域映射到物理设备，这仅在页面粒度中受支持。默认值由QEMU确定。7.9.0起(QEMU 2.8)注意：一般来说，你应该不考虑这个选项，除非你非常确定你知道自己在做什么。



#### 	1.20.5 (Virtio transitional devices)

从5.2.0开始，QEMU的一些virtio设备在与PCI/PCIe机器类型一起使用时，接受以下`mode`值：

​	`virtio-transitional`

​		这个模型选项允许设备同时与virtio 0.9和virtio 1.0的客户端驱动程序一起工作。因此，它是在需要与较旧的guest OS兼容性的情况下的最佳选择。Libvirt会将设备连接到传统的PCI插槽。

​	`virtio-non-transitional`

​		这个模型选项只能与virtio 1.0的客户端驱动程序一起工作，除非需要与较旧的guest OS兼容性，否则建议使用此选项。根据机器类型，Libvirt会将设备连接到PCI Express插槽或传统的PCI插槽，从而实现更优化的PCI拓扑结构。

​	`virtio`

​		这个模型选项在连接到PCI Express插槽时的行为类似于virtio-non-transitional设备，在连接到其他插槽时的行为类似于virtio-transitional设备。Libvirt会根据机器类型选择其中一个模型。这是在需要与早于5.2.0版本的Libvirt兼容性的情况下的最佳选择，但一般情况下不建议使用它。

需要注意的是，虽然大多数virtio设备都适用于上述模型选项，但也有一些例外情况：

- 对于SCSI控制器，由于历史原因，没有`virtio`模型可用。相反，应该使用`virtio-scsi`，它对于其他设备与virtio的行为相同。`virtio-transitional`和`virtio-non-transitional`都适用于SCSI控制器。
- 一些设备，如GPU和输入设备(键盘、平板电脑和鼠标)，仅在virtio 1.0规范中定义，因此没有过渡变体。唯一接受的模型是`virtio`，这将导致非过渡设备。

更多信息查阅：[qemu patch posting](https://lists.gnu.org/archive/html/qemu-devel/2018-12/msg00923.html)  和[virtio-1.0 spec](https://docs.oasis-open.org/virtio/virtio/v1.0/virtio-v1.0.html).



#### 	1.20.6 (Controllers)

根据guest体系结构的不同，一些设备总线可能会出现不止一次，其中一组虚拟设备连接到一个虚拟控制器。通常，libvirt可以在不需要显式XML标记的情况下自动推断此类控制器，但有时有必要提供显式控制器元素，尤其是在为预期设备热插拔的guest规划 [PCI topology](https://libvirt.org/pci-hotplug.html) 。

```xml
...
<devices>
  <controller type='ide' index='0'/>
  <controller type='virtio-serial' index='0' ports='16' vectors='4'/>
  <controller type='virtio-serial' index='1'>
    <address type='pci' domain='0x0000' bus='0x00' slot='0x0a' function='0x0'/>
  </controller>
  <controller type='scsi' index='0' model='virtio-scsi'>
    <driver iothread='4'/>
    <address type='pci' domain='0x0000' bus='0x00' slot='0x0b' function='0x0'/>
  </controller>
  <controller type='xenbus' maxGrantFrames='64' maxEventChannels='2047'/>
  ...
</devices>
...
```

每个控制器都有一个必需的`type`属性，该属性必须是以下之一：'ide'、'fdc'、'scsi'、'sata'、'usb'、'ccid'、'virtio-serial' 或 'pci'，以及一个必需的`index`属性，该属性是描述总线控制器在哪个顺序中遇到的十进制整数(用于`<address>`元素的`controller`属性)。自1.3.5以来，索引是可选的；如果未指定，将自动分配为给定控制器类型的未使用的最低索引。某些控制器类型具有控制特定功能的附加属性，例如：

​	`virtio-serial`

​		virtio-serial控制器具有两个附加的可选属性ports和vectors，用于控制可以通过该控制器连接多少设备。自5.2.0起，它支持一个可选的属性模型，可以是'virtio'、'virtio-transitional' 或 'virtio-non-transitional'。有关更多详细信息，请参阅 1.20.5 [Virtio transitional devices](https://libvirt.org/formatdomain.html#virtio-transitional-devices)。

​	`scsi`

​		`scsi`控制器具有一个可选的属性`model`，它是以下之一：'auto'、'buslogic'、'ibmvscsi'、'lsilogic'、'lsisas1068'、'lsisas1078'、'virtio-scsi'、'vmpvscsi'、'virtio-transitional'、'virtio-non-transitional'、'ncr53c90'(仅作为内置隐式控制器)、'am53c974'、'dc390'。有关更多详细信息，请参阅1.20.5 [Virtio transitional devices](https://libvirt.org/formatdomain.html#virtio-transitional-devices)。

​	`usb'auto'、'buslogic'、'ibmvscsi'、'lsilogic'、'lsisas1068'、'lsisas1078'、'virtio `

​		`usb`控制器具有一个可选的属性模型，它是以下之一："piix3-uhci"、"piix4-uhci"、"ehci"、"ich9-ehci1"、"ich9-uhci1"、"ich9-uhci2"、"ich9-uhci3"、"vt82c686b-uhci"、"pci-ohci"、"nec-xhci"、"qusb1"(xen pvusb，带有qemu后端，版本1.1) 、"qusb2"(xen pvusb，带有qemu后端，版本2.0)或 "qemu-xhci"。此外，自0.10.0起，如果需要明确禁用客户机的USB总线，可以使用`model='none'`。自1.0.5起，s390上不会构建默认的USB控制器。自1.3.5起，USB控制器接受一个`ports`属性，用于配置可以连接到控制器的设备数量。

​	`ide`

​		自3.10.0起，对于vbox驱动程序，IDE控制器具有一个可选的属性`model`，它是 "piix3"、"piix4" 或 "ich6" 中的一个。

​	`xenbus`

​		自5.2.0起，`xenbus`控制器具有一个可选的属性`maxGrantFrames`，该属性指定控制器为连接的设备提供的最大授权帧数量。自6.3.0起，xenbus控制器支持可选的`maxEventChannels`属性，该属性指定客户机可以使用的事件通道(PV中断)的最大数量。

注意：PowerPC64“spapr vio”地址没有关联的控制器。

对于本身是PCI或USB总线上设备的控制器，可选子元素`<address>`可以指定控制器与其主总线的确切关系，其语义在设备地址部分中描述。

可选的子元素`driver`可以指定特定于驱动程序的选项：

​	`queues`

​		 可选的`queues`属性指定控制器的队列数量。为了获得最佳性能，建议指定与虚拟CPU数量匹配的值。自1.0.5起(仅适用于QEMU和KVM)。

​	`cmd_per_lun`

​		 可选的`cmd_per_lun`属性指定主机控制的设备上可以排队的最大命令数。自1.2.7起(仅适用于QEMU和KVM)。

​	`max_sectors`

​		 可选的`max_sectors`属性指定在单个命令中传输到设备或从设备传输的最大数据量(以字节为单位)。传输长度以扇区为单位，其中一个扇区为512字节。自1.2.7起(仅适用于QEMU和KVM)。

​	`ioeventfd`

​		 可选的`ioeventfd`属性指定控制器是否应使用I/O异步处理。接受的值为"on"和"off"。自1.2.18起。

​	`iothread`

​		 对于控制器类型为`scsi`且使用模型`virtio-scsi`的地址类型为`pci`和`ccw`的情况，自1.3.5版本(QEMU 2.4)以来支持。可选的iothread属性将控制器分配给由域`iothreads`(请参阅 1.5 [IOThreads Allocation](https://libvirt.org/formatdomain.html#iothreads-allocation))定义的IOThread范围。分配为使用指定控制器的每个SCSI磁盘都将利用相同的IOThread。如果希望为特定的SCSI磁盘指定特定的IOThread，则必须定义多个控制器，每个控制器都具有特定的`iothread`值。`iothread`值必须在1到域iothreads值的范围内。

​	virtio options

​		对于virtio控制器，可以设置1.20.4 [Virtio-related options](https://libvirt.org/formatdomain.html#virtio-related-options)。自3.5.0起

USB配套控制器有一个可选的子元素`<master>`，用于指定配套控制器与其主控制器的确切关系。伴随控制器与其主控制器在同一总线上，因此伴随`index`值应该相等。并不是所有的控制器模型都可以用作配套控制器，libvirt可能会为一些特定的模型提供一些合理的默认值(`master startport`主启动端口的设置和地址的`function`)。首选的配套控制器是`ich uhci[123]`。

```xml
...
<devices>
  <controller type='usb' index='0' model='ich9-ehci1'>
    <address type='pci' domain='0' bus='0' slot='4' function='7'/>
  </controller>
  <controller type='usb' index='0' model='ich9-uhci1'>
    <master startport='0'/>
    <address type='pci' domain='0' bus='0' slot='4' function='0' multifunction='on'/>
  </controller>
  ...
</devices>
...
```

PCI控制器具有可选的`model`属性；该属性的可能值为

- pci-root， pci-bridge ( **since 1.0.5** )
- pcie-root， dmi-to-pci-bridge ( **since 1.1.2** )
- pcie-root-port， pcie-switch-upstream-port， pcie-switch-downstream-port ( **since 1.2.19** )
- pci-expander-bus， pcie-expander-bus ( **since 1.3.4** )
- pcie-to-pci-bridge ( **since 4.3.0** )

根控制器(`pci-root`和`pcie-root`)有一个可选的`pcihole64`元素，用于指定64位pci漏洞的大小(以千字节为单位，或以`pcihole64`的`unit`属性指定的单位为单位)。当QEMU和Seabios足够新，可以支持64位PCI holes时，一些guest(如WindowsXP或WindowsServer2003)可能会崩溃，除非禁用(设置为0)。自1.1.2起(仅限QEMU)

PCI控制器还有一个可选的子元素`<model>`，该子元素具有属性`name`。`name`属性包含qemu正在模拟的特定设备的名称(例如“i82801b11桥”)，而不是简单的设备类别(“pcie到pci桥”、“pci桥”)(在控制器元素的模型属性中设置)。在几乎所有情况下，都不应该手动向控制器添加<model>子元素，也不应该修改libvirt自动生成的子元素。自1.2.19起(仅限QEMU)。

PCI控制器还有一个可选的子元素<target>，其属性和子元素如下所示。这些是可配置项，1)对客户操作系统可见，因此必须保留以与客户ABI兼容，2)通常保留为默认值或由libvirt自动派生。在几乎所有情况下，都不应该手动将<target>子元素添加到控制器中，也不应该修改libvirt自动生成的值中的值。自1.2.19起(仅限QEMU)。

​	`chassisNr `

​		对于具有属性model="pci-bridge"的PCI控制器，还可以在<target>子元素中具有`chassisNr`属性，该属性用于控制QEMU的"chassis_nr"选项，用于pci-bridge设备(通常情况下，libvirt会自动将其设置为与pci控制器的index属性相同的值)。如果设置了`chassisNr`，必须在1到255之间。

​	`chassis `

​		pcie-root-port和pcie-switch-downstream-port控制器也可以在<target>子元素中具有`chassis`属性，该属性用于设置控制器的"chassis"配置值，该值对虚拟机可见。如果设置了`chassis`，必须在0到255之间。

​	`port `

​		pcie-root-port和pcie-switch-downstream-port控制器还可以在<target>子元素中具有port属性，该属性用于设置控制器的"port"配置值，该值对虚拟机可见。如果设置了port，必须在0到255之间。

​	`hotplug `

​		pci-root(自7.9.0)、pcie-root-port(自6.3.0)和pcie-switch-downstream-port控制器(自6.3.0)也可以在<target>子元素中具有hotplug属性，该属性用于禁用特定控制器上设备的热插拔。对于pci-root控制器，该设置会影响基于ACPI的热插拔。对于其余控制器，该设置既影响基于ACPI的热插拔，也影响PCIE本地热插拔。hotplug的默认设置为on；要禁用特定控制器上设备的热插拔，应将其设置为off。

​	`busNr `

​		pci-expander-bus和pcie-expander-bus控制器可以具有可选的`busNr`属性(1-254)。这将是新总线的总线号；在指定的总线号和255之间的所有总线号只能分配给插入到从此扩展总线开始的层次结构的PCI/PCIe控制器，而小于指定值的总线号将分配给下一个较低的扩展总线(如果没有较低的扩展总线，则为根总线)。如果不指定busNumber，libvirt将查找所有其他扩展总线中的最低总线号(如果没有其他总线，则使用256)，并自动分配找到的总线的busNr值 - 2，这为pci-expander-bus提供了一个总线号，以及为其自动附加的pci-bridge提供了一个总线号(如果 计划添加更多的pci-bridge到总线的层次结构中， 应该手动将busNr设置为较低的值)。

​		用于自动确定pcie-expander-bus的busNr属性的算法类似，但由于pcie-expander-bus没有任何内置的pci-bridge，因此第二个总线号只是为必须连接到总线中以实际插入端点设备的pcie-root-port保留的总线号。如果打算将多个设备插入pcie-expander-bus中，必须将pcie-switch-upstream-port连接到插入到pcie-expander-bus中的pcie-root-port，并将多个pcie-switch-downstream-port连接到pcie-switch-upstream-port，当然，为了使这正常工作， 需要相应地减小pcie-expander-bus的busNr，以便在其上方有足够的未使用的总线号以容纳为上游端口提供一个总线号以及为每个下游端口提供一个总线号(除了pcie-root-port和pcie-expander-bus本身)。

​	`node`

​		一些PCI控制器(用于pc机器类型的`pci-expander-bus`，用于q35机器类型的`pci-expander-bus`，以及自3.6.0以来用于pseries机器类型的PCI根)可以在<target>子元素中具有可选的<node>子元素，用于设置向该总线的guest OS报告的NUMA节点-然后guest OS将知道该总线上的所有设备都是指定NUMA节点的一部分(在将主机设备分配给域时，由libvirt API的用户将主机设备连接到正确的pci-expander-总线)。

​	`index`

​		pSeries客户的pci根控制器使用此属性来记录它们在客户机中的显示顺序。自3.6.0起

​	对于提供隐式PCI总线的机器类型，索引为0的PCI根控制器是自动添加的，并且需要使用PCI设备。pci根没有地址。如果PCI根提供的一条总线上有太多设备无法容纳，或者指定了大于零的PCI总线号，则会自动添加PCI桥。PCI桥也可以手动指定，但它们的地址应该仅指由已经指定的PCI控制器提供的PCI总线。在PCI控制器索引中留下间隙可能会导致配置无效。

```xml
...
<devices>
  <controller type='pci' index='0' model='pci-root'/>
  <controller type='pci' index='1' model='pci-bridge'>
    <address type='pci' domain='0' bus='0' slot='5' function='0' multifunction='off'/>
  </controller>
</devices>
...
```

​	对于提供隐式PCI Express(PCIe)总线的机器类型(例如，基于Q35芯片组的机器类型)，会自动将具有index=0的pcie-root控制器添加到域的配置中。pcie-root也没有地址，提供31个插槽(编号为1-31)，可用于连接PCIe或PCI设备(尽管libvirt不会自动将PCI设备分配给PCIe插槽，但允许手动指定此类分配)。连接到pcie-root的设备无法热插拔。如果在客户配置中存在传统PCI设备，将自动添加一个`pcie-to-pci-bridge`控制器：该控制器插入到pcie-root-port中，提供31个可用的PCI插槽(1-31)，支持热插拔(自4.3.0以来)。如果QEMU二进制文件不支持相应的设备，则将添加一个`dmi-to-pci-bridge`控制器，通常位于slot=0x1e的事实标准位置。dmi-to-pci-bridge控制器插入到PCIe插槽中(由pcie-root提供)，并自身提供31个标准PCI插槽(也不支持设备热插拔)。为了在客户机系统中具有可热插拔的PCI插槽，还将自动创建并连接到自动创建的dmi-to-pci-bridge控制器的插槽之一的pci-bridge控制器；libvirt自动确定的所有客户PCI设备的地址都将放在此pci-bridge设备上(自1.1.2以来)。

​	具有隐式pcie-root的域还可以添加具有`model='pcie-root-port'、model='pcie-switch-upstream-port'`和`model='pcie-switch-downstream-port'`的控制器。pcie-root-port是一种简单类型的桥接设备，只能在其上游侧连接到pcie-root总线的31个插槽之一，并在下游侧提供一个(PCIe、可热插拔)端口(在slot='0'处)。pcie-root-port可用于提供一个插槽以后热插拔PCIe设备(但本身不可热插拔-必须在启动域时包含在配置中)(自1.2.19以来)。

​	pcie-switch-upstream-port是一种更灵活(但也更复杂)的设备，只能在其上游侧插入到pcie-root-port或pcie-switch-downstream-port中(并且只能在启动域之前插入-不可热插拔)，并在下游侧提供32个端口(slot='0' - slot='31')，只接受pcie-switch-downstream-port设备；每个pcie-switch-downstream-port设备只能在其上游侧插入到pcie-switch-upstream-port中(同样不可热插拔)，在其下游侧提供一个可热插拔的pcie端口，可以接受任何标准的PCI或PCIe设备(或另一个pcie-switch-upstream-port)，即与pcie-root-port的功能相同(自1.2.19以来)。

```xml
...
<devices>
  <controller type='pci' index='0' model='pcie-root'/>
  <controller type='pci' index='1' model='pcie-root-port'>
    <address type='pci' domain='0x0000' bus='0x00' slot='0x01' function='0x0'/>
  </controller>
  <controller type='pci' index='2' model='pcie-to-pci-bridge'>
    <address type='pci' domain='0x0000' bus='0x01' slot='0x00' function='0x0'/>
  </controller>
</devices>
...
```



#### 	1.20.7 (Device leases)

当使用锁管理器时，可能需要记录针对VM的设备租赁。锁管理器将确保VM不会启动，除非可以获得租约。

```xml
...
<devices>
  ...
  <lease>
    <lockspace>somearea</lockspace>
    <key>somekey</key>
    <target path='/some/lease/path' offset='1024'/>
  </lease>
  ...
</devices>
...
```

​	`lockspace`

​		这是一个任意的字符串，用于标识密钥所在的锁空间。锁管理器可能会对锁空间名称的格式或长度施加额外的限制。

​	`key`

​		这是一个任意字符串，唯一标识要获取的租约。锁管理员可能会对密钥的格式或长度施加额外的限制。

​	`target`

​		这是与锁空间关联的文件的完全限定路径。偏移量指定租约在文件中的存储位置。如果锁管理器不需要偏移量，只需传递0即可。



#### 	1.20.8 (Host device assignment)

##### 			1.20.8.1 USB / PCI / SCSI devices

连接到主机的USB、PCI和SCSI设备可以使用`hostdev`元素传递给客户。在USB 0.4.4、PCI 0.6.0(仅限KVM)和SCSI 1.0.6(仅限KVM)之后：

```xml
...
<devices>
  <hostdev mode='subsystem' type='usb'>
    <source startupPolicy='optional' guestReset='off'>
      <vendor id='0x1234'/>
      <product id='0xbeef'/>
    </source>
    <boot order='2'/>
  </hostdev>
</devices>
...
```

```xml
...
<devices>
  <hostdev mode='subsystem' type='pci' managed='yes'>
    <source writeFiltering='no'>
      <address domain='0x0000' bus='0x06' slot='0x02' function='0x0'/>
    </source>
    <boot order='1'/>
    <rom bar='on' file='/etc/fake/boot.bin'/>
  </hostdev>
</devices>
...
```

```xml
...
<devices>
  <hostdev mode='subsystem' type='scsi'>
    <source protocol='iscsi' name='iqn.2014-08.com.example:iscsi-nopool/1'>
      <host name='example.com' port='3260'/>
      <auth username='myuser'>
        <secret type='iscsi' usage='libvirtiscsi'/>
      </auth>
      <initiator>
        <iqn name='iqn.2020-07.com.example:test'/>
      </initiator>
    </source>
    <address type='drive' controller='0' bus='0' target='0' unit='0'/>
  </hostdev>
</devices>
...
```

```xml
...
<devices>
  <hostdev mode='subsystem' type='mdev' model='vfio-pci'>
  <source>
    <address uuid='c2177883-f1bb-47f0-914d-32a22e3a8804'/>
  </source>
  </hostdev>
  <hostdev mode='subsystem' type='mdev' model='vfio-ccw'>
  <source>
    <address uuid='9063cba3-ecef-47b6-abcf-3fef4fdcad85'/>
  </source>
  <address type='ccw' cssid='0xfe' ssid='0x0' devno='0x0001'/>
  </hostdev>
</devices>
...
```

`hostdev`

​	`hostdev`元素是用于描述主机设备的主要容器。对于每个设备，`model`始终为“subsystem”，`type`为以下值之一，并注明其他属性。

​	`usb`

​		USB设备在虚拟机启动时会从主机中分离，而在虚拟机退出或设备热拔插时会重新连接到主机。

​	`pci`

​		 对于PCI设备，当`managed`属性设置为“yes”时，设备在传递给虚拟机之前会从主机中分离，并在虚拟机退出后重新连接到主机。如果`managed`被省略或设置为“no”，则用户需要使用适当的virsh命令来分离和重新连接设备。

​	`scsi`

​		对于SCSI设备，用户需要确保主机不在使用该设备。可选的`sgio`属性(自1.0.6起)指示是否对磁盘过滤未授权的SG_IO命令。有效设置为“filtered”或“unfiltered”，默认为“filtered”。可选的`rawio`属性(自1.2.9起)指示LUN是否需要rawio功能，有效设置为“yes”或“no”。请参阅1.20.1 [Hard drives， floppy disks， CDROMs](https://libvirt.org/formatdomain.html#hard-drives-floppy-disks-cdroms) 部分中的rawio说明。如果域中的磁盘lun已经具有rawio功能，则不需要此设置。

​	`scsi_host`

​		(自2.5.0起): 对于SCSI设备，用户需要确保主机不在使用该设备。这种类型将由单个HBA呈现的所有LUN传递给虚拟机。从版本5.2.0开始，可以将`model`属性指定为“virtio-transitional”，“virtio-non-transitional”或“virtio”以用于Virtio过渡设备。

​	`mdev`

​		(自3.2.0起): 对于中介设备，`model`属性指定了确定主机的vfio驱动程序如何将设备暴露给虚拟机的设备API。支持的值包括“vfio-pci”，“vfio-ccw”(自4.4.0起)和“vfio-ap”(自4.9.0起)。`display`属性(自4.6.0起)可用于启用或禁用由中介设备支持的加速远程桌面，例如NVIDIA vGPU或Intel GVT-g，其值可以为“on”或“off”(默认为“off”)。从版本5.10.0开始，对于`model='vfio-pci'`的设备，还可以使用可选的`ramfb`属性，以为虚拟机提供内存帧缓冲设备。

​	注意：`managed`属性仅与`type='pci'`一起使用，并且被所有其他设备类型忽略，因此使用pci设备以外的设备显式设置`managed`与省略它具有相同的效果。同样，`model`属性仅由中介设备支持，而被所有其他设备类型忽略。

`source`

​	`source`元素使用以下机制描述从主机看到的设备：

​		`usb`

​			USB设备可以通过使用供应商和产品元素的`vendor`/`product`id来寻址，也可以通过使用`address`元素的设备在主机上的`address`来寻址。

​			从1.0.0开始，USB设备的`source`元素可能包含`startupPolicy`属性，该属性可用于定义策略，如果找不到指定的主机USB设备，该策略将执行什么操作。该属性接受以下值：

![image-20230928103320718](D:\File Storage\TyporaStorage\image-20230928103320718.png)

​			从8.6.0起，`source`元素可以包含具有以下值的`guestReset`属性：

![image-20230928103528553](D:\File Storage\TyporaStorage\image-20230928103528553.png)

​		当为USB设备分配重置时崩溃的固件时，此属性可能很有用。

​	`pci `

​		PCI设备仅可以通过它们的地址来描述。自6.8.0(仅适用于Xen)起，PCI设备的源元素可以包含`writeFiltering`属性，用于控制对PCI配置空间的写访问。默认情况下，Xen仅允许对配置空间的已知安全值进行写操作。设置`writeFiltering='no'`将允许对设备的PCI配置空间进行所有写操作。

​	`scsi `

​		SCSI设备由适配器和地址元素共同描述。地址元素包括一个`bus`属性(2位数的总线编号)，一个`target`属性(10位数的目标编号)和一个`unit`属性(总线上的20位数的单元编号)。并非所有的虚拟化平台都支持更大的目标和单元值。每个虚拟化平台都会确定适配器支持的最大值。

​		自1.2.8起，SCSI设备的源元素可以包含`protocol`属性。当该属性设置为“iscsi”时，主机设备XML遵循网络磁盘设备(请参阅 1.20.1 [Hard drives, floppy disks, CDROMs](https://libvirt.org/formatdomain.html#hard-drives-floppy-disks-cdroms))并使用相同的`name`属性，可选择使用`auth`元素提供iSCSI服务器的身份验证凭据。

​		自6.7.0起，可选的`initiator`子元素控制通过虚拟化平台运行的起始器的IQN，通过其 `<iqn name='iqn...'` 子元素进行配置。

​	`scsi_host `

​		自2.5.0起，单个SCSI HBA后面的多个LUN通过`protocol`属性设置为“vhost”以及一个`wwpn`属性来描述，该`wwpn`属性是在主机的configfs中建立的vhost_scsi wwpn(16位十六进制数字，带有"naa."前缀)。

​	`mdev `

​		中介设备(Mediated devices)(自3.2.0起)通过地址元素来描述。地址元素包含一个必填的`uuid`属性。

`vendor， product`

​	`vendor`和`product`元素都有一个`id`属性，用于指定USB vendor和产品id。id可以以十进制、十六进制(以0x开头)或八进制(以0开头)形式给出。

`boot `

​	指定设备可用于启动。`order`属性确定了设备在启动顺序中的尝试顺序。设备级别的`boot`元素不能与BIOS引导加载器部分中的通用`boot`元素一起使用。自0.8.8起支持PCI设备，自1.0.1起支持USB设备。

`rom `

​	`rom`元素用于更改PCI设备的ROM在宿主中呈现给客户机的方式。可选的`bar`属性可以设置为"on"或"off"，并确定设备的ROM是否会出现在客户机的内存映射中(在PCI文档中，“rombar”设置控制ROM的基地址寄存器是否存在)。如果未指定rom bar，则将使用qemu的默认设置(较旧版本的qemu使用了默认的"off"，而较新版本的qemu使用了默认的"on")。自0.9.7起(仅适用于QEMU和KVM)。可选的`file`属性包含一个绝对路径，指向要呈现给客户机作为设备ROM BIOS的二进制文件。这可以用于为支持SR-IOV的以太网设备的虚拟功能提供PXE引导ROM(对于VF而言，这些设备没有VF的引导ROM)。自0.9.10起(仅适用于QEMU和KVM)。可选的`enabled`属性可以设置为"no"以完全禁用设备的PCI ROM加载；如果通过此属性禁用了PCI ROM加载，则将拒绝尝试使用`bar`或`file`属性进一步调整加载过程。自4.3.0起(仅适用于QEMU和KVM)。

`address `

​	USB设备的`address`元素具有`bus`和`device`属性，用于指定设备在主机上出现的USB总线和设备编号。这些属性的值可以使用十进制、十六进制(以0x开头)或八进制(以0开头)形式表示。对于PCI设备，该元素携带了4个属性，允许将设备指定为可以在`lspci`或`virsh nodedev-list`中找到的设备。对于SCSI设备，必须使用'drive'地址类型。对于中介设备，这些是仅由软件定义的设备，定义了物理父设备上的资源分配，使用的地址类型必须符合`hostdev`元素的`model`属性，例如，对于`vfio-pci`设备API，必须使用除PCI以外的任何地址类型，对于`vfio-ccw`设备API，必须使用除CCW以外的任何地址类型，否则将引发错误。有关address元素的更多详细信息，请参阅 1.20.3 [Device Addresses](https://libvirt.org/formatdomain.html#device-addresses)

`driver `

​	PCI设备可以具有可选的`driver`子元素，用于指定用于PCI设备分配的后端驱动程序。使用name属性来选择"vfio"(用于新的VFIO设备分配后端，与UEFI SecureBoot兼容)或"kvm"(由KVM内核模块直接处理的传统设备分配)。自1.0.5起(仅适用于QEMU和KVM，需要3.6或更新的内核)。当指定时，如果主机上不支持所请求的设备分配方法，设备分配将失败。如果未指定，默认情况下，在具有可用且加载的VFIO驱动程序的系统上为"vfio"，在较旧的系统上或未加载VFIO驱动程序的系统上为"kvm"。自1.1.3起(在此之前，默认值始终为"kvm")。

`readonly `

​	表示设备是只读的，目前仅由SCSI主机设备支持。自1.0.6起(仅适用于QEMU和KVM)。

`shareable `

​	如果存在，表示预计设备将在域之间共享(假设hypervisor和操作系统支持此功能)。仅由SCSI主机设备支持。自1.0.6起。

注意：虽然`shareable`在1.0.6中引入，但直到1.2.2之前它并未按预期工作。

##### 			1.20.8.2 Block / character devices

可以使用`hostdev`元素将来自主机的块/字符设备传递给客户。这只能通过基于容器的虚拟化实现。设备由完全限定的路径指定。自LXC 1.0.1之后：

```xml
...
<hostdev mode='capabilities' type='storage'>
  <source>
    <block>/dev/sdf1</block>
  </source>
</hostdev>
...
```

​	`hostdev`

​		`hostdev`元素是用于描述主机设备的主要容器。对于块/字符设备，直通`mode`始终为“capabilities”，`type`为块设备的“storage”，字符设备的“misc”，主机网络接口的“net”。

​	`source`

​		`source`元素描述了从主机看到的设备。对于块设备，主机操作系统中块设备的路径在嵌套的“block”元素中提供，而对于字符设备，则使用“char”元素。对于网络接口，接口的名称在“interface”元素中提供。



#### 	1.20.9 (Redirected devices)

支持通过字符设备重定向USB设备，自0.9.5起(仅限KVM)：

```xml
...
<devices>
  <redirdev bus='usb' type='spicevmc'/>
  <redirdev bus='usb' type='tcp'>
    <source mode='connect' host='localhost' service='4000'/>
    <boot order='1'/>
  </redirdev>
  <redirfilter>
    <usbdev class='0x08' vendor='0x1234' product='0xbeef' version='2.56' allow='yes'/>
    <usbdev allow='no'/>
  </redirfilter>
</devices>
...
```

​	`redirdev`	

​		`redirdev`元素是用于描述重定向设备的主要容器。对于usb设备，`bus`必须是“usb”。需要一个额外的属性类型，与支持的串行设备类型之一相匹配(请参阅 1.20.16 [Consoles, serial, parallel & channel devices](https://libvirt.org/formatdomain.html#consoles-serial-parallel-channel-devices))，以描述隧道的主机侧；`type='tcp'`或`type='picevmc'`(使用SPICE图形设备的usbredir通道(参见1.20.14 [Graphical framebuffers](https://libvirt.org/formatdomain.html#graphical-framebuffers)))是典型的。

​		`redirdev`元素有一个可选的子元素`<address>`，它可以将设备连接到特定的控制器。根据给定的类型，可能需要更多的子元素，如`<source>`，尽管不需要`<target>`子元素(因为字符设备的使用者是系统hypervisor本身，而不是 guest中可见的设备)。

​	`boot`

指定设备是可引导的。order属性确定在引导序列中尝试设备的顺序。在BIOS引导加载程序部分，每个设备的引导元素不能与常规引导元素一起使用。(自1.0.1起)

​	`redirfilter`

​		`dirfilterelement`用于创建筛选规则，以从重定向中筛选出某些设备。它使用子元素`<usbdev>`来定义每个筛选规则。`class`属性是USB 经典代码，例如0x08表示大容量存储设备。USB设备可以通过供应商`vendor`/`product`产品id使用供应商和产品属性进行寻址。`version`是bcdedit设备字段中的设备版本(不是USB协议的版本)。这四个属性是可选的，-1可以用于允许它们的任何值。`allow`属性是必需的，“yes”表示允许，“no”表示拒绝。



#### 	1.20.10 (Smartcard devices)

虚拟智能卡设备可以通过`smartcard`元素提供给 guest。主机上的USB智能卡读卡器设备不能在具有简单设备直通的guest身上使用，因为它在主机上不可用，可能会在“移除”主机时锁定主机。因此，一些hypervisor提供了一种专门的虚拟设备，该设备可以向 guest呈现智能卡接口，具有多种模式，用于描述如何从主机甚至从创建给第三方智能卡提供商的通道获得凭据。自0.8.8

```xml
...
<devices>
  <smartcard mode='host'/>
  <smartcard mode='host-certificates'>
    <certificate>cert1</certificate>
    <certificate>cert2</certificate>
    <certificate>cert3</certificate>
    <database>/etc/pki/nssdb/</database>
  </smartcard>
  <smartcard mode='passthrough' type='tcp'>
    <source mode='bind' host='127.0.0.1' service='2001'/>
    <protocol type='raw'/>
    <address type='ccid' controller='0' slot='0'/>
  </smartcard>
  <smartcard mode='passthrough' type='spicevmc'/>
</devices>
...
```

`<smartcard>`元素具有强制属性`mode`。支持以下模式；在每种模式下， guest都会在其USB总线上看到一个设备，其行为类似于物理USB CCID(芯片/智能卡接口设备)卡。

​	`host`

​		最简单的操作，系统hypervisor将来自 guest的所有请求中继为通过NSS直接访问主机的智能卡。不需要其他属性或子元素。请参阅下面关于可选`<address>`子元素的使用。

​	`host-certificates`

​		不需要将智能卡插入主机，而是可以提供驻留在主机数据库中的三个NSS证书名称。这些证书可以通过命令`certutil-d/etc/pki/nssdb-x-t CT，CT，CT-S-S CN=cert1-n cert1`生成，并且生成的三个证书名称必须作为三个`<certificate>`子元素中的每一个的内容提供。一个额外的子元素`<database>`可以指定到备用目录的绝对路径(在创建证书时匹配certutil命令的-d选项)；如果不存在，则默认为/etc/pki/nssdb。

​	`passthrough`

​		与其让系统hypervisor直接与主机通信，不如通过次要字符设备将所有请求隧道传输到第三方提供商(第三方供应商可能反过来与智能卡通信或使用三个证书文件)。在这种操作模式下，需要一个额外的属性`type`，与支持的串行设备类型之一相匹配(请参阅1.20.16 [Consoles, serial，parallel & channel devices](https://libvirt.org/formatdomain.html#consoles-serial-parallel-channel-devices))，以描述隧道的主机侧；`type='tcp'`或`type='picevmc'`(使用SPICE图形设备的智能卡通道(参见1.20.16 [Graphical framebuffers](https://libvirt.org/formatdomain.html#graphical-framebuffers)))是典型的。根据给定的类型，可能需要更多的子元素，如`<source>`，尽管不需要`<target>`子元素(因为字符设备的使用者是系统hypervisor本身，而不是 guest中可见的设备)。

每个模式都支持一个可选的子元素`<address>`(请参阅1.20.3 [Device Addresses](https://libvirt.org/formatdomain.html#device-addresses))，它可以微调智能卡和ccid总线控制器之间的相关性。目前，qemu最多只支持一个智能卡，地址为bus=0 slot=0。



#### 	1.20.11 (Network interfaces)

```xml
...
<devices>
  <interface type='direct' trustGuestRxFilters='yes'>
    <source dev='eth0'/>
    <mac address='52:54:00:5d:c7:9e'/>
    <boot order='1'/>
    <rom bar='off'/>
    <acpi index='4'/>
  </interface>
</devices>
...
```

有多种方法可以指定对虚拟机可见的网络接口。以下各小节提供了有关常见设置选项的更多详细信息：

自1.2.10起，接口元素的`trustGuestRxFilters`属性允许主机通过将该属性设置为`yes`来检测和信任虚拟机关于接口MAC地址和接收过滤器更改的报告。出于安全原因，该属性的默认设置为`no`，其支持取决于虚拟机的网络设备模型以及主机上的连接类型 - 目前仅支持virtio设备模型和主机上的macvtap连接。

每个`<interface>`元素都可以包含一个可选的`<address>`子元素，该子元素可以将接口绑定到特定的PCI插槽，属性`type='pci'`的设置如[Device Addresses](https://libvirt.org/formatdomain.html#device-addresses) 部分所述。

自6.6.0起，可以通过在`<mac/>`元素中添加`type="static"`属性来强制libvirt在MAC地址位于保留的VMware范围内时保持提供的MAC地址不变。请注意，如果提供的MAC地址在保留的VMware范围之外，则此属性将无效。

自7.3.0起，可以为网络接口设置ACPI索引。对于某些操作系统(例如带有systemd的Linux)，ACPI索引用于提供网络接口设备命名，以便在分配给设备的PCI地址更改时保持稳定。该值必须在所有设备中是唯一的，并且必须介于1和(16*1024-1)之间。



##### 			1.20.11.1 (Virtual network)

**这是适用于具有动态/无线网络配置的主机或多主机环境的一般虚拟机连接的推荐配置。**如果主机硬件详细信息在`<network>`定义中单独描述(自0.9.4起)

提供了一个连接，其详细信息由命名的网络定义描述。根据虚拟网络的“转发模式”配置，网络可以是完全隔离的(未提供`<forward>`元素)，NAT到显式网络设备或默认路由(`<forward mode='nat'>`)，无NAT路由(`<forward mode='route'/>`)，或直接连接到主机的一个网络接口(通过macvtap)或桥接设备(`<forward mode='bridge|private|vepa|passthrough'/>` 自0.9.4起)。

对于具有桥接、private、vepa和passthrough转发模式的网络，假定主机已经在libvirt范围之外设置了必要的DNS和DHCP服务。对于隔离、nat和路由网络，libvirt提供了虚拟网络上的DHCP和DNS，IP范围可以通过查看`virsh net-dumpxml [networkname]`的虚拟网络配置来确定。默认情况下，系统中已经设置了一个名为'default'的虚拟网络，它会将流量进行NAT转发到默认路由，并具有IP范围为`192.168.122.0/255.255.255.0`。每个虚拟机将会有一个相关联的tun设备，其名称为vnetN，也可以通过`<target>`元素进行覆盖(请参阅1.20.11.17[Overriding the target element](https://libvirt.org/formatdomain.html#overriding-the-target-element))。

当接口的源是网络时，可以指定一个`portgroup`以及网络的名称；一个网络可以定义多个portgroup，每个portgroup包含了针对不同类型的网络连接的略微不同的配置信息(自0.9.4起)。

当虚拟机运行时，类型为`network`的接口可能包括一个`portid`属性。这提供了一个关联的virNetworkPortPtr对象的UUID，记录了域接口与网络之间的关联。由于端口对象在启动和关闭期间会自动创建和删除，所以该属性是只读的(自5.1.0起)。

此外，类似于`direct`网络连接(下文描述)，类型为`network`的连接可以指定一个`virtualport`元素，其中包含要转发到vepa(802.1Qbg)或802.1Qbh兼容交换机(自0.8.2起)，或者转发到Open vSwitch虚拟交换机(自0.9.11起)。

由于实际交换机类型可能根据主机上`<network>`中的配置而有所不同，因此可以省略virtualport `type`属性，并指定来自多个不同virtualport类型的属性(也可以省略某些属性)；在域启动时，将通过合并网络中定义的类型和属性以及接口引用的portgroup中的类型和属性来构建完整的`<virtualport>`元素。新构建的virtualport是它们的组合。较低virtualport的属性不能更改较高virtualport中定义的属性。接口具有最高优先级，端口组具有最低优先级(自0.10.0起)。例如，为了与802.1Qbh交换机和Open vSwitch交换机正常工作，可以选择不指定类型，但同时指定`profileid`(如果交换机是802.1Qbh)和`interfaceid`(如果交换机是Open vSwitch)(也可以省略其他属性，例如managerid、typeid或profileid，以从网络的`<virtualport>`填充)。如果要限制虚拟机仅连接到某些类型的交换机，可以指定virtualport类型，但仍然省略某些/所有参数 - 在这种情况下，如果主机的网络具有不同类型的virtualport，接口的连接将失败。

```xml
...
<devices>
  <interface type='network'>
    <source network='default'/>
  </interface>
  ...
  <interface type='network'>
    <source network='default' portgroup='engineering'/>
    <target dev='vnet7'/>
    <mac address="00:11:22:33:44:55"/>
    <virtualport>
      <parameters instanceid='09b11c53-8b5c-4eeb-8f00-d84eaa0aaa4f'/>
    </virtualport>
  </interface>
</devices>
...
```



##### 			1.20.11.2 (Bridge to LAN)

**这是针对在具有静态有线网络配置的主机上实现一般虚拟机连通性的推荐配置。**

它提供了一个桥接连接，将虚拟机直接连接到局域网(LAN)。这假定主机上有一个桥接设备，其中附加了一个或多个主机的物理网卡。虚拟机将具有一个相关联的tun设备，名称为vnetN，也可以使用<target>元素进行覆盖(请参阅1.20.11.17 [Overriding the target element](https://libvirt.org/formatdomain.html#overriding-the-target-element))。tun设备将连接到桥接设备上。IP范围/网络配置与LAN上使用的相同。这为虚拟机提供了与物理机一样的完整的入站和出站网络访问权限。

在Linux系统上，桥接设备通常是标准的Linux主机桥接。在支持Open vSwitch的主机上，还可以通过在接口定义中添加`<virtualport type='openvswitch'/>`来连接到Open vSwitch桥接设备(自0.9.11以来)。Open vSwitch类型的virtualport在其`<parameters>`元素中接受两个参数 - 一个`interfaceid`，这是一个标准的UUID，用于唯一标识Open vSwitch中的这个特定接口(如果您没有指定一个，当您首次定义接口时，将为您生成一个随机的interfaceid)，以及一个可选的`profileid`，它作为接口的“port-profile”发送到Open vSwitch。

```xml
...
<devices>
  ...
  <interface type='bridge'>
    <source bridge='br0'/>
  </interface>
  <interface type='bridge'>
    <source bridge='br1'/>
    <target dev='vnet7'/>
    <mac address="00:11:22:33:44:55"/>
  </interface>
  <interface type='bridge'>
    <source bridge='ovsbr'/>
    <virtualport type='openvswitch'>
      <parameters profileid='menial' interfaceid='09b11c53-8b5c-4eeb-8f00-d84eaa0aaa4f'/>
    </virtualport>
  </interface>
  ...
</devices>
...
```

在内核侧支持Open vSwitch并配置了Midonet Host Agent的主机上，也可以通过在接口定义中添加`<virtualport type='Midonet'/>`来连接到'Midonet'桥接设备。(自1.2.13起)。Midonet虚拟端口类型在其`<parameters>`元素中需要一个`interfaceid`属性。此接口id是指定虚拟网络拓扑中哪个端口将绑定到接口的UUID。

```xml
...
<devices>
  ...
  <interface type='bridge'>
    <source bridge='br0'/>
  </interface>
  <interface type='bridge'>
    <source bridge='br1'/>
    <target dev='vnet7'/>
    <mac address="00:11:22:33:44:55"/>
  </interface>
  <interface type='bridge'>
    <source bridge='midonet'/>
    <virtualport type='midonet'>
      <parameters interfaceid='0b2d64da-3d0e-431e-afdd-804415d6ebbb'/>
    </virtualport>
  </interface>
  ...
</devices>
...
```



##### 	1.20.11.3 (Userspace(SLIRP or passt) connection)

`user`类型通过一个透明的用户空间代理将guest接口连接到外部，该代理不需要任何特殊的系统权限，使其在libvirt本身没有权限运行的情况下可用(例如，libvirt的“会话模式”守护进程，或当libvirt在无特权容器中运行时)。

默认情况下，此用户代理是由QEMU的内部SLRP驱动程序完成的，该驱动程序具有DHCP和DNS服务，这些服务提供从10.0.2.15开始的guestIP地址，默认路由为`10.0.2.2`，DNS服务器为`10.0.2.3`。自3.8.0起，可以通过在其一个强制属性`address`中包含指定IPv4地址的`ip`元素来覆盖默认网络地址。可选地，可以指定`family`属性设置为“ipv6”的第二个ip元素，以向接口添加ipv6地址。`address`可以选择指定地址前缀。

```xml
...
<devices>
  <interface type='user'/>
  ...
  <interface type='user'>
    <mac address="00:11:22:33:44:55"/>
    <ip family='ipv4' address='172.17.2.0' prefix='24'/>
    <ip family='ipv6' address='2001:db8:ac10:fd01::' prefix='64'/>
  </interface>
</devices>
...
```

自9.0.0版本以来，可以通过将界面的`<backend>`子元素`type`属性设置为`passt`来选择`user`界面类型的备用后端实现。在这种情况下，将使用passt传输([https://passt.top)。与SLIRP类似，passt具有内部DHCP服务器，为请求的虚拟机提供一个IPv4和一个IPv6地址；然后，它使用用户空间代理和单独的网络命名空间来提供出站UDP/TCP/ICMP会话，并可选择将目标主机的入站流量重定向到虚拟机。

当使用passt后端时，`<backend>`属性logFile可用于告诉passt进程为此接口的消息日志写入的位置，并且`<source>`属性dev可以告诉它使用特定的主机接口来派生向虚拟机提供用于转发流量的路由。由于passt的设计决策，如果使用SELinux，建议将日志文件放置在passt进程将运行的用户的运行时目录中，最有可能是/run/user/$UID，其中$UID是用户的UID，例如qemu。请注意，为避免可能的问题，特别是由于此日志文件属性主要用于调试，因此libvirt不会创建此目录，除非它已经存在。

此外，当使用passt时，可以添加多个`<portForward>`元素以将目标主机的入站网络流量转发到此虚拟机界面。每个`<portForward>`必须具有`proto`属性(设置为`tcp`或`udp`)，可选的原始地址(如果未指定，则将所有传入的会话转发到给定proto/port的任何主机IP的guest)，以及可选的`dev`属性，以将转发的流量限制为特定的主机接口。

关于要转发哪些端口的决策由`<portForward>`的零个或多个`<range>`子元素描述(如果没有`<range>`，则将转发给定proto/address的所有端口)。每个`<range>`具有一个`start`和可选的`end`属性。如果省略`end`，则将转发单个端口，否则将转发`start`和`end`(包括在内)之间的所有端口。如果端口号应保持不变，因为会话被转发，那么不需要进一步的选项，但如果虚拟机期望在不同的端口上接收会话，则应在`<range>`的`to`属性中指定 - 范围内每个转发的会话的端口号将偏移“`to - start`”。`<range>`元素还可用于指定不应转发的端口范围。这是通过将范围的`exclude`属性设置为`yes`来完成的。这可能看起来不太有用，但在希望转发一长段端口的同时排除某些子集时，可以使用。

```xml
...
<devices>
  ...
  <interface type='user'>
    <backend type='passt' logFile='/run/user/$UID/passt-domain.log'/>
    <mac address="00:11:22:33:44:55"/>
    <source dev='eth0'/>
    <ip family='ipv4' address='172.17.2.4' prefix='24'/>
    <ip family='ipv6' address='2001:db8:ac10:fd01::20'/>
    <portForward proto='tcp'>
      <range start='2022' to='22'/>
    </portForward>
    <portForward proto='udp' address='1.2.3.4'>
      <range start='5000' end='5020' to='6000'/>
      <range start='5010' end='5015' exclude='yes'/>
    </portForward>
    <portForward proto='tcp' address='2001:db8:ac10:fd01::1:10' dev='eth0'>
      <range start='80'/>
      <range start='443' to='344'/>
    </portForward>
  </interface>
</devices>
...
```



##### 	1.20.11.4 (Generic ethernet connection)

提供一种使用新的或现有的tap设备(或veth设备对，取决于hypervisor驱动程序的需求)的方法，该设备部分或完全在libvirt之外进行设置(要么在虚拟机启动之前，要么在通过配置中指定的可选脚本启动虚拟机时)。

可以选择使用`<target>`元素的`dev`属性指定tap设备的名称。如果未指定目标`dev`，则libvirt将创建一个新的标准tap设备，其名称模式为“vnetN”，其中“N”将替换为数字。如果指定了目标dev并且该设备不存在，那么将创建一个具有给定dev名称的新标准tap设备。如果指定的目标dev存在，则将使用该现有设备。通常，libvirt会对设备进行一些基本设置，包括设置MAC地址和IFF_UP标志，但如果dev是一个预先存在的设备，并且`<target>`元素的`managed`属性也设置为“no”(默认值为“yes”)，甚至连这个基本设置都不会执行 - libvirt将只是将设备传递给hypervisor而没有任何设置。从5.7.0版本开始，使用`managed='no'与预先创建的tap设备很有用，因为它允许由非特权libvirtd管理的虚拟机具有基于tap设备的模拟网络设备。

在创建/打开tap设备后，将运行一个可选的shell脚本(在`<script>`元素的`path`属性中给出)。自0.2.1版本以来，此外，在分离/关闭tap设备后，将运行一个可选的shell脚本(在`<downscript>`元素的`path`属性中给出)。自6.4.0版本以来，可以使用这些脚本执行所需的任何额外的主机网络集成。

```xml
...
<devices>
  <interface type='ethernet'>
    <script path='/etc/qemu-ifup-mynet'/>
    <downscript path='/etc/qemu-ifdown-mynet'/>
  </interface>
  ...
  <interface type='ethernet'>
    <target dev='mytap1' managed='no'/>
    <model type='virtio'/>
  </interface>
</devices>
...
```



##### 	1.20.11.5 (Direct attachment to physical interface)

提供虚拟机的NIC到主机的给定物理接口的直接连接。自0.7.7起(仅限QEMU和KVM)

此设置要求Linux macvtap驱动程序可用。(自Linux 2.6.34.以来。)macvtap设备的操作模式可以选择“vepa”(“虚拟以太网端口聚合器” ['Virtual Ethernet Port Aggregator'](https://www.ieee802.org/1/files/public/docs2009/new-evb-congdon-vepa-modular-0709-v01.pdf))、“bridge”或“private”模式之一，“vepa”是默认模式。各个模式导致数据包的传递行为如下：

如果模型类型设置为`virtio`，并且接口的`trustGuestRxFilters`属性设置为`yes`，则对guest中的接口mac地址、单播/多播接收筛选器和vlan设置所做的更改将被监控并传播到主机上的相关macvtap设备(自1.2.10起)。如果未设置`trustGuestRxFilters`，或者使用中的设备型号不支持该过滤器，则尝试更改源自客户端的mac地址将导致网络连接无法工作。

​	`vepa `

​		所有虚拟机的数据包都被发送到外部网桥。目的地是位于与数据包起源相同的主机上的虚拟机的数据包将被 VEPA 兼容的网桥发送回到主机(今天的网桥通常不支持 VEPA)。

​	`bridge`

​		 目的地是与起源相同的主机上的数据包将直接传递到目标 macvtap 设备。为了进行直接传递，起源和目标设备都需要处于桥接模式。如果其中一个设备处于 `vepa` 模式，则需要一个 VEPA 兼容的网桥。

​	`private`

​		 所有数据包都被发送到外部网桥，并且只有当它们通过外部路由器或网关发送回主机时，才会传递到同一主机上的目标虚拟机。如果源设备或目标设备中的任何一个处于`private`模式，将按照此过程进行传递。

​	`passthrough`

​		 此功能将 SRIOV 兼容的网卡的虚拟功能直接附加到虚拟机上，而不会丧失迁移功能。所有数据包都被发送到配置的网络设备的 VF/IF。根据设备的功能，可能会应用额外的先决条件或限制；例如，在 Linux 上，这需要内核版本为 2.6.38 或更新版本。自0.9.2版本以来。

```xml
...
<devices>
  ...
  <interface type='direct' trustGuestRxFilters='no'>
    <source dev='eth0' mode='vepa'/>
  </interface>
</devices>
...
```

直连虚拟机的网络访问可以由主机的物理接口连接到的硬件交换机来管理。

如果交换机符合IEEE 802.1Qbg标准，则接口可以具有如下所示的附加参数。在IEEE 802.1Qbg标准中更详细地记录了虚拟端口元件的参数。这些值是特定于网络的，应由网络管理员提供。在802.1Qbg术语中，虚拟站接口(VSI)表示虚拟机的虚拟接口。自0.8.2

请注意，IEEE 802.1Qbg要求VLAN ID为非零值。

​	`managerid`

​		VSI管理器ID标识包含VSI类型和实例定义的数据库。这是一个整数值，保留值0。

​	`typeid`

​		VSI类型ID标识表征网络接入的VSI类型。VSI类型通常由网络管理员管理。这是一个整数值。

​	`typeidversion`

​		VSI类型版本允许一个VSI类型的多个版本。这是一个整数值。

​	`instanceid`

​		VSI实例ID标识符是在创建VSI实例(即虚拟机的虚拟接口)时生成的。这是一个全局唯一标识符。

```xml
...
<devices>
  ...
  <interface type='direct'>
    <source dev='eth0.2' mode='vepa'/>
    <virtualport type="802.1Qbg">
      <parameters managerid="11" typeid="1193047" typeidversion="2" instanceid="09b11c53-8b5c-4eeb-8f00-d84eaa0aaa4f"/>
    </virtualport>
  </interface>
</devices>
...
```

如果交换机符合IEEE 802.1Qbh标准，则接口可以具有如下所示的附加参数。这些值是特定于网络的，应由网络管理员提供。自0.8.2

​	`profileid`

​		配置文件ID包含要应用到此接口的端口配置文件的名称。该名称由端口配置文件数据库解析为端口配置文件中的网络参数，这些网络参数将应用于该接口。

```xml
...
<devices>
  ...
  <interface type='direct'>
    <source dev='eth0' mode='private'/>
    <virtualport type='802.1Qbh'>
      <parameters profileid='finance'/>
    </virtualport>
  </interface>
</devices>
...
```



##### 1.20.11.6 (PCI Passthrough)

PCI网络设备(由`<source>`元素指定)是通过通用设备直通首先可选择将设备的MAC地址设置为配置的值，然后使用可选指定的`<virtualport>`元素(请参阅上面给出的用于`type='direct'`网络设备的虚拟端口示例)直接分配给虚拟机的。请注意，由于标准的单端口PCI以太网卡驱动程序设计的限制，只能以这种方式分配SR-IOV(Single Root I/O Virtualization)虚拟功能(VF)设备；要将标准的单端口PCI或PCIe以太网卡分配给虚拟机，应使用传统的`<hostdev>`设备定义(自0.9.11起)。

要使用VFIO设备分配而不是传统/传统的KVM设备分配(VFIO是一种与UEFI安全引导兼容的设备分配的新方法)，类型为'hostdev'的接口可以具有一个可选的driver子元素，其中name属性设置为"vfio"。要使用传统的KVM设备分配，可以将name设置为"kvm"(默认值为在支持VFIO驱动程序的系统上为"vfio"，在较旧的系统上为"kvm")(自1.1.3起，在此之前默认值始终为"kvm")。

请注意，这种网络设备的"智能直通"功能与标准的`<hostdev>`设备的功能非常相似，不同之处在于此方法允许指定传递的设备的MAC地址和`<virtualport>`。如果不需要这些功能，如果 有一个不支持SR-IOV的标准单端口PCI、PCIe或USB网络卡(因此在分配给虚拟机域后重置时将丢失配置的MAC地址)，或者如果 正在使用早于0.9.11的libvirt版本，则应使用标准的`<hostdev>`而不是`<interface type='hostdev'/>`将设备分配给虚拟机。

类似于标准的`<hostdev>`设备的功能，当`managed`为"yes"时，在传递给虚拟机之前，它会从主机上分离，并在虚拟机退出后重新连接到主机。如果省略了managed或"no"，则用户需要在启动虚拟机或热插拔设备之前调用`virNodeDeviceDettach`(或`virsh nodedev-detach`)，在热拔插或停止虚拟机后调用`virNodeDeviceReAttach`(或`virsh nodedev-reattach`)来分离设备，并重新连接设备。

```xml
...
<devices>
  <interface type='hostdev' managed='yes'>
    <driver name='vfio'/>
    <source>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x07' function='0x0'/>
    </source>
    <mac address='52:54:00:6d:90:02'/>
    <virtualport type='802.1Qbh'>
      <parameters profileid='finance'/>
    </virtualport>
  </interface>
</devices>
...
```



##### 	1.20.11.7 (vDPA devices)

vDPA网络设备可用于在域内提供线速网络性能。vDPA设备是一种特殊类型的网络设备，它使用符合virtio规范但具有特定于供应商的控制路径的数据路径。要将此类设备与libvirt一起使用，主机设备必须已绑定到相应的设备特定的vDPA驱动程序。这将创建一个vDPA char设备(例如/dev/vhost-vDPA-0)，可用于将该设备分配给libvirt域。自6.9.0起(仅限QEMU，需要QEMU 5.1.0或更新版本)

```xml
...
<devices>
  <interface type='vdpa'>
    <source dev='/dev/vhost-vdpa-0'/>
  </interface>
</devices>
...
```



##### 1.20.11.8 (Teaming a virtio / hostdev NIC pair)

自6.1.0(仅限QEMU和KVM，需要QEMU 4.2.0或更新版本，以及一个支持“**failover**故障切换”功能的guest virtio网络驱动程序，例如Linux内核4.18及更新版本中包含的驱动程序)，两个接口的`<teaming>`元素可用于将它们连接为guest中的团队/绑定设备(假设hypervisor和guest网络驱动程序中有适当的支持)。

```xml
...
<devices>
  <interface type='network'>
    <source network='mybridge'/>
    <mac address='00:11:22:33:44:55'/>
    <model type='virtio'/>
    <teaming type='persistent'/>
    <alias name='ua-backup0'/>
  </interface>
  <interface type='network'>
    <source network='hostdev-pool'/>
    <mac address='00:11:22:33:44:55'/>
    <model type='virtio'/>
    <teaming type='transient' persistent='ua-backup0'/>
  </interface>
</devices>
...
```

本例中的第二个接口引用的网络是SRIOV VF池(即“hostdev network”)。您可以直接参考SRIOV VF设备：

```xml
...
  <interface type='hostdev'>
    <source>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x07' function='0x0'/>
    </source>
    <mac address='00:11:22:33:44:55'/>
    <teaming type='transient' persistent='ua-backup0'/>
  </interface>
...
```

`<teaming>`元素所需的属性`type`将设置为`persistent`，以指示应始终存在于域中的设备，或`transient`，以指示可能定期删除的设备，然后重新添加到域中。当`type=“transient”`时，`<teaming>`应该有一个名为`persistent`的第二个属性-该属性应该设置为对中另一个设备的别名(具有`<teamingtype=“persistent'/>`的设备)。

在QEMU的特殊情况下，libvirt的`<teaming>`元素用于设置virtio-net“故障切换”设备对。对于此设置，持久设备必须是具有`<model type="virtio"/>`的接口，而瞬态设备必须是`<interface type="hostdev"/>`(或`<interface type='network'/>`，其中引用的网络定义了SRIOV VF池)。然后，guest将拥有一个由virtio NIC+hostdev NIC对组成的简单网络团队/绑定设备。在这种配置中，性能更高的hostdev NIC通常是所有网络流量的首选，但当域迁移时，QEMU将自动从客户机上拔下VF，然后在迁移完成后热插拔类似的设备；在进行迁移时，网络流量将使用virtio NIC。(当然，模拟的virtio NIC和hostdev NIC必须连接到同一个子网才能正常工作)。

自7.1.0`<teaming>`元素也可以添加到普通的`<hostdev>`设备中。

```xml
...
  <hostdev mode='subsystem' type='pci' managed='no'>
    <source>
      <address domain='0x0000' bus='0x00' slot='0x07' function='0x0'/>
    </source>
    <teaming type='transient' persistent='ua-backup0'/>
  </hostdev>
...
```

该设备必须是网络设备，但不一定是SRIOV VF。如果要分配VFIO的设备是标准NIC(而不是VF)，或者如果libvirt没有设置VF MAC地址所需的资源和权限(例如，如果libvirt在无特权或容器中运行)，则使用普通的`<hostdev>`而不是`<interface type='hostdev'>`或`<interface type='network'>`是有用的。这当然意味着用户(或另一个应用程序)负责设置设备的MAC地址，使其能够在客户驱动程序初始化后幸存下来。对于标准NIC(即不是SRIOV VF)，这可能意味着NIC的工厂编程MAC地址将需要用于组队对(因为guest中的任何驱动程序init都会将MAC重置回工厂)。如果是SRIOV VF，则需要通过VF的PF设置其MAC地址，例如，如果您要使用PF enp2sf1的VF 2，则可以使用类似以下命令的内容：

```bash
ip link set enp2s0f1 vf 2 mac 52:54:00:11:22:33
```

NB1：由于在配置hostdev NIC时必须知道virtio NIC的别名，因此需要在virtio网卡的配置中手动设置它(与所有其他手动设置的别名一样，这意味着它必须以“ua-”开头)。

NB2：目前唯一支持virtio网络故障切换的guest OS virtio net驱动程序的实现要求virtio和hostdev NIC的MAC地址必须匹配。由于这在未来可能并不总是一个要求，libvirt不会强制执行此限制——这取决于创建配置的人员/管理应用程序，以确保两个设备的MAC地址匹配。

NB3：由于作为迁移源和目的地的主机上SRIOV VF的PCI地址几乎肯定会不同，因此更高级别的管理软件需要在迁移开始时修改hostdev NIC的`<source>`(`<interface type='hostdev'>`)，或者(一个更简单的解决方案)配置将需要使用一个libvirt“hostdev”虚拟网络，该网络维护一个此类设备池，正如示例中使用名为“hostdevpool”的libvirt网络所暗示的那样——只要两个主机上的hostdev网络池具有相同的名称，libvirt本身就会负责在迁移的两端分配适当的设备。类似地，virtio接口的XML也必须在迁移的源和目的地上正确工作(例如，通过连接到两台主机上的同一网桥设备，或者通过使用相同的虚拟网络)，或者管理软件必须在迁移过程中正确修改接口XML，以便在迁移前后virtio设备保持连接到同一网段。



##### 1.20.11.9 (Multicast tunnel)

多播组被设置为表示虚拟网络。网络设备位于同一多播组中的任何虚拟机都可以相互通信，甚至可以跨主机进行通信。无特权用户也可以使用此模式。没有默认的DNS或DHCP支持，也没有传出网络访问。要提供传出网络访问，其中一个虚拟机应具有第二个NIC，该NIC连接到前4种网络类型中的一种，并执行适当的路由。多播协议也与用户模式linux客户使用的协议兼容。使用的源地址必须来自多播地址块。

```xml
...
<devices>
  <interface type='mcast'>
    <mac address='52:54:00:6d:90:01'/>
    <source address='230.0.0.1' port='5558'/>
  </interface>
</devices>
...
```



##### 1.20.11. 10 (TCP tunnel)

TCP客户端/服务器体系结构提供了一个虚拟网络。一个VM提供网络的服务器端，所有其他VMS都配置为客户端。所有网络流量都通过服务器在虚拟机之间路由。无特权用户也可以使用此模式。没有默认的DNS或DHCP支持，也没有传出网络访问。要提供传出网络访问，其中一个虚拟机应具有第二个NIC，该NIC连接到前4种网络类型中的一种，并执行适当的路由。

```xml
...
<devices>
  <interface type='server'>
    <mac address='52:54:00:22:c9:42'/>
    <source address='192.168.0.1' port='5558'/>
  </interface>
  ...
  <interface type='client'>
    <mac address='52:54:00:8b:c9:51'/>
    <source address='192.168.0.1' port='5558'/>
  </interface>
</devices>
...
```



##### 1.20.11.11 (UDP unicast tunnel)

UDP单播体系结构提供了一个虚拟网络，该网络使用QEMU的UDP基础设施实现QEMU实例之间的连接。xml“源”地址是UDP套接字数据包将从运行QEMU的主机发送到的端点地址。xml“本地”地址是接口的地址，UDP套接字数据包将从该接口源自QEMU主机。自1.2.20

```xml
...
<devices>
  <interface type='udp'>
    <mac address='52:54:00:22:c9:42'/>
    <source address='127.0.0.1' port='11115'>
      <local address='127.0.0.1' port='11116'/>
    </source>
  </interface>
</devices>
...
```



##### 1.20.11.12 (Null network interface)

未连接的网络接口听起来毫无意义，但可以在没有任何指定网络连接的情况下显示在VMWare中。自8.7.0

```xml
...
<devices>
  <interface type='null'>
    <mac address='52:54:00:22:c9:42'/>
  </interface>
</devices>
...
```



##### 1.20.11.13 (VMware Distributed Switch)

接口可以连接到VMWare分布式交换机，但由于libvirt无法提供有关该体系结构的信息，因此此处提供的信息只能从VM配置中收集。可以创建具有此接口类型的虚拟机，这样XML的编辑就可以正常工作，但是libvirt不能保证这些参数中的任何更改在系统hypervisor中都是有效的。自8.7.0

```xml
...
<devices>
  <interface type='vds'>
    <mac address='52:54:00:22:c9:42'/>
    <source switchid='12345678-1234-1234-1234-123456789abc' portid='6' portgroupid='pg-4321' connectionid='12345'/>
  </interface>
</devices>
...
```



##### 1.20.11.14 (Setting the NIC model)

```xml
...
<devices>
  <interface type='network'>
    <source network='default'/>
    <target dev='vnet1'/>
    <model type='ne2k_pci'/>
  </interface>
</devices>
...
```

对于支持此功能的虚拟机监控程序， 可以设置模拟网络接口卡的型号。

类型的值不是由libvirt专门定义的，而是由底层hypervisor支持的内容(如果有的话)定义的。对于QEMU和KVM， 可以使用以下命令获得支持的型号列表：

```xml
qemu -net nic，model=? /dev/null
qemu-kvm -net nic，model=? /dev/null
```

QEMU和KVM的典型值包括：ne2k_isa i82551 i82557b i82559er ne2k_pci pcnet rtl8139 e1000 virtio。从5.2.0开始，支持`virtio transitional`和`virtio non-transitional`值。有关更多详细信息，请参阅[Virtio transitional devices](https://libvirt.org/formatdomain.html#virtio-transitional-devices) 。自9.3.0还支持 igb。



##### 1.20.11.15 (Setting NIC driver-specific options)

```xml
...
<devices>
  <interface type='network'>
    <source network='default'/>
    <target dev='vnet1'/>
    <model type='virtio'/>
    <driver name='vhost' txmode='iothread' ioeventfd='on' event_idx='off' queues='5' rx_queue_size='256' tx_queue_size='256'>
      <host csum='off' gso='off' tso4='off' tso6='off' ecn='off' ufo='off' mrg_rxbuf='off'/>
      <guest csum='off' tso4='off' tso6='off' ecn='off' ufo='off'/>
    </driver>
    </interface>
</devices>
...
```

某些NIC可能具有特定于驱动程序的可调选项。这些被设置为接口定义的驱动程序子元素的属性。目前，以下属性可用于virtio NIC驱动程序：

​	`name` 

​		可选的`name`属性用于强制指定使用哪种类型的后端驱动程序。该值可以是'qemu'(用户空间后端)或'vhost'(内核后端，需要内核提供vhost模块)；如果尝试要求在没有内核支持的情况下使用vhost驱动程序，则将被拒绝。如果未提供此属性，则如果存在，域将默认为'vhost'，但在没有错误的情况下悄悄回退到'qemu'。自0.8.8以来(仅限于QEMU和KVM)对于类型为'type='hostdev''(PCI直通设备)的接口，`name`属性可以选择设置为"vfio"或"kvm"。 "vfio"告诉libvirt使用VFIO设备分配而不是传统的KVM设备分配(VFIO是与UEFI安全引导兼容的设备分配的新方法)，而"kvm"告诉libvirt使用kvm内核模块直接执行的传统设备分配(默认值目前为"kvm"，但可能会更改)。自1.0.5以来(仅限于QEMU和KVM，需要内核3.6或更新版本)对于类型为'type='vhostuser''的接口，将忽略`name`属性。始终使用vhost-user作为后端驱动程序。

​	`txmode `

​		`txmode`属性指定在传输缓冲区已满时如何处理数据包的传输。该值可以是'iothread'或'timer'。自0.8.8以来(仅限于QEMU和KVM)如果设置为'iothread'，数据包的传输全部在底层驱动程序的iothread中完成(此选项相当于在qemu命令行的-device virtio-net-pci选项中添加"tx=bh")。如果设置为'timer'，传输工作在qemu中完成，如果有更多的传输数据，超过当前时间可以发送的数据，那么在qemu继续执行其他操作之前，将设置一个计时器；当计时器触发时，将尝试发送更多数据。根据添加了此选项的qemu开发人员的说法，结果的不同之处在于："bh使tx更具异步性并减少延迟，但可能会导致处理器带宽争用更多，因为执行tx的CPU不一定是生成数据包的虚拟机CPU。"通常情况下，您应该保持此选项不变，除非您非常确定自己知道正在做什么。

​	`ioeventfd`

​		 此可选属性允许用户设置接口设备的域I/O异步处理 [domain I/O asynchronous handling](https://patchwork.kernel.org/patch/43390/)。默认值由虚拟化监控程序自行决定。接受的值为"on"和"off"。启用此选项允许qemu在单独的线程处理I/O时执行虚拟机。通常，经历I/O期间高系统CPU利用率的虚拟机将从中受益。另一方面，在负载过重的主机上，它可能会增加虚拟机I/O的延迟。自0.9.3以来(仅限于QEMU和KVM)通常情况下，您应该保持此选项不变，除非您非常确定自己知道正在做什么。

​	`event_idx`

​		 `event_idx`属性控制某些设备事件处理的一些方面。该值可以是'on'或'off' - 如果设置为'on'，它将减少虚拟机的中断和退出次数。默认值由QEMU确定；通常情况下，如果支持该功能，则默认值为'on'。如果存在情况下，这种行为不够理想，此属性提供了一种强制关闭该功能的方法。自0.9.5以来(仅限于QEMU和KVM)通常情况下，您应该保持此选项不变，除非您非常确定自己知道正在做什么。

​	`queues `

​		`queues`属性是可选的，用于控制 [Multiqueue virtio-net](https://www.linux-kvm.org/page/Multiqueue)或vhost-user(请参阅1.20.11.28 [vhost-user interface](https://libvirt.org/formatdomain.html#vhost-user-interface))网络接口使用的队列数量。使用多个数据包处理队列要求接口具有<model type='virtio'/>元素。每个队列可能由不同的处理器处理，从而实现更高的吞吐量。virtio-net自1.0.6以来(仅适用于QEMU和KVM)vhost-user自1.2.17以来(仅适用于QEMU和KVM)

​	`rx_queue_size `

​		`rx_queue_size`属性是可选的，用于控制上述每个队列的virtio环的大小。默认值取决于hypervisor，并可能在其版本发布时更改。此外，一些hypervisor对实际值可能会有一些限制。例如，最新的QEMU(截至2016-09-01)要求值为[256， 1024]范围内的2的幂。**自2.3.0以来(仅适用于QEMU和KVM)通常情况下，除非您非常确定自己知道正在做什么，否则应保持此选项不变。**

​	`tx_queue_size `

​		`tx_queue_size`属性是可选的，用于控制上述每个队列的virtio环的大小。默认值取决于虚拟化管理程序，并可能在其版本发布时更改。此外，一些虚拟化管理程序对实际值可能会有一些限制。例如，QEMU v2.9要求值为[256， 1024]范围内的2的幂。除此之外，这可能仅适用于某些接口类型，例如前面提到的QEMU仅为vhostuser类型启用此选项。自3.7.0以来**(仅适用于QEMU和KVM)通常情况下，除非您非常确定自己知道正在做什么，否则应保持此选项不变。**

​	`rss `

​		rss选项启用virtio NIC的in-qemu/ebpf RSS。RSS仅适用于virtio和tap后端。Virtio NIC将以"rss"属性启动。目前，libvirt支持"in-qemu" RSS。如果QEMU具有CAP_SYS_ADMIN权限，它可能会加载eBPF RSS，但libvirt默认情况下不支持。自8.3.0和QEMU 5.1以来，通常情况下，**除非您非常确定自己知道正在做什么，否则应保持此选项不变。正确的RSS配置取决于vcpu、tap和vhost设置。**

​	`rss_hash_report `

​		`rss_hash_report`选项启用virtio NIC的in-qemu RSS哈希报告。Virtio NIC将以"hash"属性启动。提供给VM的网络数据包将在virt头中包含数据包的哈希。通常与`rss`一起启用。如果没有`rss`选项，哈希报告不会影响定向本身，但会提供包含计算的哈希的vnet头。自8.3.0和QEMU 5.1以来，**通常情况下，除非您非常确定自己知道正在做什么，否则应保持此选项不变。正确的RSS配置取决于vcpu、tap和vhost设置。**

​	virtio选项 

​		对于virtio接口，还可以设置 [Virtio-related options](https://libvirt.org/formatdomain.html#virtio-related-options) 。自3.5.0以来

主机和客户机的卸载选项可以使用以下子元素进行配置：

​	`host`

​		`csum、gso、tso4、tso6、ecn`和`ufo`属性(可能值为`on`和`off`)可用于关闭主机卸载选项。默认情况下，支持的卸载由QEMU启用。自1.2.9起(仅限QEMU)`mrg_rxbuf`属性可用于控制主机端的可合并rx缓冲区。可能的值有`on`(默认)和`off`。自1.2.13起(仅限QEMU)

​	`guest`

​		`csum、tso4、tso6、ecn`和`ufo`属性(可能值为`on`和`off`)可用于关闭客户机卸载选项。默认情况下，支持的卸载由QEMU启用。自1.2.9起(仅限QEMU)



##### 1.20.11.16 (Setting network backend-specific options)

```xml
...
<devices>
  <interface type='network'>
    <source network='default'/>
    <target dev='vnet1'/>
    <model type='virtio'/>
    <backend tap='/dev/net/tun' vhost='/dev/vhost-net'/>
    <driver name='vhost' txmode='iothread' ioeventfd='on' event_idx='off' queues='5'/>
    <tune>
      <sndbuf>1600</sndbuf>
    </tune>
  </interface>
</devices>
...
```

为了调整网络的后端，可以使用`backend`元素。`vhost`属性可以覆盖具有`virtio`模型的设备的默认vhost-设备路径(`/dev/vhost-net`)。`tap`属性覆盖网络和网桥接口的tun/tap设备路径(默认值：`/dev/net/tun`)。这在会话模式下不起作用。自1.2.9

对于tap 设备，还有`sndbuf`元素，它可以调整主机中发送缓冲区的大小。自0.8.8



##### 1.20.11.17 (Overriding the target element)

```xml
...
<devices>
  <interface type='network'>
    <source network='default'/>
    <target dev='vnet1'/>
  </interface>
</devices>
...
```

如果没有指定目标，某些hypervisor将自动为创建的tun设备生成一个名称。可以手动指定此名称，但名称不应以“vnet”、“vif”、“macvtap”或“macvlan”开头，这些前缀由libvirt和某些hypervisor保留。可以忽略使用这些前缀手动指定的目标。

注意，对于LXC容器，这定义了主机端接口的名称。1.2.7起，为了在客户端定义设备的名称，应使用`guest`元素，如以下片段所示：

```xml
...
<devices>
  <interface type='network'>
    <source network='default'/>
    <guest dev='myeth'/>
  </interface>
</devices>
...
```



##### 1.20.11.18 (Specifying boot order)

```xml
...
<devices>
  <interface type='network'>
    <source network='default'/>
    <target dev='vnet1'/>
    <boot order='1'/>
  </interface>
</devices>
...
```

对于支持此功能的虚拟机监控程序，您可以设置用于网络引导的特定NIC。`order`属性确定在引导序列中尝试设备的顺序。在BIOS引导加载程序部分，每个设备的`boot`元素不能与常规引导元素一起使用。自0.8.8



##### 1.20.11.19 (Interface ROM BIOS configuration)

```xml
...
<devices>
  <interface type='network'>
    <source network='default'/>
    <target dev='vnet1'/>
    <rom bar='on' file='/etc/fake/boot.bin'/>
  </interface>
</devices>
...
```

对于支持此功能的hypervisor， 可以更改PCI网络设备的ROM显示给 guest的方式。`bar`属性可以设置为“on”或“off”，并确定设备的ROM在guest的内存映射中是否可见。(在PCI文档中，“rombar”设置控制ROM的基址寄存器的存在)。如果没有指定rom条，则将使用qemu默认值(较旧版本的qemu使用默认值“off”，而较新版本的qemos使用默认值为“on”)。可选的`file`属性用于指向将作为设备的ROM BIOS呈现给 guest的二进制文件。这对于提供用于网络设备的替代引导ROM是有用的。自0.9.10起(仅限QEMU和KVM)。



##### 1.20.11.20 (Setting up a network backend in a driver domain)

```xml
...
<devices>
  ...
  <interface type='bridge'>
    <source bridge='br0'/>
    <backenddomain name='netvm'/>
  </interface>
  ...
</devices>
...
```

可选的`backenddomain`元素允许为接口指定后端域(也称为驱动程序域)。使用`name`属性可以指定后端域名。 可以使用它在域之间创建直接的网络链接(这样数据就不会通过主机系统)。与类型“ethernet”一起使用可创建纯网络链接，或与类型“bridge”一起使用以连接到后端域内的网桥。自1.2.13起(仅限Xen)



##### 1.20.11.21 (Quality of service)

```xml
...
<devices>
  <interface type='network'>
    <source network='default'/>
    <target dev='vnet0'/>
    <bandwidth>
      <inbound average='1000' peak='5000' floor='200' burst='1024'/>
      <outbound average='128' peak='256' burst='256'/>
    </bandwidth>
  </interface>
</devices>
...
```

接口XML的这一部分提供了设置服务质量。传入和传出流量可以独立成形。`bandwidth`元素及其子元素在网络XML的[QoS](https://libvirt.org/formatnetwork.html#quality-of-service) 部分中进行了描述



##### 1.20.11.22 (Setting VLAN tag(on supported network types only))

```xml
...
<devices>
  <interface type='bridge'>
    <vlan>
      <tag id='42'/>
    </vlan>
    <source bridge='ovsbr0'/>
    <virtualport type='openvswitch'>
      <parameters interfaceid='09b11c53-8b5c-4eeb-8f00-d84eaa0aaa4f'/>
    </virtualport>
  </interface>
  <interface type='bridge'>
    <vlan trunk='yes'>
      <tag id='42'/>
      <tag id='123' nativeMode='untagged'/>
    </vlan>
    ...
  </interface>
</devices>
...
```

如果(且仅当)guest使用的网络连接支持对guest透明的VLAN标记，则可选的`<VLAN>`元素可以指定一个或多个VLAN标记以应用于guest的网络流量。自0.10.0起。支持guest透明VLAN标记的网络连接包括1)type='bridge'接口，该接口连接到Open vSwitch桥接。自0.100起，2)SRIOV虚拟功能(VF)自0.10.0起通过type='hostdev'(直接设备分配)使用，3)SRIOV虚拟功能自1.3.5起通过type='direct'使用，mode='passthrough'(macvtap“passthrough”模式)使用。所有其他连接类型，包括标准linux网桥和libvirt自己的虚拟网络，不支持。802.1Qbh(vn链路)和802.1Qbg(VEPA)交换机提供它们自己的方式(在libvirt之外)来将guest流量标记到特定的VLAN上。每个标记都在`<vlan>`的一个单独的`<tag>`子元素中给出(例如：`<tag id='42'/>`)。对于多个标签的VLAN中继(仅在Open vSwitch连接上支持)，可以指定多个`<tag>`子元素，这意味着用户希望在接口上为所有指定的标签进行VLAN中继。在需要单个标签的VLAN中继的情况下，可以将可选属性trunk='yes'添加到顶层`<VLAN>`元素，以区分单个标签的中继和正常标记。

对于使用Open vSwitch的网络连接，自1.1.0起，还可以配置“本机标记”和“本机未标记”VLAN模式。这是通过`<tag>`子元素上的可选`nativeMode`属性完成的：`nativeMode`可以设置为“taged”或“untaged”。包含本机模式的`<tag>`子元素的id属性设置哪个VLAN被视为该接口的“本机”VLAN，而本机模式属性确定是否标记该VLAN的流量。



##### 1.20.11.23 (Isolating guests'network traffic from each other)

```xml
...
<devices>
  <interface type='network'>
    <source network='default'/>
    <port isolated='yes'/>
  </interface>
</devices>
...
```

自6.1.0。当设置为`yes`(默认设置为`no`)时，`port`元素属性`isolated`用于将此接口的网络流量与连接到同一网络的其他 guest接口的网络通信量隔离开来，这些接口也具有`<port isolated='yes'/>`。只有使用标准tap设备通过Linux host bridge连接到网络的模拟接口设备才支持此设置。此属性可以从libvirt网络继承，因此，如果将连接到网络的所有 guest都应该隔离，则最好将该设置放在网络配置中。(注意：这只会阻止`isolated='yes'`的guest相互交流；如果同一个桥上有一个guest没有`isolated='yes'`，即使是隔离的guest也可以与之交流。)

##### 1.20.11.24 (Modifying virtual link state)

```xml
...
<devices>
  <interface type='network'>
    <source network='default'/>
    <target dev='vnet0'/>
    <link state='down'/>
  </interface>
</devices>
...
```

该元素提供了设置虚拟网络链路的状态的手段。属性`state`的可能值为`up`和`down`。如果将`down`指定为值，则接口的行为就像断开了网络电缆一样。如果未指定此元素，则默认行为是`on`链接状态。自0.9.5



##### 1.20.11.25 (MTU configuration)

```xml
...
<devices>
  <interface type='network'>
    <source network='default'/>
    <target dev='vnet0'/>
    <mtu size='1500'/>
  </interface>
</devices>
...
```

该元素提供了设置虚拟网络链路的MTU的手段。目前只有一个属性`size`接受非负整数，该整数指定接口的MTU大小。自3.1.0



##### 1.20.11.26 (Coalesce settings)

```xml
...
<devices>
  <interface type='network'>
    <source network='default'/>
    <target dev='vnet0'/>
    <coalesce>
      <rx>
        <frames max='7'/>
      </rx>
    </coalesce>
  </interface>
</devices>
...
```

该元素提供了为一些接口设备设置合并设置的方法(目前仅类型为`network`和`bridge`。目前，`rx`组的元素`frames`中只有一个属性`max`需要调整，它接受一个非负整数，该整数指定中断前将接收的最大数据包数。自3.3.0



##### 1.20.11.27 (IP configuration)

```xml
...
<devices>
  <interface type='network'>
    <source network='default'/>
    <target dev='vnet0'/>
    <ip address='192.168.122.5' prefix='24'/>
    <ip address='192.168.122.5' prefix='24' peer='10.0.0.10'/>
    <route family='ipv4' address='192.168.122.0' prefix='24' gateway='192.168.122.1'/>
    <route family='ipv4' address='192.168.122.8' gateway='192.168.122.1'/>
  </interface>
  ...
  <hostdev mode='capabilities' type='net'>
    <source>
      <interface>eth0</interface>
    </source>
    <ip address='192.168.122.6' prefix='24'/>
    <route family='ipv4' address='192.168.122.0' prefix='24' gateway='192.168.122.1'/>
    <route family='ipv4' address='192.168.122.8' gateway='192.168.122.1'/>
  </hostdev>
  ...
</devices>
...
```

1.2.12起具有网络功能的网络设备和hostdev设备可以选择性地提供一个或多个IP地址，以便在客户机的网络设备上设置。请注意，某些hypervisor或网络设备类型将简单地忽略它们或只使用第一种。族属性可以设置为`ipv4`或`ipv6`，`address`属性包含IP地址。可选`prefix`是网络掩码中的1比特数，如果未指定，将自动设置-对于IPv4，默认前缀根据网络“类别”(A、B或C-请参阅RFC870)确定，对于IPv6，默认前缀为64。可选的`peer`属性保存点对点网络设备另一端的IP地址(自2.1.0起)。

1.2.12起路由元素也可以添加，以定义要添加到客户中的IP路由。该元素的属性在 [network definitions](https://libvirt.org/formatnetwork.html#static-routes)中的`route`元素文档中进行了描述。由LXC驱动程序使用的。

```xml
...
<devices>
  <interface type='ethernet'>
    <source/>
      <ip address='192.168.123.1' prefix='24'/>
      <ip address='10.0.0.10' prefix='24' peer='192.168.122.5'/>
      <route family='ipv4' address='192.168.42.0' prefix='24' gateway='192.168.123.4'/>
    <source/>
    ...
  </interface>
  ...
</devices>
...
```

2.1.0起“ethernet”类型的网络设备可以选择提供一个或多个IP地址和一个或更多路由，以便在网络设备的主机侧进行设置。这些元素被配置为接口的`<source>`元素的子元素，并且具有与用于配置接口的客户端(如上所述)的类似名称的元素相同的属性。



##### 1.20.11.28 (vhost-user interface)

自1.2.7以来，vhost用户使用Virtio传输协议实现了QEMU虚拟机和其他用户空间进程之间的通信。char-dev(例如Unix套接字)用于控制平面，而数据平面基于共享内存。

```xml
...
<devices>
  <interface type='vhostuser'>
    <mac address='52:54:00:3b:83:1a'/>
    <source type='unix' path='/tmp/vhost1.sock' mode='server'/>
    <model type='virtio'/>
  </interface>
  <interface type='vhostuser'>
    <mac address='52:54:00:3b:83:1b'/>
    <source type='unix' path='/tmp/vhost2.sock' mode='client'>
      <reconnect enabled='yes' timeout='10'/>
    </source>
    <model type='virtio'/>
    <driver queues='5'/>
  </interface>
</devices>
...
```

`<source>`元素必须与char设备的类型一起指定。目前，只支持type='unix'，其中需要路径(套接字的目录路径)和模式属性。同时支持`mode='server'`和`mode='client'`。vhost-user需要virtio模型类型，因此`<model>`元素是必需的。自4.1.0，该元素有一个可选的子元素`reconnect`，如果连接丢失，该子元素会配置重新连接超时。它`enabled`了两个属性(接受yes和no)和`timeout`，`timeout`指定系统hypervisor尝试重新连接的秒数。



##### 1.20.11.29 (Traffic filtering with NWFilter)

0.8.0起，`nwfilter`配置文件可以分配给域接口，这允许为虚拟机配置流量过滤规则。有关更完整的详细信息，请参阅[nwfilter](https://libvirt.org/formatnwfilter.html) 文档。

```xml
...
<devices>
  <interface ...>
    ...
    <filterref filter='clean-traffic'/>
  </interface>
  <interface ...>
    ...
    <filterref filter='myfilter'>
      <parameter name='IP' value='104.207.129.11'/>
      <parameter name='IP6_ADDR' value='2001:19f0:300:2102::'/>
      <parameter name='IP6_MASK' value='64'/>
      ...
    </filterref>
  </interface>
</devices>
...
```

`filter`属性指定要使用的nwfilter的名称。可以指定可选的`<parameter>`元素，用于通过`name`和`value`属性向nwfilter传递附加信息。有关参数的信息，请参阅[nwfilter](https://libvirt.org/formatnwfilter.html#usage-of-variables-in-filters)文档。



#### 	1.20.12 (Input devices)

输入设备允许与客户虚拟机中的图形帧缓冲区进行交互。当启用framebuffer帧缓冲区时，会自动提供一个输入设备。可以明确地添加附加设备，例如，提供用于绝对光标移动的图形板。

```xml
...
<devices>
  <input type='mouse' bus='usb'/>
  <input type='keyboard' bus='usb'/>
  <input type='mouse' bus='virtio'/>
  <input type='keyboard' bus='virtio'/>
  <input type='tablet' bus='virtio'/>
  <input type='passthrough' bus='virtio'>
    <source evdev='/dev/input/event1'/>
  </input>
  <input type='evdev'>
    <source dev='/dev/input/event1234' grab='all' repeat='on' grabToggle='ctrl-ctrl'/>
  </input>
</devices>
...
```

`input`

​	`input`元素有一个强制属性，该`type`的值可以是“mouse”、“tablet”、(自1.2.2起)“keyboard”、(从1.3.0起)“passthrough”或(自7.4.0以来)“evdev”。平板电脑提供绝对的光标移动，而鼠标使用相对移动。可选的`bus`属性可用于细化确切的设备类型。它采用值“xen”(半虚拟化)、“ps2”和“usb”或(自1.3.0起)“virtio”。

`input`元素有一个可选的子元素`<address>`，它可以将设备连接到特定的PCI插槽，记录[Device Addresses](https://libvirt.org/formatdomain.html#device-addresses)。在S390上，地址可用于为输入设备提供CCW地址(自4.2.0起)。对于`passthrough`和`evdev`类型，强制子元素`source`必须具有`evdev`(`passthrough`)或`dev`(`evdev`)属性，该属性包含传递给guest的事件设备的绝对路径。对于类型`evdev，source`有三个可选属性`grab`，值为“all”，当启用时会抓取所有输入设备，而不是仅抓取一个，用值“on”/“off”重复以启用/禁用自动重复事件，并用值`ctrl-ctrl、alt-alt、shift-shift、meta-meta、scrolllock`或`ctrl-scolllock`切换`grabToggle`(自7.6.0起)以更改抓取键组合。输入类型`evdev`目前仅在linux设备上受支持。(仅限KVM)从5.2.0开始，`input`元素接受一个值为“virtio”、“virtio-transitional”和“virtio-notransitional)的`mode`属性。有关更多详细信息，请参阅1.20.5 [Virtio transitional devices](https://libvirt.org/formatdomain.html#virtio-transitional-devices)。

子元素`driver`可用于调整设备的virtio选项：还可以设置1.20.4 [Virtio-related options](https://libvirt.org/formatdomain.html#virtio-related-options)。(自3.5.0起)



#### 1.20.13 (Hub devices)

集线器是一种将单个端口扩展为多个端口的设备，以便有更多端口可用于将设备连接到主机系统。

```xml
...
<devices>
  <hub type='usb'/>
</devices>
...
```

`hub`

​	`hub`元素有一个强制属性，该`type`的值只能是“usb”。



#### 1.20.14 图形帧缓冲区(Graphical framebuffers)

图形设备允许与guest OS进行图形交互。guest通常会配置一个帧缓冲区或一个文本控制台，以允许与管理员交互。

```xml
...
<devices>
  <graphics type='sdl' display=':0.0'/>
  <graphics type='vnc' port='5904' sharePolicy='allow-exclusive'>
    <listen type='address' address='1.2.3.4'/>
  </graphics>
  <graphics type='rdp' autoport='yes' multiUser='yes' />
  <graphics type='desktop' fullscreen='yes'/>
  <graphics type='spice'>
    <listen type='network' network='rednet'/>
  </graphics>
</devices>
...
```

`graphics`

​	`graphics`元素具有一个强制`type`属性，该属性的值为`sdl、vnc、spice、rdp、desktop`或`egl-headless`：

​	`sdl`

​		这会在主机桌面上显示一个窗口，可以提供3个可选参数：`display`属性用于指定要使用的显示器，`xauth`属性用于身份验证标识符，还有一个可选的`fullscreen`属性，接受`yes`或`no`的值。

​	 可以使用带有`enable="yes"`属性的`gl`来启用SDL中的OpenGL支持。同样， 可以使用`enable="no"`明确禁用OpenGL支持。

​	`vnc`

​		启动一个VNC服务器。`port`属性指定TCP端口号(-1是旧的语法，表示应该自动分配)。`autoport`属性是指示自动分配TCP端口的新首选语法。`passwd`属性以明文提供VNC密码。如果`passwd`属性设置为空字符串，则禁用VNC访问。`keymap`属性指定要使用的键映射。可以通过给定一个时间戳`passwdValidTo='2010-04-09T15:51:00'`来限制密码的有效性，假设该时间戳在UTC时间中。`connected`属性允许在更改密码时控制连接的客户端。自0.9.3版本起，VNC仅接受`keep`值。注意，这可能不被所有的hypervisor支持。

​		可选的`sharePolicy`属性指定VNC服务器显示共享策略。`allow-exclusive`允许客户端通过放弃其他连接来请求独占访问。并行连接多个客户端需要所有客户端请求共享会话(vncviewer：-Shared开关)。这是默认值。`force-shared`禁用独占客户端访问，每个连接都必须为vncviewer指定-Shared开关。ignore无条件欢迎每个连接，自1.0.6起。

​		QEMU支持一个`socket`属性，而不是使用listen/port来监听unix域socket路径(自0.8.8版本起)。

​		对于VNC WebSocket功能，可以使用`websocket`属性来指定要监听的端口(-1表示自动分配，`autoport`由于安全原因无效)(自1.0.6版本起)。

​		对于VNC，可以使用`powerControl`属性来启用VNC客户端的VM关闭、重启和重置电源控制功能。如果经过身份验证的VNC客户端用户已经在客户机中具有管理员权限，那么这是合适的(自7.1.0起)。

​	尽管VNC本身不支持OpenGL，但可以与graphics类型`egl-headless`(参见下文)配对使用，这将指示QEMU打开并使用用于OpenGL渲染的drm节点。

​	还可以选择将VNC服务器映射到特定的主机音频后端，使用<audio>子元素：

```xml
<graphics type='vnc' ...>
  <audio id='1'>
</graphics>
```

​	其中1是音频设备的id(请参见“Audio backends”)。如果未指定ID，则将使用默认的音频后端。自7.2.0开始，qemu。

​	`SPICE` 自0.8.6以来

​		 启动一个SPICE服务器。`port`属性指定TCP端口号(-1是旧的语法，表示应该自动分配)，而`tlsPort`属性给出了另一种安全端口号。`autoport`属性是指示自动分配所需端口号的新首选语法。`passwd`属性以明文提供SPICE密码。如果`passwd`属性设置为空字符串，则禁用SPICE访问。`keymap`属性指定要使用的键映射。可以通过给定一个时间戳`passwdValidTo='2010-04-09T15:51:00'`来限制密码的有效性，假设该时间戳在UTC时间中。

​		`connected`属性允许在更改密码时控制连接的客户端。SPICE接受`keep`以保持客户端连接，`disconnect`以断开客户端连接，`fail`以在更改密码时失败。注意，这可能不被所有的hypervisor支持(自0.9.3以来)。

​		`defaultMode`属性设置默认的通道安全策略，有效值包括`secure、insecure`和默认值`any`(如果可能的话是安全的，但如果没有安全路径可用，则回退到不安全而不是出错)。自0.9.12起。

​		当SPICE同时配置了普通和TLS安全的TCP端口时，限制可以在每个端口上运行哪些通道是可取的。这可以通过在主要的<graphics>元素内添加一个或多个<channel>元素并将`mode`属性设置为`secure`或`insecure`来实现。设置`mode`属性会覆盖由`defaultMode`属性设置的默认值。(请注意，将`mode`设置为`any`会丢弃该条目，因为通道将继承默认模式)有效的通道名称包括`main、display、inputs、cursor、playback、record`(自0.8.6以来)；`smartcard`(自0.8.8以来)；和`usbredir`(自0.9.12以来)。

```xml
<graphics type='spice' port='-1' tlsPort='-1' autoport='yes'>
  <channel name='main' mode='secure'/>
  <channel name='record' mode='insecure'/>
  <image compression='auto_glz'/>
  <streaming mode='filter'/>
  <clipboard copypaste='no'/>
  <mouse mode='client'/>
  <filetransfer enable='no'/>
  <gl enable='yes' rendernode='/dev/dri/by-path/pci-0000:00:02.0-render'/>
</graphics>
```

​		Spice支持音频、图像和流媒体的可变压缩设置。这些设置可以通过以下所有元素中的`compression`属性进行访问：`image`用于设置图像压缩(接受`auto_glz、auto_lz、quic、glz、lz、off`)、`jpeg`用于在广域网上进行图像的JPEG压缩(接受`auto、never、always`)、`zlib`用于配置广域网图像压缩(接受`auto、never、always`)和playback用于启用音频流压缩(接受on或off)。自0.9.1起

​		流媒体模式由`streaming`元素设置，通过将其`mode`属性设置为`filter`、`all`或`off`中的一个来设置。自0.9.2起

​		通过`clipboard`元素设置剪贴板元素的复制和粘贴功能(通过Spice代理)。默认情况下启用，可以通过将`copypaste`属性设置为no来禁用。自0.9.3起

​		通过`mouse`元素设置鼠标模式，通过将其`mode`属性设置为`server`或`client`中的一个来设置。如果未指定模式，则将使用qemu默认模式(client模式)。自0.9.11起

​		使用`filetransfer`元素设置文件传输功能(通过Spice代理)。默认情况下启用，可以通过将`enable`属性设置为`no`来禁用。自1.2.2起

​		Spice可以提供具有OpenGL的服务器端加速渲染。 可以使用`gl`元素显式启用或禁用OpenGL支持，通过设置`enable`属性。(仅限QEMU，自1.3.3起)。请注意，这仅在本地工作，因为这需要使用UNIXsocket，即使用`listen`类型'socket'或'none'。要进行远程支持的加速OpenGL，请考虑将此元素与类型`egl-headless`配对(见下文)。但是，与本机Spice OpenGL支持相比，这将提供较弱的性能。

​		默认情况下，QEMU将选择第一个可用的GPU DRM渲染节点。 可以指定要使用的DRM渲染节点路径。 (仅限QEMU，自3.1.0起)。

​	`RDP`

​		 启动一个RDP服务器。port属性指定TCP端口号(-1表示传统语法，表示应自动分配端口)。autoport属性是指示自动分配要使用的TCP端口的新首选语法。在VirtualBox驱动程序中，当启动VM时，autoport将使虚拟机选择3389-3689范围内可用的端口。选择的端口将反映在port属性中。multiUser属性是一个布尔值，用于决定是否允许对VM进行多个同时连接。replaceUser属性是一个布尔值，用于决定是否在新客户端以单连接模式连接时，必须断开现有连接并由VRDP服务器建立新连接。

​	`desktop`

​		这个值目前为止仅供VirtualBox域保留。它在主机桌面上显示一个窗口，类似于"sdl"，但使用VirtualBox查看器。就像"sdl"一样，它接受可选的`display`和`fullscreen`属性。

​	`egl-headless`自4.6.0起

​		这种显示类型提供了对OpenGL加速显示的支持，可以在本地和远程访问(相比之下，Spice的本机OpenGL支持目前只能在本地使用UNIXsocket，但性能更好)。由于这种显示类型不提供窗口或图形控制台，出于实际原因，它应与`vnc`或`spice`图形类型配对使用。此显示类型仅由QEMU域支持(需要QEMU 2.10或更新版本)。从5.0.0版本开始，此元素接受一个带有可选属性`rendernode`的<gl/>子元素，该属性可用于指定用于OpenGL渲染的主机DRI设备的绝对路径。

```xml
<graphics type='spice' autoport='yes'/>
<graphics type='egl-headless'>
  <gl rendernode='/dev/dri/renderD128'/>
</graphics>
```

​	`dbusSince 8.4.0`

​		通过D-Bus导出显示。默认情况下，它将使用专用总线，除非指定了p2p或地址。

```xml
<graphics type='dbus'/>
```

​		`p2p`(接受值`on`或`off`)支持通过virDomainOpenGraphics()API建立的对等连接。

​		`address`(接受[D-Bus address](https://dbus.freedesktop.org/doc/dbus-specification.html#addresses))，将连接到指定的总线地址。

​		此元素接受具有可选属性`rendernode`的`<gl/>`子元素，该属性可用于指定要用于OpenGL渲染的主机DRI设备的绝对路径。

​		由于QEMU剪贴板管理器和SPICE `vdagent`协议，提供了复制和粘贴功能。有关更多详细信息，请参阅`qemu vdagent`。

​		D-Bus可以使用<audio>子元素导出音频后端：

```xml
<graphics type='dbus' ...>
  <audio id='1'>
</graphics>
```

​		其中1是音频设备的id(请参阅1.20.18 [Audio backends](https://libvirt.org/formatdomain.html#audio-backends))。

图形设备使用<listen>来设置设备应监听客户端的位置。它有一个指定侦听类型的强制属性类型。只有`vnc、spice`和`rdp`支持<listen>元素。自0.9.4.起。可用类型有：

​	`address` 

​		告诉图形设备使用在`address`属性中指定的地址，该属性将包含一个IP地址或主机名(将通过DNS查询解析为IP地址)以进行监听。

​		也可以省略`address`属性，以便从配置文件中使用地址。自1.3.5起。

​		`address`属性在`graphics`元素中作为`listen`属性的重复项，用于向后兼容。如果两者都提供，它们必须相等。

​	`network` 

​		用于指定在libvirt的已配置网络列表中的`network`属性的现有网络。将检查命名网络配置以确定适当的监听地址，并将该地址存储在地址属性中。例如，如果网络配置中有一个IPv4地址(例如，如果它具有路由、NAT或没有前向类型(隔离)的前向类型)，则将使用网络配置中列出的第一个IPv4地址。如果网络描述主机桥接，将使用与该桥接设备关联的第一个IPv4地址，如果网络描述“direct”(macvtap)模式之一，将使用第一个前向设备的第一个IPv4地址。

​	`socket` 自2.0.0起(仅适用于QEMU) 

​		这种监听类型告诉图形服务器监听Unixsocket。`socket`属性包含Unixsocket的路径。如果省略此属性，libvirt将为 生成此路径。支持vnc和spice图形类型。

​		为了向后兼容vnc图形，第一个`listen`元素的`socket`属性被复制为`graphics`元素中的`socket`属性。如果`graphics`元素包含`socket`属性，则将忽略所有`listen`元素。

​	`none` 自2.0.0起(仅适用于QEMU) 

​		此监听类型没有其他属性。Libvirt支持通过API `virDomainOpenGraphics()` 和 `virDomainOpenGraphicsFD()` 传递文件描述符。如果使用此类型并且图形设备不监听任何地方，则不允许使用其他监听类型。 需要使用这两个API之一将FD传递给QEMU，以连接到此图形设备。支持`vnc`和`spice`图形类型。



#### 1.20.15 (Video devices)

```xml
...
<devices>
  <video>
    <model type='vga' vram='16384' heads='1'>
      <acceleration accel3d='yes' accel2d='yes'/>
    </model>
    <driver name='qemu'/>
  </video>
</devices>
...
```

​	`video `

​		`video`元素是描述视频设备的容器。为了向后兼容，如果没有设置`video`元素但在域XML中存在`graphics`元素，那么libvirt将根据客户类型添加默认的video设备。

​		对于类型为"kvm"的客户，默认的video设备是：`type`属性值为"cirrus"，`vram`属性值为"16384"，heads属性值为"1"。默认情况下，域XML中的第一个video设备是主设备，但是可以使用可选的primary属性(自1.0.2起)并将其值设置为'yes'，以标记多个video设备的主设备。非主设备必须是类型为"qxl"或(自2.4.0起)"virtio"。

​	`model` 

​		`model`元素具有一个强制的`type`属性，该属性的值可以是"vga"、"cirrus"、"vmvga"、"xen"、"vbox"、"qxl"(自0.8.6起)、"virtio"(自1.3.0起)、"gop"(自3.2.0起)、"bochs"(自5.6.0起)、"ramfb"(自5.9.0起)或"none"(自4.6.0起)，具体取决于可用的hypervisor功能。

​		注意：`type`为`none`的目的是指示libvirt不要在客户中添加默认的video设备(请参阅上述`video`元素描述)，因为在GPU中介设备打算是客户中唯一的渲染设备的情况下，这种行为是不方便的。如果这是 的用例，请在XML中指定一个`none`类型的`video`设备以停止默认行为。请参阅 1.20.8 [Host device assignment](https://libvirt.org/formatdomain.html#host-device-assignment)以了解如何将中介设备添加到客户中。

​		 可以使用`vram`以kibibytes(1024字节的块)提供视频内存的大小。这仅支持客户类型为"vz"、"qemu"、"kvm"、"hvf"、"vbox"、"vmx"和"xen"。如果没有提供值，将使用默认值。如果大小不是2的幂，则会四舍五入到最接近的幂。

​		可以使用`heads`设置屏幕的数量。这仅适用于客户类型为"vz"、"kvm"、"hvf"、"vbox"和"vmx"的客户。

​		对于客户类型为"kvm"、"hvf"或"qemu"且型号类型为"qxl"的客户，还有一些可选的属性。ram属性(自1.0.2起)指定主要BAR的大小，而`vram`属性指定次要BAR的大小。如果未提供ram或vram，则将使用默认值。`ram`和`vram`也应四舍五入为2的幂。还有可选的`vgamem`属性(自1.2.11起)，用于设置QXL设备的回退模式的VGA帧缓冲区大小。`vram64`属性(自1.3.3起)扩展了次要BAR，并使其可寻址为64位内存。

​		自9.2.0起(仅适用于QEMU驱动程序)，类型为"virtio"的设备具有可选的`blob`属性，可以设置为"on"或"off"。将blob设置为"on"将启用设备中blob资源的使用。这可以通过减少或消除在客户和主机之间复制像素数据来加速显示路径。请注意，blob资源支持需要QEMU版本6.1或更高版本。

​		自5.9.0起，`model`元素还可以具有可选的resolution子元素。`resolution`元素具有`x`和`y`属性，用于设置video设备的最小分辨率。此子元素对于model类型为"vga"、"qxl"、"bochs"、"gop"和"virtio"的设备是有效的。

`acceleration`

​	配置是否应启用视频加速。

​		`accel2d`

​			启用2D加速(仅适用于vbox驱动程序，自0.7.1起)

​		`accel3d`

​			启用3D加速(自0.7.1起用于vbox驱动程序，自1.3.0起用于qemu驱动程序)

​		`rendernode`

​			用于渲染的主机DRI设备的绝对路径(自5.8.0起，仅适用于“vhostuser”驱动程序)。如果没有指定，libvirt将选择一个可用的。

`address`

​	可选的`address`子元素可以用于将视频设备连接到特定的PCI插槽。在S390上，地址可用于为视频设备提供CCW地址(自4.2.0起)。

`driver`

​	子元素驱动程序可用于调整设备：

​	`name`

​		指定要使用的后端驱动程序，“qemu”或“vhostuser”，具体取决于可用的系统hypervisor功能(自5.8.0起)。“qemu”是默认的qemu后端。“vhostuser”将使用一个单独的vhostuser进程后端(用于virtio设备)。

​	virtio选项

​		还可以设置[Virtio-related options](https://libvirt.org/formatdomain.html#virtio-related-options) (自3.5.0起)

​	VGA configuration

​		使用`vgaconf`属性控制视频设备如何暴露给 guest，该属性的值为“io”、“on”或“off”。目前，它只适用于bhyve的“gop”视频型号(自3.5.0起)



#### 1.20.16 (Consoles， serial， parallel & channel device)

字符设备提供了一种与虚拟机交互的方式。半虚拟化控制台、串行端口、并行端口和通道都被归类为字符设备，因此使用相同的语法表示

```xml
...
<devices>
  <parallel type='pty'>
    <source path='/dev/pts/2'/>
    <target port='0'/>
  </parallel>
  <serial type='pty'>
    <source path='/dev/pts/3'/>
    <target port='0'/>
  </serial>
  <serial type='file'>
    <source path='/tmp/file' append='on'>
      <seclabel model='dac' relabel='no'/>
    </source>
    <target port='0'/>
  </serial>
  <console type='pty'>
    <source path='/dev/pts/4'/>
    <target port='0'/>
  </console>
  <channel type='unix'>
    <source mode='bind' path='/tmp/guestfwd'/>
    <target type='guestfwd' address='10.0.2.1' port='4600'/>
  </channel>
</devices>
...
```

在这些指令中，顶层元素的名称(parallel、serial、console、channel)描述了设备如何呈现给虚拟机。虚拟机接口由`target`元素配置。

呈现给主机的接口在顶层元素的`type`属性中指定。主机接口由`source`元素配置。

`source`元素可以包含一个可选的`seclabel`，用于覆盖socket路径上的标签设置方式。如果此元素不存在，则[Security label](https://libvirt.org/formatdomain.html#security-label) 从每个域的设置中继承。

如果呈现给主机的接口类型是"file"，那么`source`元素可以包含一个可选的`append`属性，用于指定在域重新启动时是否应保留文件中的信息。允许的值为"on"和"off"(默认为"off")。自1.3.1起。

无论类型如何，字符设备可以关联一个可选的日志文件。这通过一个log子元素来表示，其中包含一个`file`属性。还可以有一个`append`属性，它采用上述相同的值。自1.3.3起。

```xml
...
<log file="/var/log/libvirt/qemu/guestname-serial0.log" append="off"/>
...
```

每个字符设备元素都有一个可选的子元素`<address>`，它可以将设备连接到特定的控制器(请参阅 1.20.6 [Controllers](https://libvirt.org/formatdomain.html#controllers))或PCI插槽。

对于类型为`unix`或`tcp`的字符设备，源有一个可选的元素`reconnect`，用于在连接丢失时配置重新连接超时。有两个属性，在可能的值为“yes”和“no”的情况下启用，`timeout`以秒为单位。`reconnect`属性仅对`connect`模式有效。自3.7.0起(仅限QEMU驱动程序)。



##### 	1.20.16.1 (Guest interface)

字符设备以以下类型之一的形式呈现给 guest。

**Parallel port**

```xml
...
<devices>
  <parallel type='pty'>
    <source path='/dev/pts/2'/>
    <target port='0'/>
  </parallel>
</devices>
...
```

`target`可以有一个`port`属性，用于指定端口号。端口编号从0开始。通常有0个、1个或2个并行端口。

**Serial port**

```xml
...
<devices>
  <!-- Serial port -->
  <serial type='pty'>
    <source path='/dev/pts/3'/>
    <target port='0'/>
  </serial>
  <!-- Debug port for SeaBIOS / EDK II -->
  <serial type='file'>
    <target type='isa-debug'/>
    <address type='isa' iobase='0x402'/>
    <source path='/tmp/DOMAIN-ovmf.log'/>
  </serial>

</devices>
...
```

```xml
...
<devices>
  <!-- USB serial port -->
  <serial type='pty'>
    <target type='usb-serial' port='0'>
      <model name='usb-serial'/>
    </target>
    <address type='usb' bus='0' port='1'/>
  </serial>
</devices>
...
```

`target`元素可以具有一个可选的`port`属性，用于指定端口号(从0开始)，以及一个可选的`type`属性。有效的值有，自1.0.2起，`isa-serial`(可用于x86虚拟机)，`usb-serial`(只要支持USB)，`pci-serial`(只要支持PCI)；自3.10.0起，还可以使用`spapr-vio-serial`(用于ppc64/pseries虚拟机)，`system-serial`(用于aarch64/virt和自4.7.0起的riscv/virt虚拟机)，`sclp-serial`(用于s390和s390x虚拟机)，自8.1.0起，`isa-debug`(用于x86虚拟机)。

自3.10.0起，`target`元素可以具有一个可选的`model`子元素；其`name`属性的有效值有：`isa-serial`(可用于`isa-serial`目标类型)；`usb-serial`(可用于`usb-serial`目标类型)；`pci-serial`(可用于`pci-serial`目标类型)；`spapr-vty`(可用于spapr-vio-serial目标类型)；pl011和自4.7.0起的16550a(可用于system-serial目标类型)；`sclpconsole`和`sclplmconsole`(可用于`sclp-serial`目标类型)。自8.1.0起，`isa-debugcon`(可用于`isa-debug`目标类型)；提供了一个虚拟控制台，用于接收来自x86平台固件的调试消息。通常不需要提供目标型号：libvirt会自动选择适用于所选目标类型的值，通常不建议覆盖该值。

如果用户未指定任何属性，libvirt将选择适用于大多数用户的值。

大多数目标类型支持配置虚拟机可见的设备地址，如“设备地址”部分所述；更具体地说，可接受的地址类型有`isa`(用于`isa-serial`)、`usb`(用于`usb-serial`)、`pci`(用于`pci-serial`)和`spapr-vio`(用于`spapr-vio-serial`)。`system-serial`和`sclp-serial`目标类型不支持指定地址。

有关串行端口和控制台之间的关系，请参阅 1.20.16.1 [Relationship between serial ports and consoles](https://libvirt.org/formatdomain.html#relationship-between-serial-ports-and-consoles)。

**Console**

```xml
...
<devices>
  <!-- Serial console -->
  <console type='pty'>
    <source path='/dev/pts/2'/>
   <target type='serial' port='0'/>
  </console>
</devices>
...
```

```xml
...
<devices>
  <!-- KVM virtio console -->
  <console type='pty'>
    <source path='/dev/pts/5'/>
    <target type='virtio' port='0'/>
  </console>
</devices>
...
```

`console`元素用于表示交互式串行控制台。根据正在使用的虚拟机类型和配置的具体情况，`console`元素可能表示与现有的`serial`元素相同的设备，也可能表示一个单独的设备。

支持`target`子元素，并且与`serial`元素的工作方式相同(有关详细信息，请参见1.20.16.1 [Serial port](https://libvirt.org/formatdomain.html#serial-port))。`type`属性的有效值包括：`serial`(如下所述)；`virtio`(只要支持VirtIO)；`xen`、`lxc`和`openvz`(在使用相应的hypervisor时可用)。出于兼容性原因，`sclp`和`sclplm`(适用于s390和s390x QEMU虚拟机)也受支持，但不应用于新的虚拟机：相反，应使用带有`serial`元素的`sclpconsole`和`sclplmconsole`目标型号。

在上面列出的目标类型中，`serial`是特殊的，因为它不代表一个单独的设备，而是与第一个`serial`元素相同的设备。因此，每个虚拟机只能有一个带有目标类型为`serial`的`console`元素。

Virtio控制台通常从虚拟机内部作为`/dev/hvc[0-7]`访问；有关更多信息，请参阅https://fedoraproject.org/wiki/Features/VirtioSerial。自0.8.3起

有关串行端口和控制台之间的关系，请参阅1.20.16.1[Relationship between serial ports and consoles](https://libvirt.org/formatdomain.html#id78)

**Relationship between serial ports and consoles**

串行端口和控制台之间的关系 由于历史原因，`serial`和`console`元素在一定程度上有重叠的范围。

总的来说，这两个元素都用于配置一个或多个串行控制台，用于与虚拟机进行交互。它们之间的主要区别在于，`serial`用于模拟的、通常是本机的串行控制台，而`console`用于半虚拟化的串行控制台。

模拟和半虚拟化的串行控制台都有各自的优缺点：

​	模拟的串行控制台通常比半虚拟化的串行控制台早初始化，因此可以用于控制引导加载程序并显示固件和早期启动消息；

​	在多个平台上，每个虚拟机只能有一个模拟的串行控制台，但半虚拟化控制台不受相同的限制。

例如以下配置：

```xml
...
<devices>
  <console type='pty'>
    <target type='serial'/>
  </console>
  <console type='pty'>
    <target type='virtio'/>
  </console>
</devices>
...
```

将在任何平台上工作，并将产生一个模拟串行控制台，用于  early boot logging / interactive / recovery ，以及一个半虚拟化串行控制台，例如用作侧通道。大多数人都可以在配置中只使用第一个`console`元素，但如果需要特定的配置，则应该指定这两个元素。

请注意，由于前面提到的兼容性问题，以下所有配置：

```xml
...
<devices>
  <serial type='pty'/>
</devices>
...
```

```xml
...
<devices>
  <console type='pty'/>
</devices>
...
```

```xml
...
<devices>
  <serial type='pty'/>
  <console type='pty'/>
</devices>
...
```

将被同等对待，并将导致单个模拟串行控制台可供guest使用。

**Channel**

这表示主机和guest之间的专用通信通道。

```xml
...
<devices>
  <channel type='unix'>
    <source mode='bind' path='/tmp/guestfwd'/>
    <target type='guestfwd' address='10.0.2.1' port='4600'/>
  </channel>

  <!-- KVM virtio channel -->
  <channel type='pty'>
    <target type='virtio' name='arbitrary.virtio.serial.port.name'/>
  </channel>
  <channel type='unix'>
    <source mode='bind' path='/var/lib/libvirt/qemu/f16x86_64.agent'/>
    <target type='virtio' name='org.qemu.guest_agent.0' state='connected'/>
  </channel>
  <channel type='spicevmc'>
    <target type='virtio' name='com.redhat.spice.0'/>
  </channel>
</devices>
...
```

这可以通过多种方式来实现。通道的具体类型在`target`元素的`type`属性中给出。不同的通道类型具有不同的`target`属性。

​	`guestfwd`

​		将由客户机发送到指定IP地址和端口的TCP流量转发到主机上的通道设备。目标元素必须具有`address`和`port`属性。自版本0.7.3起可用。

​	`virtio Para`

​		虚拟化的virtio通道。通道在客户机中以/dev/vport*的形式公开，如果指定了可选元素`name`，则可以在/dev/virtio-ports/$name中访问(有关详细信息，请参阅https://fedoraproject.org/wiki/Features/VirtioSerial)。可选的元素地址可以将通道绑定到特定的`type='virtio-serial'`控制器，如在“1.20.3 [Device Addresses](https://libvirt.org/formatdomain.html#device-addresses) ”部分中所述。对于QEMU，如果`name`为“org.qemu.guest_agent.0”，则libvirt可以与安装在客户机中的客户机代理进行交互，执行诸如客户机关闭或文件系统静默操作之类的操作。自版本0.7.7以来，客户机代理互动自版本0.9.10以来。此外，自版本1.0.6以来，可以自动生成virtio Unix通道的源路径。在QEMU客户代理的情况下，这非常有用，因为用户通常不关心源路径，因为是libvirt与客户机代理通信。如果用户想要使用此功能，他们应该省略<source>元素。自版本1.2.11以来，virtio通道的活动XML可以包含一个可选的`state`属性，反映了客户机上的一个进程是否在通道上活动。此属性仅用于输出。状态属性的可能值为`connected`和`disconnected`。

​	`xen `

​		半虚拟化的Xen通道。通道在客户机中显示为Xen控制台，但具有名称。设置和使用Xen通道取决于客户机中的软件和配置。有关更多信息，请参阅xen-pv-channel(7)手册页。通道源路径语义与virtio目标类型相同。不支持`state`属性，因为Xen通道缺少必要的探测机制。自版本2.3.0以来。

​	`spicevmc`

​		 半虚拟化的SPICE通道。域还必须具有SPICE服务器作为图形设备(请参阅1.20.14 [Graphical framebuffers](https://libvirt.org/formatdomain.html#graphical-framebuffers))，在这种情况下，主机通过`main`通道传递消息。必须存在`target`元素，属性为`type='virtio'`；可选的属性`name`控制客户机将如何访问通道，默认为`name='com.redhat.spice.0'`。可选的`address`元素可以将通道绑定到特定的`type='virtio-serial'`控制器。自版本0.8.8以来。

​	`qemu-vdagent `

​		半虚拟化的qemu vdagent通道。此通道实现了SPICE vdagent协议，但由qemu内部处理，因此不需要SPICE图形设备。与spicevmc通道类似，`target`元素必须存在，属性为`type='virtio'`；可选的属性`name`控制客户机将如何访问通道，默认为`name='com.redhat.spice.0'`。可选的`address`元素可以将通道绑定到特定的`type='virtio-serial'`控制器。可以使用`source`元素启用或禁用某些vdagent协议功能。

​		通过设置`copypaste`属性来设置复制和粘贴功能，默认情况下处于禁用状态，可以通过将`copypaste`属性设置为`yes`来启用。这允许客户机的剪贴板与qemu剪贴板管理器同步，从而在使用VNC图形设备(请参阅 1.20.14 [Graphical framebuffers](https://libvirt.org/formatdomain.html#graphical-framebuffers))或其他支持qemu剪贴板管理器的图形类型时，在客户机和客户端之间进行复制和粘贴。

​		鼠标模式由`mouse`元素设置，将其`mode`属性设置为`server`或`client`。如果未指定模式，则将使用qemu默认值(客户端模式)。自版本8.4.0以来。



##### 	1.20.16.2 (Host interface)

字符设备以以下类型之一的形式呈现给主机。

**Domain logfile**

```xml
...
<devices>
  <console type='stdio'>
    <target port='1'/>
  </console>
</devices>
...
```

**Device logfile**

```xml
...
<devices>
  <serial type="file">
    <source path="/var/log/vm/vm-serial.log"/>
    <target port="1"/>
  </serial>
</devices>
...
```

**Virtual console**

将字符设备连接到虚拟控制台中的图形帧缓冲区。这通常通过一个特殊的热键序列来访问，如“ctrl+alt+3”

```xml
...
<devices>
  <serial type='vc'>
    <target port="1"/>
  </serial>
</devices>
...
```

**Null device**

将字符设备连接到空白。从未向输入端提供任何数据。所有写入的数据都将被丢弃。

```xml
...
<devices>
  <serial type='null'>
    <target port="1"/>
  </serial>
</devices>
...
```

**Pseudo TTY** 

Pseudo TTY是使用/dev/ptmx分配的。一个合适的客户端，如“virsh console”，可以连接到本地与串行端口进行交互。

```xml
...
<devices>
  <serial type="pty">
    <source path="/dev/pts/3"/>
    <target port="1"/>
  </serial>
</devices>
...
```

注意特殊情况，如果<console type＝'pty'>，则TTY路径也被复制为顶级<console>标记上的属性TTY＝'/dev/pts/3'。这为<console>标记提供了与现有语法的兼容性。

**Host device proxy**

字符设备被传递到底层物理字符设备。设备类型必须匹配，例如模拟串行端口应仅连接到主机串行端口，而不要将串行端口连接到并行端口。

```xml
...
<devices>
  <serial type="dev">
    <source path="/dev/ttyS0"/>
    <target port="1"/>
  </serial>
</devices>
...
```

**Named pipe**

字符设备将输出写入named pipe。有关详细信息，请参见pipe(7)。

```
...
<devices>
  <serial type="pipe">
    <source path="/tmp/mypipe"/>
    <target port="1"/>
  </serial>
</devices>
...
```

**TCP client / server**

字符设备充当连接到远程服务器的TCP客户端。

```xml
...
<devices>
  <serial type="tcp">
    <source mode="connect" host="0.0.0.0" service="2445"/>
    <protocol type="raw"/>
    <target port="1"/>
  </serial>
</devices>
 ...
```

或者作为TCP服务器等待客户端连接。

```xml
...
<devices>
  <serial type="tcp">
    <source mode="bind" host="127.0.0.1" service="2445"/>
    <protocol type="raw"/>
    <target port="1"/>
  </serial>
</devices>
...
```

或者， 可以使用`telnet`而不是`raw`TCP，以便使用telnet协议进行连接。

自0.8.5以来，一些hypervisor支持使用`telnet`(secure telnet)或`tls`(通过安全socket层)作为连接的传输协议。

```xml
...
<devices>
  <serial type="tcp">
    <source mode="connect" host="0.0.0.0" service="2445"/>
    <protocol type="telnet"/>
    <target port="1"/>
  </serial>
  ...
  <serial type="tcp">
    <source mode="bind" host="127.0.0.1" service="2445"/>
    <protocol type="telnet"/>
    <target port="1"/>
  </serial>
</devices>
...
```

自2.4.0起，可选属性`tls`可用于控制chardev TCP通信通道是否会使用hypervisor配置的tls X.509证书环境来加密数据通道。对于QEMUhypervisor，TLS环境的使用可以通过文件/etc/libvirt/QEMU.conf中的`chardev_TLS`和`chardev_TLS_x509_cert_dir`或`default_TLS_x509 _cert_dar`设置在主机上进行控制。如果启用了`chardev_TLS`，则除非`TLS`属性设置为“no”，否则libvirt将使用主机配置的TLS环境。如果禁用了`chardev_tls`，但`tls`属性设置为“yes”，则如果存在`chardev_tls_x509_cert_dir`或`default_tls_x509 _cert_dirTLS`目录结构，则libvirt将尝试使用主机tls环境。

```xml
...
<devices>
  <serial type="tcp">
    <source mode='connect' host="127.0.0.1" service="5555" tls="yes"/>
    <protocol type="raw"/>
    <target port="0"/>
  </serial>
</devices>
...
```

**UDP network console**

字符设备充当UDP网络控制台服务，发送和接收数据包。这是一项有损服务。

```xml
...
<devices>
  <serial type="udp">
    <source mode="bind" host="0.0.0.0" service="2445"/>
    <source mode="connect" host="0.0.0.0" service="2445"/>
    <target port="1"/>
  </serial>
</devices>
...
```

**UNIX domain socket client/server**

字符设备充当UNIX域socket服务器，接受来自本地客户端的连接

**Spice channel**

可以通过`channel`属性中指定的通道名称下的spice连接访问字符设备。自1.2.2

注意：根据hypervisor的不同，在具有或不具有spice图形的域上可能启用(也可能不启用)spicereports(请参阅1.20.14[Graphical framebuffers)](https://libvirt.org/formatdomain.html#graphical-framebuffers)

```xml
...
<devices>
  <serial type="spiceport">
    <source channel="org.qemu.console.serial.0"/>
    <target port="1"/>
  </serial>
</devices>
...
```

**Nmdm device**

FreeBSD上提供的nmdm设备驱动程序提供了通过虚拟零调制解调器电缆连接在一起的两个tty设备。自1.2.4

```xml
...
<devices>
  <serial type="nmdm">
    <source master="/dev/nmdm0A" slave="/dev/nmdm0B"/>
  </serial>
</devices>
...
```

`source`元素具有以下属性：

​	`master`

​		传递给hypervisor的对的主设备。设备由完全限定的路径指定。

​	`slave`

​		该对的从属设备，传递给客户端以连接到客户控制台。设备由完全限定的路径指定。



#### 1.20.17 (Sound devices)

#### 1.20.18 (Audio backends)

##### 	1.20.18.1 (None audio backend)

##### 	1.20.18.2 (ALSA audio backend)

##### 1.20.18.3 (Coreaudio audio backend)

##### 1.20.18.4 (D-Bus audio backend)

##### 1.20.18.5 (Jack audio backend)

##### 1.20.18.6 (OSS audio backend)

##### 1.20.18.7 (Pulse audio backend)

##### 1.20.18.8 (SDL audio backend)

##### 1.20.18.9 (Spice audio backend)

##### 1.20.18.10 (File audio backend)



#### 1.20.19 看门狗设备(Watchdog devices)

虚拟硬件看门狗设备可以通过`watchdog`元素添加到客户机。自0.7.3起，仅限QEMU和KVM

看门狗设备需要在客户机中添加一个额外的驱动程序和管理守护程序。仅仅在libvirt配置中启用看门狗本身并没有任何用处。

目前，libvirt不支持在看门狗启动时发出通知。此功能计划用于libvirt的未来版本。

拥有多个看门狗通常不是很常见，但要注意，例如，当一个隐含的看门狗设备作为另一个设备的一部分添加时，可能会发生这种情况。例如，iTCO看门狗是ich9南桥的一部分，与q35机型一起使用。自9.1.0

```xml
...
<devices>
  <watchdog model='i6300esb'/>
</devices>
...
```

```xml
  ...
  <devices>
    <watchdog model='i6300esb' action='poweroff'/>
  </devices>
</domain>
```

`model`

​	所需的`model`属性指定要模拟的实际看门狗设备。有效值特定于底层hypervisor。

​	QEMU和KVM支持：

​		 "itco"-默认情况下包含在q35机器类型中，自9.1.0起

​		"i6300esb"-推荐的设备，模拟PCI Intel 6300ESB

​		"ib700"-模拟ISA iBase ib700

​		"diag288"-自1.2.17起模拟S390 diag288设备

`action`

​	可选操作属性描述了看门狗过期时要采取的操作。有效值特定于底层hypervisor。

​	QEMU和KVM支持：

​		"重置"-默认值，强制重置客户机

​		"shutdown"-正常关闭客户机(不推荐)

​		"poweroff"-强制关闭客户机的电源

​		"暂停"-暂停客户机

​		"没有"-什么都不做

​		"dump"-自0.8.7起自动转储客户机

​		"inject-nmi"-自1.2.17起向客户机注入不可屏蔽的中断

注1："关机"操作要求客户机对ACPI信号作出响应。在看门狗过期的情况下，客户机通常无法对ACPI信号做出响应。因此，不建议使用"shutdown"。

注2：保存转储文件的目录可以通过文件/etc/libvirt/qemu.conf中的`auto_dump_path`进行配置。



#### 1.20.20 内存气球设备(Memory balloon device)

虚拟内存气球设备被添加到所有Xen和KVM/QEMU客户机。它将被视为`memballoop`元素。它将在适当的时候自动添加，因此无需在客户机XML中显式添加此元素，除非需要分配特定的PCI插槽。自0.8.3，仅Xen、QEMU和KVM。此外，自0.8.4，如果需要显式禁用memballood设备，则可以使用`model='none'`。

示例：使用KVM自动添加设备

```xml
...
<devices>
  <memballoon model='virtio'/>
</devices>
...
```

示例：手动添加的设备已请求静态PCI插槽2

```xml
 ...
  <devices>
    <memballoon model='virtio'>
      <address type='pci' domain='0x0000' bus='0x00' slot='0x02' function='0x0'/>
      <stats period='10'/>
      <driver iommu='on' ats='on'/>
    </memballoon>
  </devices>
</domain>
```

`model`

​	所需的`model`属性指定提供的引出序号设备的类型。有效值特定于虚拟化平台

​		"virtio"-QEMU/KVM的默认值

​		'virtio transitional'自5.2.0起

​		'virtio non-transitional'自5.2.0起

​		"xen"-xen的默认值

​	有关更多详细信息，请参阅1.20.5[Virtio transitional devices](https://libvirt.org/formatdomain.html#virtio-transitional-devices)

`autodeflate`

​	可选的autodeflate属性允许启用/禁用(分别为值"on"/"off")QEMU virtio内存气球在客户机进程被内存不足杀手杀死之前的最后时刻释放一些内存的能力。自1.3.1起，仅限QEMU和KVM

`freePageReporting`

​	可选的`freePageReporting`属性允许启用/禁用(分别为"on"/"off")QEMU virtio内存气球将未使用的页面返回到hypervisor以供其他客户机或进程使用的功能。请注意，尽管有其名称，但它对`virDomainMemoryStats()`和/或`virsh-dommemstat`报告的可用内存没有影响。自6.9.0起，仅限QEMU和KVM

`period`

​	可选`period`允许QEMU virtio内存气球驱动程序通过`virsh dommemstat` [domain]命令提供统计信息。默认情况下，不启用收集。为了启用，请使用`virsh dommemstat[domain]--period[number]`命令或`virsh edit`命令将该选项添加到XML定义中。`virsh-dommemstat`将接受选项`--live、--current`或`--config`。如果未提供选项，则只会对活动客户机更改正在运行的域。如果QEMU驱动程序的版本不正确，则设置周期的尝试将失败。较大的值(例如好几年)可能会被忽略。自1.1.1起，要求QEMU 1.5

`driver`

​	对于模型`virtio`内存气球，还可以设置与[Virtio-related options](https://libvirt.org/formatdomain.html#virtio-related-options)。(自3.5.0起)



#### 1.20.21 随机数生成器装置(Random number generator device)

虚拟随机数生成器设备允许主机将熵传递给guest OS。自1.0.3

示例：RNG设备的用法：

```xml
...
<devices>
  <rng model='virtio'>
    <rate period="2000" bytes="1234"/>
    <backend model='random'>/dev/random</backend>
    <!-- OR -->
    <backend model='egd' type='udp'>
      <source mode='bind' service='1234'/>
      <source mode='connect' host='1.2.3.4' service='1234'/>
    </backend>
    <!-- OR -->
    <backend model='builtin'/>
  </rng>
</devices>
...
```

`model`

​	所需的`model`属性指定提供的RNG设备的类型。有效值特定于虚拟化平台：

​		"virtio"-由qemu和virtio-rng内核模块支持

​		'virtio transitional'自5.2.0起

​		'virtio non-transitional'自5.2.0起

有关更多详细信息，请参阅1.20.5 [Virtio transitional devices](https://libvirt.org/formatdomain.html#virtio-transitional-devices)。

`rate`

​	可选的`rate`元素允许限制可以从源消耗熵的速率。强制属性`bytes`指定每个周期允许消耗的字节数。可选的`period`属性以毫秒为单位指定周期的持续时间；如果省略，则周期取为1000毫秒(1秒)。自1.0.4起

`backend`

​	`backend`元素指定要用于域的熵的来源。使用`model`属性配置源模型。支持的源模型包括：

​	`random`

​		此后端类型需要一个非阻塞字符设备作为输入。文件名被指定为`backend`元素的内容。自1.3.4起，可接受任何路径。在此之前，`/dev/random` 和 `/dev/hwrng` 是唯一可接受的路径。如果未指定文件名，则使用hypervisor默认值。对于QEMU，默认值是`/dev/random`。然而，推荐的熵源是`/dev/urandom`(因为它没有`/dev/random`的限制)。

​	`egd`

​		这个后端使用EGD协议连接到一个源。源被指定为字符设备。有关详细信息，请参阅主机界面。

​	`builtin`

​		这个后端使用qemu内置的随机生成器，该生成器使用`getrandom()`syscall作为熵的来源。(自6.1.0和QEMU 4.2起)

`driver`

​	子元素`driver`可用于调整设备：	

​	virtio选项

​		还可以设置[Virtio-related options](https://libvirt.org/formatdomain.html#virtio-related-options)。(自3.5.0起)



#### 1.20.22 (TPM device)Trusted Platform Module硬件安全模块

TPM设备使QEMU客户机能够访问TPM功能。TPM设备可以是TPM 1.2或者TPM 2.0。

TPM直通设备类型为一个QEMU客户机提供对主机TPM的访问。在QEMU客户机启动时，没有其他软件可以使用TPM设备，通常是/dev/tpm0。"从1.0.5开始

示例：TPM直通设备的使用

```xml
...
<devices>
  <tpm model='tpm-tis'>
    <backend type='passthrough'>
      <device path='/dev/tpm0'/>
    </backend>
  </tpm>
</devices>
...
```

模拟器设备类型提供对TPM模拟器的访问，TPM模拟器为每个VM提供TPM功能。QEMU通过Unix socket与它进行对话。使用模拟器设备类型，每个guest都有自己的专用TPM。"emulator"模拟器自4.5.0以来TPM模拟器的状态可以通过提供`encryption`元素来加密。"encryption"加密自5.6.0

```xml
...
<devices>
  <tpm model='tpm-tis'>
    <backend type='emulator' version='2.0'>
      <encryption secret='6dd3e4a5-1d76-44ce-961f-f119f5aad935'/>
      <active_pcr_banks>
          <sha256/>
      </active_pcr_banks>
    </backend>
  </tpm>
</devices>
...
```

`model`

​	`model`属性指定QEMU为客户机提供的设备模型。如果没有提供型号名称，将自动为非PPC64架构选择`tpm-tis`。自4.4.0以来，另一个可用的选择是`tpm-crb`，它只应在后端设备是tpm 2.0时使用。从6.1.0开始，支持PPC64上的pSeries客户机，默认为`tpm-spapr`。从6.5.0开始，为pSeries客户机添加了一个名为`spapr-tpm-proxy`的新模型。此模型仅适用于直通后端。它创建一个TPM代理设备，该设备与主机中现有的TPM资源管理器通信，例如`/dev/tpmrm0`，使客户机能够在Ultravisor的帮助下以安全虚拟机模式运行。向pSeries客户机添加TPM代理不会带来任何安全好处，除非客户机运行在具有Ultravisor和TPM资源管理器的PPC64主机上。每个客户机只允许使用一个TPM代理设备，但TPM代理设备可以与其他TPM设备一起添加。

`backend`

​	`backend`元素指定TPM设备的类型。支持以下类型：

​	`passthrough`

​		使用主机的TPM或TPM资源管理器设备。

​		此后端类型要求独占访问主机上的TPM设备。这种设备的一个例子是/dev/tpm0。完全限定的文件名由源元素的path属性指定。如果未指定文件名，则会自动使用/dev/tpm0。从6.5.0开始，在选择spapr tpm代理模型时，指定的文件名应为tpm资源管理器设备，例如/dev/tpmrm0。

​	`emulator`

​		对于此后端类型，主机上必须安装"swtpm"TPM Emulator。Libvirt将为每个请求访问它的QEMU客户机自动启动一个独立的TPM模拟器。

`version`

​	version属性指示TPM的版本。此属性仅适用于模拟器后端。支持以下版本：

​		"1.2"：创建TPM 1.2

​		"2.0"：创建TPM 2.0

​	使用的默认版本取决于hypervisor、客户体系结构、TPM模型和后端的组合。

`persistent_state`

​	`persistent_state`属性指示在瞬态域断电或未定义时是否保持"swtpm"TPM状态。此选项可用于保留TPM状态。默认情况下，值为`no`。此属性仅适用于`emulator`后端。可接受的值为`yes`和`no`。自7.0.0起

`active_pcr_banks`

​	`active_pcr_banks`节点用于定义要激活TPM 2.0的哪些PCR库。例如，有效名称为sha1、sha256、sha384和sha512。如果提供了此节点，则在每次启动VM之前激活一组PCR库，并将此步骤记录在swtpm的日志中。如果删除或省略了此节点，那么libvirt将不会在VM启动时修改活动的PCR库，而是将它们保留在最后的配置中。此属性要求安装swtpm_setup v0.7或更高版本，否则可能不会产生任何效果。PCR库的选择仅适用于`emulator`后端。自7.10.0

`encryption`

​	`encryption`元素允许对TPM模拟器的状态进行加密。`secret`必须引用一个机密对象，该对象包含将从中派生加密密钥的密码短语。

#### 1.20.23 (NVRAM device)

nvram设备总是添加到PPC64上的pSeries客户机，并且允许更改其地址。元素`nvram`(自1.0.5以来，仅对pSeries客户机有效)用于启用地址设置。

示例：NVRAM配置的使用

```xml
...
<devices>
  <nvram>
    <address type='spapr-vio' reg='0x00003000'/>
  </nvram>
</devices>
...
```

`spapr vio`

​	VIO设备地址类型，仅对PPC64有效。

`reg`

​	设备地址



#### 1.20.24 (panic device)

panic设备使libvirt能够接收来自QEMU客户机的panic通知。自1.2.1起，仅限QEMU和KVM

此功能始终为以下各项启用：

​		pSeries客户机，因为它是由客户机固件实现的

​		S390客户机，因为它是S390体系结构的组成部分

对于上面列出的guest类型，libvirt会自动向域XML添加一个panic元素。

示例：紧急配置的使用

```xml
...
<devices>
  <panic model='hyperv'/>
  <panic model='isa'>
    <address type='isa' iobase='0x505'/>
  </panic>
</devices>
...
```

`model`

​	可选`model`属性指定提供的紧急设备类型。缺少此属性时使用的panic模型取决于hypervisor和guest arch。

​		"isa"-用于isa pvpanic设备

​		"pseries"-默认值，仅对pseries客户机有效。

​		"hyperv"-用于Hyper-V崩溃CPU功能。自1.3.0起，仅限QEMU和KVM

​		"s390"-s390客户机的默认值。自1.3.5

​		"pvpanic"-用于PCI pvpanic设备自9.1.0起，仅限QEMU

`address`

​	panic的地址。默认ioport为0x505。大多数用户不需要指定地址，s390、pseries和hyperv模型完全禁止这样做。



#### 1.20.25 (Shared memory device)

共享存储器设备允许在不同的虚拟机和主机之间共享存储器区域。自1.2.10起，仅限QEMU和KVM

```xml
...
<devices>
  <shmem name='my_shmem0' role='peer'>
    <model type='ivshmem-plain'/>
    <size unit='M'>4</size>
  </shmem>
  <shmem name='shmem_server'>
    <model type='ivshmem-doorbell'/>
    <size unit='M'>2</size>
    <server path='/tmp/socket-shmem'/>
    <msi vectors='32' ioeventfd='on'/>
  </shmem>
</devices>
...
```

shmem

​	`shmem`元素有一个强制性属性`name`，用于标识共享内存。此属性不能是特定于的 . 或 .. 目录，它也不能涉及路径分隔符 / 

​	可选(自6.6.0起)`role`属性指定共享内存是否可迁移。该值可以是"master"或"peer"，前者意味着在迁移时，共享内存中的数据会随域一起迁移。每个共享内存对象应该只有一个"master"。已禁用具有"对等"角色的迁移。如果需要迁移此类域，则需要在迁移前拔下shmem设备，并在迁移成功后在目的地插入。如果未指定角色，则使用hypervisor默认值。该属性目前仅适用于`model`类型`ivshmem-plain`和`ivshmem-door`。

`model`

​	可选元素`model`的属性`type`指定提供`shmem`设备的底层设备的模型。目前支持的型号有`ivshmem`(同时支持服务器和无服务器的shmem，但被较新的QEMU弃用，转而支持-plain和-downer变体)、`ivshmem-plain`(仅适用于无服务器的shmem)和`ivshmem-downer`(仅适用于带服务器的shem)。

`size`

​	可选的`size`元素指定共享内存的大小。这必须是2的幂并且大于或等于1MiB。

`server`

​	可选的`server`元素可用于配置设备应该连接到的服务器socket。可选的path属性指定unix socket的绝对路径，默认为 `/var/lib/libvirt/shmem/$shmem-$name sock`。

`msi`

​	可选的`msi`元素启用/禁用(值分别为"on"/"off")msi中断。此选项当前只能与`server`元素一起使用。`vectors`矢量属性可用于指定中断矢量的数量。`ioeventd`属性启用/禁用(值分别为"on"/"off")ioeventfd。



#### 1.20.26 (Memory devices)

​	   除了分配给客户机的初始存储器之外，存储器设备还允许以存储器模块的形式将额外的存储器分配给客户机。根据客户机的内存资源需求，内存设备可以是热插拔的。某些hypervisor可能需要为客户机配置NUMA。

示例：内存设备的使用

```xml
...
<devices>
  <memory model='dimm' access='private' discard='yes'>
    <target>
      <size unit='KiB'>524287</size>
      <node>0</node>
    </target>
  </memory>
  <memory model='dimm'>
    <source>
      <pagesize unit='KiB'>2048</pagesize>
      <nodemask>1-3</nodemask>
    </source>
    <target>
      <size unit='KiB'>524287</size>
      <node>1</node>
    </target>
  </memory>
  <memory model='nvdimm'>
    <uuid>9066901e-c90a-46ad-8b55-c18868cf92ae</uuid>
    <source>
      <path>/tmp/nvdimm</path>
    </source>
    <target>
      <size unit='KiB'>524288</size>
      <node>1</node>
      <label>
        <size unit='KiB'>128</size>
      </label>
      <readonly/>
    </target>
  </memory>
  <memory model='nvdimm' access='shared'>
    <uuid>e39080c8-7f99-4b12-9c43-d80014e977b8</uuid>
    <source>
      <path>/dev/dax0.0</path>
      <alignsize unit='KiB'>2048</alignsize>
      <pmem/>
    </source>
    <target>
      <size unit='KiB'>524288</size>
      <node>1</node>
      <label>
        <size unit='KiB'>128</size>
      </label>
    </target>
  </memory>
  <memory model='virtio-pmem' access='shared'>
    <source>
      <path>/tmp/virtio_pmem</path>
    </source>
    <target>
      <size unit='KiB'>524288</size>
      <address base='0x140000000'/>
    </target>
  </memory>
  <memory model='virtio-mem'>
    <source>
      <nodemask>1-3</nodemask>
      <pagesize unit='KiB'>2048</pagesize>
    </source>
    <target>
      <size unit='KiB'>2097152</size>
      <node>0</node>
      <block unit='KiB'>2048</block>
      <requested unit='KiB'>1048576</requested>
      <current unit='KiB'>524288</current>
      <address base='0x150000000'/>
    </target>
  </memory>
  <memory model='sgx-epc'>
    <source>
      <nodemask>0-1</nodemask>
    </source>
    <target>
      <size unit='KiB'>16384</size>
      <node>0</node>
    </target>
  </memory>
  <memory model='sgx-epc'>
    <target>
      <size unit='KiB'>16384</size>
    </target>
  </memory>
</devices>
...
```

`model`

​	提供`dimm`以向客户机添加虚拟 DIMM 模块。自1.2.14起，提供添加非易失性DIMM模块的`nvdimm`模型。自3.2.0起，提供`virtio-pmem`模型来添加半虚拟化的持久内存设备。从7.1.0开始，提供virtiomem模型来添加半虚拟化的内存设备。自7.9.0起，提供`sgx-epc`模型，为客户机添加sgx飞地页面缓存(epc)内存。自8.10.0和QEMU 7.0.0

`access`

​	一种可选的属性`access`(自3.2.0起)，提供了在每个模块的基础上微调内存映射的能力。值与 [Memory Backing](https://libvirt.org/formatdomain.html#memory-backing)相同：`shared`和`private`。对于`nvdimm`模型，如果使用真实的NVDIMM DAX设备作为后端，则需要`shared`。对于virtio-pem模型需要`shared`。

`discard`

​	可选的属性`discard`(自4.4.0起)，可根据每个模块对数据的丢弃进行微调。可接受的值为`yes`和`no`。此处介绍了该功能： [Memory Backing](https://libvirt.org/formatdomain.html#memory-backing)。此属性仅适用于`model='dimm'`。

`uuid`

​	对于pSeries客户机，可以设置uuid来标识nvdimm模块。如果不存在，libvirt将自动生成一个uuid。此属性只允许用于pSeries客户机的`model='nvdim'`。自6.2.0

`source`

​	对于模型`dimm`和` virtio mem`，这个元素是可选的，可以微调用于给定内存设备的内存源。如果未提供元素，则使用通过`numatune`配置的默认值。如果提供了元素，则可以提供以下可选元素：

​	`pagesize`

​		此元素可用于覆盖用于支持内存设备的默认主机页大小。配置的值必须与主机支持的页面大小相对应。

​	`nodemask`

​		此元素可用于覆盖将分配内存的默认NUMA节点集。

​	对于模型`nvdimm`，`source`元素是必需的。强制子元素`path`表示主机中支持客户机中nvdimm模块的路径。可以使用以下可选元件：

​	`alignsize`

​		`alignsize`元素定义页面大小对齐方式，用于mmap后端`path`的地址范围。如果未提供，则使用主机页大小。例如，要对真正的NVDIMM设备进行mmap，可能需要2M对齐的页面，并且主机页面大小为4KB，那么我们需要将此元素设置为2MB。自5.0.0起

​	`pmem`

如果hypervisor支持并启用持久性内存以保证对vNVDIMM后端的持久性写入，则使用pmem元素以利用该功能。自5.0.0起

对于模型`virtio pmem`，`source`元素是必需的。可以使用以下可选元件：

​	`path`

​		表示主机中支持客户机中的virtio内存模块的路径。它是强制性的。

对于模型`sgx-epc`，此元素是可选的。可以使用以下可选元件：

​	`nodemask`

​		此元素可用于覆盖将分配内存的默认NUMA节点集。自8.10.0和QEMU 7.0.0

`target`

​	强制`target`元素从客户机的角度配置添加的内存的位置和大小。

​	强制`size`子元素将添加的内存的大小配置为缩放整数。对于`virtio-mem`，这表示暴露给客户机的最大可能大小。

​	`node`子元素配置客户机NUMA节点以将内存连接到。只有当客户机配置了NUMA节点时，才应使用该元素。

​	可以使用以下可选元素：

​	`label`

​		对于NVDIMM类型的设备，可以使用标签及其子元素大小来配置NVDIMM模块内的名称空间标签存储的大小。`size`元素具有在 [Memory Allocation](https://libvirt.org/formatdomain.html#memory-allocation) 小节部分中描述的通常含义。`label`对于pSeries客户机是强制性的，对于所有其他体系结构是可选的。对于QEMU域，适用以下限制：

​		1. 最小标签大小为128KiB，

​		2. 默认情况下，剩余的大小(总大小-标签大小)，也称为guest区域，将与4KiB对齐。对于pSeries客户机，guest区将向下对齐至256MiB，guest区的最小大小必须至少为256MiB。

​	`readonly`

​		`readonly`元素用于将vNVDIMM标记为只读。只有真正的NVDIMM设备后端才能保证客户机写入持久性，因此其他后端类型应该使用`readonly`元素。自5.0.0起

​	`block`

​		仅供`virtio mem`使用。单个块的大小，内存块划分的粒度。必须是二的幂，并且至少等于transparent hugepage的大小(x84_64上的2MiB)。默认情况是依赖于虚拟机监控程序。

​	`requested`

​		仅供`virtio mem`使用。暴露给客户机的总大小。必须符合`block`粒度并小于或等于`size`。

​	`current`

​		`virtio-mem`模型的active XML可能包含反映相应virtio内存设备当前大小的`current`元素。该元素被格式化为live XML，并且从未被解析，即它是仅输出的元素。

​	`address`

​		仅适用于`virtio-mem`和`virtio-pmem`。内存中映射设备的物理地址。自9.4.0起



#### 1.20.27 (IOMMU devices)

`iommu`元素可以用于添加IOMMU设备。自2.1.0起。例子：

```xml
...
<devices>
  <iommu model='intel'>
    <driver intremap='on'/>
  </iommu>
</devices>
...
```

`model`

​	支持的值为 `intel`(适用于Q35 客户机)`smmuv3`(自5.5.0起，适用于ARM virt客户机)和virtio(自8.3.0起，用于Q35和ARM virt 客户机)。

`driver`

​	`driver`子元素可用于配置其他选项，其中一些选项可能仅适用于某些IOMMU 模型：

​	`intermap`

​		`intremap`属性有`on`和`off`的值，可用于开启中断重映射，这是VT-d功能的一部分。目前，这需要拆分I/O APIC(<ioapic driver='qemu'/>)。自3.4.0起(仅限QEMU/KVM)

​	`caching_mode`

​		`caching_mode`属性(有`on`和`off`的值)可用于打开VT-d缓存模式(对指定的设备有用)。自3.4.0起(仅限QEMU/KVM)

​	`eim`

​		`eim`属性(有`on`和`off`的值)可用于配置扩展中断模式。具有拆分I/O APIC(如**Hypervisor features**小节中所述)的q35域，并且IOMMU的中断重映射和EIM都已打开，将能够使用超过255个vCPU。自3.4.0起(仅限QEMU/KVM)

​	`iotlb`

​		可以使用值为`on`和`off`的`iotlb`属性来打开IOTLB，该`iotlb`用于缓存来自设备的地址转换请求。自3.5.0起(仅限QEMU/KVM)

​	`aw_bits`

​		`aw_bits`属性可用于设置地址宽度，以允许在客户机中映射更大的iova地址。自6.5.0起(仅限QEMU/KVM)

`virtio` IOMMU设备还可以具有`address`元素，如**Device addresses**小节中所述(地址必须根据`pci`类型而定)。



#### 1.20.28 (Vsock)

vsock主机/客户机接口。`model`属性默认为`virtio`。自5.2.0 `model`也可以是"virtio transitional"和"virtio-notransitional"，请参阅** 1.20.5[Virtio transitional devices](https://libvirt.org/formatdomain.html#virtio-transitional-devices)小节中以了解更多详细信息。`cid`元素的可选属性`address`指定分配给客户机的cid。如果属性`auto`设置为`yes`，那么libvirt将在域启动时自动分配一个空闲CID。自4.4.0起，可选的`driver`元素允许指定virtio选项，请参阅1.20.4 [Virtio-related options](https://libvirt.org/formatdomain.html#virtio-related-options)以了解更多详细信息。自7.1.0起

```xml
...
<devices>
  <vsock model='virtio'>
    <cid auto='no' address='3'/>
  </vsock>
</devices>
...
```



#### 1.20.29 (Crypto)

加密设备。`model`属性默认为`virtio`。自v9.0.0 `model`仅支持`virtio`。`type`属性默认为`qemu`。自v9.0.0起 `type`仅支持`qemu`。如果`type`为`qemu`，则需要可选属性`backend`，`model`属性可以是`builtint`的和`lkcf`，可选属性`queues`指定virtio加密的virt队列数量。

```xml
...
<devices>
  <crypto model='virtio' type='qemu'>
    <backend model='builtin' queues='1'/>
  </crypto>
</devices>
...
```



### 1.21 安全标识(Security label)

`seclabel`元素允许控制安全驱动程序的操作。有三种基本的操作模式，"dynamic"(libvirt自动生成安全唯一标识)、"static"(应用程序/管理员选择标签)或"none"(禁用限制)。通过动态标签生成，libvirt将始终自动重新标记与虚拟机关联的任何资源。默认情况下，使用静态标签分配，管理员或应用程序必须确保在任何资源上正确设置标签，但是，如果需要，可以启用自动重新标记。"自0.6.1以来为"dynamic"，自0.6.2以来为"static"，自0.9.10以来为"none"。

如果libvirt使用多个安全驱动程序，则可以使用多个`seclabel`标记，每个驱动程序一个，并且可以使用属性`model`定义每个标记引用的安全驱动程序

顶级安全标签的有效输入XML配置为：

```xml
<seclabel type='dynamic' model='selinux'/>

<seclabel type='dynamic' model='selinux'>
  <baselabel>system_u:system_r:my_svirt_t:s0</baselabel>
</seclabel>

<seclabel type='static' model='selinux' relabel='no'>
  <label>system_u:system_r:svirt_t:s0:c392，c662</label>
</seclabel>

<seclabel type='static' model='selinux' relabel='yes'>
  <label>system_u:system_r:svirt_t:s0:c392，c662</label>
</seclabel>

<seclabel type='none'/>
```

如果输入XML中没有提供"type"属性，则将使用安全驱动程序的默认设置，该设置可以是"none"或"dynamic"。如果设置了"baselabel"，但没有设置"type"，则该类型被假定为"dynamic"

在自动资源重新标记处于激活状态的情况下查看正在运行的客户机的XML时，将包括一个额外的XML元素`imagelabel`。这是一个仅输出的元素，因此在用户提供的XML文档中将被忽略

`type`

​	`static`、`dynamic`或`none`来确定libvirt是否自动生成安全唯一标识。

`model`

​	有效的安全模型名称，与当前激活的安全模型匹配。当客户机由无特权用户运行时，模型`dac`不可用。

`relabel`

​	`yes`或`no`。如果使用动态标签分配，则必须始终为`yes`。对于静态标签分配，它将默认为`no`。

`label`

​	如果使用静态标签，则必须指定要分配给虚拟域的完整安全标签。内容的格式取决于所使用的安全驱动程序：

​	·SELinux：一个SELinux上下文。

​	·AppArmor:AppArmor配置文件。

​	·DAC：所有者和组用冒号分隔。它们既可以定义为用户名/组名，也可以定义为uid/gid。驱动程序将首先尝试将这些值解析为名称，但可以使用前导加号强制驱动程序将它们解析为uid或gid。

`baselabel`

​	如果使用动态标签，则可以选择性地使用它来指定将用于生成实际标签的基本安全标签。内容的格式取决于所使用的安全驱动程序。SELinux驱动程序在生成的标签中只使用基线标签的`type`字段。当使用SELinux基线时，其他字段从父进程继承。(上面的示例演示了使用`my_svirt_t`作为`type`字段的值。)

`imagelabel`	

​	这是一个仅输出的元素，显示与虚拟域关联的资源上使用的安全标签。内容的格式取决于所使用的安全驱动程序

当重新标记生效时，也可以通过禁用标记(如果文件位于NFS或其他缺乏安全标识的文件系统上，则很有用)或请求备用标签(当管理应用程序创建特殊标签以允许在域之间共享部分但非全部资源时很有用)来微调为特定源文件名所做的标记，自0.9.9起。当`seclabel`元素附加到特定路径而不是顶级域分配时，只支持属性`relabel`或子元素标签。此外，自1.1.2起，对于磁盘上的活动域，由于镜像位于缺少安全标记的文件系统上而跳过了标记，将出现仅输出元素`labelskip`。

### 1.22 (Key Wrap)

可选`keywrap`元素的内容指定是否允许客户机执行S390加密密钥管理操作。通过使用为主机上运行的每个客户机VM生成的唯一包装密钥对清除密钥进行加密，可以保护该密钥。生成了两种封装密钥变体：一种版本用于使用DEA/TDEA算法加密受保护的密钥，另一种版本则用于使用AES算法加密的密钥。如果不包括`keywrap`元素，则默认情况下，客户机将被授予对AES和DEA/TDEA密钥封装的访问权限。

```xml
<domain>
  ...
  <keywrap>
    <cipher name='aes' state='off'/>
  </keywrap>
  ...
</domain>
```

`keywrap`元素中必须至少嵌套一个`cipher`元素。

`cipher`密码

​	`name`属性标识用于加密受保护密钥的算法。此属性支持的值为`aes`(用于在aes包装密钥下加密)或`dea`(用于在dea/TDEA包装密钥下进行加密)。`state`属性指示是否应为指定的加密算法启用加密密钥管理操作。该值可以设置为`on`或`off`。

注：DEA/TDEA是DES/TDES的同义词



### 1.23 (Launch Security)

在s390域中指定<launchSecurity type='s390-pv'/>可使客户机准备在受保护的虚拟化安全模式(也称为IBM安全执行)下运行。有关更多所需的主机和可不急准备步骤，请参阅s390上的Protected Virtualization(自7.6.0起)

<launchSecurity type='sev'>元素的内容用于提供客户机所有者输入，用于使用AMD sev功能(安全加密虚拟化)创建加密虚拟机。SEV是AMD-V体系结构的扩展，支持在KVM控制下运行加密虚拟机(VM)。加密的虚拟机对其页面(代码和数据)进行了保护，这样只有客户机自己才能访问未加密的版本。每个加密的VM都与唯一的加密密钥相关联；如果使用不同的密钥将其数据访问到不同的实体，则加密的客户机数据将被错误地解密，从而导致无法理解的数据。有关更多信息，请参阅各种输入参数及其格式，请参阅自4.4.0起的**SEV API spec** https://support.amd.com/TechDocs/55766_SEV-KM_API_Specification.pdf

```xml
<domain>
  ...
  <launchSecurity type='sev' kernelHashes='yes'>
    <policy>0x0001</policy>
    <cbitpos>47</cbitpos>
    <reducedPhysBits>1</reducedPhysBits>
    <dhCert>RBBBSDDD=FDDCCCDDDG</dhCert>
    <session>AAACCCDD=FFFCCCDSDS</session>
  </launchSecurity>
  ...
</domain>
```

`kernelHashes`

​	可选的`kernelHashes`属性指定内核、ramdisk和命令行的哈希是否应包含在固件完成的测量中。这仅在使用直接内核引导时有效。自8.0.0

`cbitpos`

​	所需的`cbitpos`元素在客户机页面表条目中提供C-bit(也称为加密位)位置。`cbitpos`的值依赖于系统监控程序，可以通过域功能中的`sev`元素获得。

`reducedPhysBits`

​	所需的`reducedPhysBits`元素提供物理地址位缩减。与`cbitpos`类似，`reduced-phys-bit`的值依赖于系统监控程序，并且可以通过域功能中的`sev`元素获得。

`policy`

​	所需的`policy`元素提供必须由SEV固件维护的客户机策略。此策略由固件强制执行，并限制hypervisor可以在此客户机上执行的配置和操作命令。客户机启动期间提供的客户机策略与客户机绑定，在客户机的整个生命周期内不能更改。该策略还在快照和迁移流期间传输，并在目标平台上强制执行。客户机策略是一个4个无符号字节，其字段如表所示：

![image-20230926103543472](D:\File Storage\TyporaStorage\image-20230926103543472.png)

`dhCert`

​	可选的`dhCert`元素提供客户机所有者base64编码的Diffie-Hellman(DH)密钥。密钥用于在SEV固件和客户机所有者之间协商主密钥。然后，该主密钥用于在SEV固件和客户机所有者之间建立可信通道。

`session`

​	可选`session`元素提供在SEV API规范中定义的客户机所有者base64编码的会话blob。有关会话blob格式，请参阅SEV规范LAUNCH_START部分。



## 2. 配置示例

下面列出的驱动程序特定页面上提供了每个驱动程序的示例配置

- [Xen 示例](https://libvirt.org/drvxen.html#example-domain-xml-config)
- [QEMU/KVM 示例](https://libvirt.org/drvqemu.html#example-domain-xml-config) 

注1：Xen 是一种虚拟化技术和开源虚拟机监控程序(Hypervisor)，旨在在一台物理计算机上实现多个虚拟机实例，从而使多个操作系统和应用程序能够同时运行，实现资源的有效利用和隔离。Type1型虚拟化，虚拟化层直接运行在硬件之上，没有所谓的宿主机操作系统。它能直接控制硬件资源以及客户机。







```
user_hosts = [{ name = "master01.sugon.local", bond-manage_ip = "10.0.33.197", bond-storagepub_ip = "10.0.34.200" },
{ name = "compute01.sugon.local", bond-manage_ip = "10.0.33.199", bond-storagepub_ip = "10.0.34.206" },
]
```

```
user_hosts = [{ name = "master01.sugon.local", bond-manage_ip = "10.0.33.197", bond-storagepub_ip = "100.125.254.1" },{ name = "compute01.sugon.local", bond-manage_ip = "10.0.33.199", bond-storagepub_ip = "10.0.34.206" },
]
```

