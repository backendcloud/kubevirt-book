# KubeVirt 基本使用

KubeVirt目的是让虚拟机运行在容器中，下面就用下KubeVirt的几个基本操作：
* create & start 虚拟机
* vnc 登录 虚拟机
* stop & delete 虚拟机

```bash
# vm.yaml: 定义一个虚拟机
controlplane $ curl https://kubevirt.io/labs/manifests/vm.yaml
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  name: testvm
spec:
  running: false
  template:
    metadata:
      labels:
        kubevirt.io/size: small
        kubevirt.io/domain: testvm
    spec:
      domain:
        devices:
          disks:
            - name: containerdisk
              disk:
                bus: virtio
            - name: cloudinitdisk
              disk:
                bus: virtio
          interfaces:
          - name: default
            masquerade: {}
        resources:
          requests:
            memory: 64M
      networks:
      - name: default
        pod: {}
      volumes:
        - name: containerdisk
          containerDisk:
            image: quay.io/kubevirt/cirros-container-disk-demo
        - name: cloudinitdisk
          cloudInitNoCloud:
            userDataBase64: SGkuXG4=

# 创建一个虚拟机定义
controlplane $ kubectl apply -f https://kubevirt.io/labs/manifests/vm.yaml
virtualmachine.kubevirt.io/testvm created
controlplane $ kubectl get vms
NAME     AGE   STATUS    READY
testvm   2s    Stopped   False
controlplane $ kubectl get vms -o yaml testvm
apiVersion: kubevirt.io/v1
kind: VirtualMachine
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"kubevirt.io/v1","kind":"VirtualMachine","metadata":{"annotations":{},"name":"testvm","namespace":"default"},"spec":{"running":false,"template":{"metadata":{"labels":{"kubevirt.io/domain":"testvm","kubevirt.io/size":"small"}},"spec":{"domain":{"devices":{"disks":[{"disk":{"bus":"virtio"},"name":"containerdisk"},{"disk":{"bus":"virtio"},"name":"cloudinitdisk"}],"interfaces":[{"masquerade":{},"name":"default"}]},"resources":{"requests":{"memory":"64M"}}},"networks":[{"name":"default","pod":{}}],"volumes":[{"containerDisk":{"image":"quay.io/kubevirt/cirros-container-disk-demo"},"name":"containerdisk"},{"cloudInitNoCloud":{"userDataBase64":"SGkuXG4="},"name":"cloudinitdisk"}]}}}}
    kubevirt.io/latest-observed-api-version: v1
    kubevirt.io/storage-observed-api-version: v1alpha3
  creationTimestamp: "2022-05-06T06:32:17Z"
  generation: 1
  managedFields:
  - apiVersion: kubevirt.io/v1alpha3
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          f:kubevirt.io/latest-observed-api-version: {}
          f:kubevirt.io/storage-observed-api-version: {}
      f:status:
        .: {}
        f:conditions: {}
        f:printableStatus: {}
        f:volumeSnapshotStatuses: {}
    manager: Go-http-client
    operation: Update
    time: "2022-05-06T06:32:17Z"
  - apiVersion: kubevirt.io/v1
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:kubectl.kubernetes.io/last-applied-configuration: {}
      f:spec:
        .: {}
        f:running: {}
        f:template:
          .: {}
          f:metadata:
            .: {}
            f:labels:
              .: {}
              f:kubevirt.io/domain: {}
              f:kubevirt.io/size: {}
          f:spec:
            .: {}
            f:domain:
              .: {}
              f:devices:
                .: {}
                f:disks: {}
                f:interfaces: {}
              f:resources:
                .: {}
                f:requests:
                  .: {}
                  f:memory: {}
            f:networks: {}
            f:volumes: {}
    manager: kubectl
    operation: Update
    time: "2022-05-06T06:32:17Z"
  name: testvm
  namespace: default
  resourceVersion: "5996"
  selfLink: /apis/kubevirt.io/v1/namespaces/default/virtualmachines/testvm
  uid: 1b047ffb-6eeb-4fda-b081-169391886bfb
spec:
  running: false
  template:
    metadata:
      creationTimestamp: null
      labels:
        kubevirt.io/domain: testvm
        kubevirt.io/size: small
    spec:
      domain:
        devices:
          disks:
          - disk:
              bus: virtio
            name: containerdisk
          - disk:
              bus: virtio
            name: cloudinitdisk
          interfaces:
          - masquerade: {}
            name: default
        machine:
          type: q35
        resources:
          requests:
            memory: 64M
      networks:
      - name: default
        pod: {}
      volumes:
      - containerDisk:
          image: quay.io/kubevirt/cirros-container-disk-demo
        name: containerdisk
      - cloudInitNoCloud:
          userDataBase64: SGkuXG4=
        name: cloudinitdisk
status:
  conditions:
  - lastProbeTime: "2022-05-06T06:32:18Z"
    lastTransitionTime: "2022-05-06T06:32:18Z"
    message: VMI does not exist
    reason: VMINotExists
    status: "False"
    type: Ready
  printableStatus: Stopped
  volumeSnapshotStatuses:
  - enabled: false
    name: containerdisk
    reason: Snapshot is not supported for this volumeSource type [containerdisk]
  - enabled: false
    name: cloudinitdisk
    reason: Snapshot is not supported for this volumeSource type [cloudinitdisk]

# 开启虚拟机并检查状态
controlplane $ ./virtctl start testvm
VM testvm was scheduled to start

# 获取虚拟机信息
controlplane $ kubectl get vms
NAME     AGE   STATUS    READY
testvm   40s   Running   True

# 获取虚拟机实例信息
controlplane $ kubectl get vmis
NAME     AGE    PHASE     IP            NODENAME   READY
testvm   104s   Running   10.244.1.12   node01     True
controlplane $  kubectl get vmis -o yaml testvm
apiVersion: kubevirt.io/v1
kind: VirtualMachineInstance
metadata:
  annotations:
    kubevirt.io/latest-observed-api-version: v1
    kubevirt.io/storage-observed-api-version: v1alpha3
  creationTimestamp: "2022-05-06T06:32:38Z"
  finalizers:
  - kubevirt.io/virtualMachineControllerFinalize
  - foregroundDeleteVirtualMachine
  generation: 9
  labels:
    kubevirt.io/domain: testvm
    kubevirt.io/nodeName: node01
    kubevirt.io/size: small
  managedFields:
  - apiVersion: kubevirt.io/v1alpha3
    fieldsType: FieldsV1
    fieldsV1:
      f:metadata:
        f:annotations:
          .: {}
          f:kubevirt.io/latest-observed-api-version: {}
          f:kubevirt.io/storage-observed-api-version: {}
        f:finalizers: {}
        f:labels:
          .: {}
          f:kubevirt.io/domain: {}
          f:kubevirt.io/nodeName: {}
          f:kubevirt.io/size: {}
        f:ownerReferences: {}
      f:spec:
        .: {}
        f:domain:
          .: {}
          f:devices:
            .: {}
            f:disks: {}
            f:interfaces: {}
          f:firmware:
            .: {}
            f:uuid: {}
          f:machine:
            .: {}
            f:type: {}
          f:resources:
            .: {}
            f:requests:
              .: {}
              f:memory: {}
        f:networks: {}
        f:volumes: {}
      f:status:
        .: {}
        f:activePods:
          .: {}
          f:5ed78651-9f18-426f-a248-4bf7b2c8b003: {}
        f:conditions: {}
        f:guestOSInfo: {}
        f:interfaces: {}
        f:launcherContainerImageVersion: {}
        f:migrationMethod: {}
        f:migrationTransport: {}
        f:nodeName: {}
        f:phase: {}
        f:phaseTransitionTimestamps: {}
        f:qosClass: {}
        f:virtualMachineRevisionName: {}
        f:volumeStatus: {}
    manager: Go-http-client
    operation: Update
    time: "2022-05-06T06:32:47Z"
  name: testvm
  namespace: default
  ownerReferences:
  - apiVersion: kubevirt.io/v1
    blockOwnerDeletion: true
    controller: true
    kind: VirtualMachine
    name: testvm
    uid: 1b047ffb-6eeb-4fda-b081-169391886bfb
  resourceVersion: "6141"
  selfLink: /apis/kubevirt.io/v1/namespaces/default/virtualmachineinstances/testvm
  uid: 3721e713-37e9-4871-bb5e-aaad3ab5d44b
spec:
  domain:
    cpu:
      cores: 1
      model: host-model
      sockets: 1
      threads: 1
    devices:
      disks:
      - disk:
          bus: virtio
        name: containerdisk
      - disk:
          bus: virtio
        name: cloudinitdisk
      interfaces:
      - masquerade: {}
        name: default
    features:
      acpi:
        enabled: true
    firmware:
      uuid: 5a9fc181-957e-5c32-9e5a-2de5e9673531
    machine:
      type: q35
    resources:
      requests:
        memory: 64M
  networks:
  - name: default
    pod: {}
  volumes:
  - containerDisk:
      image: quay.io/kubevirt/cirros-container-disk-demo
      imagePullPolicy: Always
    name: containerdisk
  - cloudInitNoCloud:
      userDataBase64: SGkuXG4=
    name: cloudinitdisk
status:
  activePods:
    5ed78651-9f18-426f-a248-4bf7b2c8b003: node01
  conditions:
  - lastProbeTime: null
    lastTransitionTime: "2022-05-06T06:32:46Z"
    status: "True"
    type: Ready
  - lastProbeTime: null
    lastTransitionTime: null
    status: "True"
    type: LiveMigratable
  guestOSInfo: {}
  interfaces:
  - ipAddress: 10.244.1.12
    ipAddresses:
    - 10.244.1.12
    mac: 52:54:00:f8:cf:ce
    name: default
  launcherContainerImageVersion: quay.io/kubevirt/virt-launcher:v0.49.0
  migrationMethod: BlockMigration
  migrationTransport: Unix
  nodeName: node01
  phase: Running
  phaseTransitionTimestamps:
  - phase: Pending
    phaseTransitionTimestamp: "2022-05-06T06:32:39Z"
  - phase: Scheduling
    phaseTransitionTimestamp: "2022-05-06T06:32:39Z"
  - phase: Scheduled
    phaseTransitionTimestamp: "2022-05-06T06:32:46Z"
  - phase: Running
    phaseTransitionTimestamp: "2022-05-06T06:32:48Z"
  qosClass: Burstable
  virtualMachineRevisionName: revision-start-vm-1b047ffb-6eeb-4fda-b081-169391886bfb-2
  volumeStatus:
  - name: cloudinitdisk
    size: 1048576
    target: vdb
  - name: containerdisk
    target: vda

# vnc 登录 创建 的 虚拟机实例
controlplane $ ./virtctl console testvm
Successfully connected to testvm console. The escape sequence is ^]

login as 'cirros' user. default password: 'gocubsgo'. use 'sudo' for root.
testvm login: cirros
Password: 
$ pwd
/home/cirros
$ ls /
bin         home        lib64       mnt         root        tmp
boot        init        linuxrc     old-root    run         usr
dev         initrd.img  lost+found  opt         sbin        var
etc         lib         media       proc        sys         vmlinuz
$ exit

login as 'cirros' user. default password: 'gocubsgo'. use 'sudo' for root.
testvm login: controlplane $ 
controlplane $ ssh 10.224.1.12
^C
controlplane $ ./virtctl stop testvm
VM testvm was scheduled to stop
controlplane $ kubectl get vms
NAME     AGE     STATUS    READY
testvm   4m23s   Stopped   False
controlplane $ kubectl get vmis
NAME     AGE    PHASE       IP            NODENAME   READY
testvm   4m4s   Succeeded   10.244.1.12   node01     False
controlplane $ kubectl delete vms testvm
virtualmachine.kubevirt.io "testvm" deleted
controlplane $ kubectl get vms
No resources found in default namespace.
controlplane $ kubectl get vmis
No resources found in default namespace.
```