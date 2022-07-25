# 什么是 KubeVirt ？

Kubevirt 是Redhat开源的以容器方式运行虚拟机的项目，以k8s add-on方式，利用k8s CRD为增加资源类型VirtualMachineInstance（VMI）， 使用容器的image registry去创建虚拟机并提供VM生命周期管理。 CRD的方式是的kubevirt对虚拟机的管理不局限于pod管理接口，但是也无法使用pod的RS DS Deployment等管理能力，也意味着 kubevirt如果想要利用pod管理能力，要自主去实现，目前kubevirt实现了类似RS的功能。 kubevirt目前支持的runtime是docker和runv。


# 基本组件

* virt-api: 
  * 为 Kubevirt 提供 API 服务能力，比如许多自定义的 API 请求，如开机、关机、重启等操作，通过 APIService 作为 Kubernetes Apiserver 的插件，业务可以通过 Kubernetes Apiserver 直接请求到 virt-api；
* virt-controller: 
  * Kubevirt 的控制器，功能类似于 Kubernetes 的 controller-manager，管理和监控 VMI 对象及其关联的 Pod，对其状态进行更新；
* virt-handler:
  * 以 Daemonset 形式部署，功能类似于 Kubelet，通过 Watch 本机 VMI 和实例资源，管理本宿主机上所有虚机实例；
  * 主要执行动作如下:
    * 使 VMI 中定义的 Spec 与相应的 libvirt （本地 socket 通信）保持同步;
    * 汇报及控制更新虚拟机状态;
    * 调用相关插件初始化节点上网络和存储资源;
    * 热迁移相关操作;
* virt-launcher: 
  * Kubevirt 会为每一个 VMI 对象创建一个 Pod，该 Pod 的主进程为 virt-launcher，virt-launcher 的 Pod 提供了 cgroups 和 namespaces 的隔离，virt-launcher 为虚拟机实例的主进程。
  * virt-handler 通过将 VMI 的 CRD 对象传递给 virt-launcher 来通知 virt-launcher 启动 VMI。然后，virt-launcher 在其容器中使用本地 libvirtd 实例来启动 VMI。virt-launcher 托管 VMI 进程，并在 VMI 退出后终止。
  * 如果 Kubernetes 运行时在 VMI 退出之前尝试关闭 virt-launcher 容器，virt-launcher 会将信号从Kubernetes 转发到 VMI 进程，并尝试推迟容器的终止，直到 VMI 成功关闭。


# virt-launcher 与 libvirt 通信概略图
![](2022-07-25-17-24-01.png)


# 资源对象
Kubevirt 是 Kubernetes 的虚拟机管理插件，通过自定义控制器和资源实现对虚拟机的管理功能，通过自定义资源(CRD)机制，同时 Kubevirt 可以自定义额外的操作，来调整常规容器中不可用的行为。这里介绍几个关键资源：
* VirtualMachineInstance（VMI） ：是管理虚拟机的最小资源，一个 VirtualMachineInstance 对象表示一台正在运行的虚拟机实例，包含一个虚拟机所需要的各种配置。
* VirtualMachine（ VM ） ：为集群内的 VirtualMachineInstance 提供管理功能，例如开机/关机/重启虚拟机，确保虚拟机实例的启动状态，与虚拟机实例是 1:1 的关系。
* VirtualMachineInstanceMigrations：虚拟机迁移需要的资源，一个资源对象表示为一次迁移任务，并反映出虚拟机迁移的状态。
* VirtualMachineInstanceReplicaSet：类似 ReplicaSet，可以指定数量，批量创建虚拟机。
* DataVolume:   是对 PVC 之上的抽象，通过自定义数据源，由 CDI 控制器自动创建 PVC 并导入数据到 PVC 中供虚拟机使用。