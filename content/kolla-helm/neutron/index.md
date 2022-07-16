---
"title": "部署 neutron"
"weight": 4
---

目前 kolla-helm 支持部署两种网络插件：linuxbridge 和 openvswitch。

## 部署 neutron

- 使用 liunxbridge 做为网络插件

```shell
helm install -n openstack openstack-neutron kolla-helm/neutron 
    --set networkPlugins.external_interface=eth1
```

- 使用 openvswitch 做为网络插件

```shell
helm install -n openstack openstack-glance kolla-helm/glance \
    --set networkPlugins.external_interface=eth1 \
    --set networkPlugins.linux_bridge.enabled=false \ 
    --set networkPlugins.ovs.enabled=ture \
    --set networkPlugins.ovs.tunnel_interface=eth2 
```

默认情况下是采用 liunxbridge 做为网络插件,
`networkPlugins.external_interface=eth1` 是根据具体的的环境，指定一张网卡，做为外网网卡 `eth1` 为该外网网卡的名称。
`networkPlugins.ovs.tunnel_interface=eth2` 是根据具体的的环境，指定一张网卡，做为隧道网网卡 `eth2` 为该隧道网卡的名称。

## 验证

通过下面命令观察

```shell

watch -n 1 kubectl -n openstack get pods -l app.kubernetes.io/instance=openstack-neutron
```

等待所有的 pod 都 ready 后，创建 image 检验 neutron 的功能是否正常。

### 验证 neutron 各个 agent 是否正常

```shell
openstack network agent list
```

### 创建网络

```shell
openstack network create 
    --provider-network-type=vlan 
    --provider-physical-network=physnet1 
    --provider-segment=101 
    --external --share net1
```
