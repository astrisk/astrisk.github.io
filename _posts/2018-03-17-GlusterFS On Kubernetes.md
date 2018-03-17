---
title:  "kubernetes动态卷-GlusterFS"
date:   2018-03-17 19:37:00
categories: kubernetes-docker
---

之前按照《kubernetes权威指南》在kubernetes中部署GlusterFS总是不成功，最后各种Google，才搞成功。这之间踩了不少坑。

在kubernetes中部署GlusterFS有两个须知：
1.需要3个或以上kubernetes node节点。
2.用来部署GlusterFS的硬盘，必须是裸盘（没有文件系统），如果安装失败，要重新安装，必须删除安装的vg 和pv(如何删除，下面清理资源部分会提到)。

**install GlusterFS client on all nodes**

{% highlight c %}
# yum install -y glusterfs glusterfs-fuse

{% endhighlight %}

**更新apiserver启动参数**


**如果安装失败，重新安装需要清理资源**

**master 上操作删除**

{% highlight c %}

kubectl delete -f kube-templates/deploy-heketi-deployment.yaml
kubectl delete -f kube-templates/heketi-deployment.yaml
kubectl delete -f kube-templates/heketi-service-account.yaml
kubectl delete -f kube-templates/glusterfs-daemonset.yaml

kubectl delete  clusterrolebinding/heketi-sa-view
kubectl delete secrets/heketi-config-secret 

kubectl delete svc/glusterfs-dynamic-gluster1
kubectl delete svc/heketi-storage-endpoints

{% endhighlight %}

**在各个node上删除**

{% highlight c %}

# vgs
/run/lvm/lvmetad.socket: connect failed: Connection refused
WARNING: Failed to connect to lvmetad. Falling back to device scanning.
VG                                  #PV #LV #SN Attr   VSize   VFree
vg_7307f98ae54a8dede4c7664f7cbe7926   1   4   0 wz--n- 199.87g 192.80g
#  vgremove vg_7307f98ae54a8dede4c7664f7cbe7926
/run/lvm/lvmetad.socket: connect failed: Connection refused
WARNING: Failed to connect to lvmetad. Falling back to device scanning.
Do you really want to remove volume group "vg_7307f98ae54a8dede4c7664f7cbe7926" containing 4 logical volumes? [y/n]: y
Removing pool "tp_dc9b5839cfa520e246a2baf15292b473" will remove 1 dependent volume(s). Proceed? [y/n]: y
Do you really want to remove active logical volume vg_7307f98ae54a8dede4c7664f7cbe7926/brick_dc9b5839cfa520e246a2baf15292b473? [y/n]: y
Logical volume "brick_dc9b5839cfa520e246a2baf15292b473" successfully removed
Do you really want to remove active logical volume vg_7307f98ae54a8dede4c7664f7cbe7926/tp_dc9b5839cfa520e246a2baf15292b473? [y/n]: y
Logical volume "tp_dc9b5839cfa520e246a2baf15292b473" successfully removed
Removing pool "tp_3b2071d61b1cc72e144c3fd16d9f3c56" will remove 1 dependent volume(s). Proceed? [y/n]: y
Do you really want to remove active logical volume vg_7307f98ae54a8dede4c7664f7cbe7926/brick_3b2071d61b1cc72e144c3fd16d9f3c56? [y/n]: y
Logical volume "brick_3b2071d61b1cc72e144c3fd16d9f3c56" successfully removed
Do you really want to remove active logical volume vg_7307f98ae54a8dede4c7664f7cbe7926/tp_3b2071d61b1cc72e144c3fd16d9f3c56? [y/n]: y
Logical volume "tp_3b2071d61b1cc72e144c3fd16d9f3c56" successfully removed
Volume group "vg_7307f98ae54a8dede4c7664f7cbe7926" successfully removed

# pvs
# pvremove /dev/vdc
/run/lvm/lvmetad.socket: connect failed: Connection refused
WARNING: Failed to connect to lvmetad. Falling back to device scanning.
Labels on physical volume "/dev/vdc" successfully wiped.

# rm -rf /var/lib/heketi
# rm -rf /var/lib/glusterd

{% endhighlight %}


**DefaultStorageClass** 

- 在api server 中开启 admission controller 和 DefaultStorageClass插件
- 创建一个DefaultStorageClass

{% highlight c %}

apiVersion: storage.k8s.io/v1beta1
kind: StorageClass
metadata:
  name: slow
  annotations:
   storageclass.beta.kubernetes.io/is-default-class: "true"
provisioner: kubernetes.io/glusterfs
parameters:
  resturl: "http://x.x.x.x:30625"
  restuser: ""
  restuserkey: ""

{% endhighlight %}

**验证**

{% highlight c %}
# kubectl get storageclass

NAME             TYPE
gluster-heketi   kubernetes.io/glusterfs
slow (default)   kubernetes.io/glusterfs

{% endhighlight %}

**创建一个测试pvc,使用DefaultStorageClass**

{% highlight c %}

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: gluster-defalut-test
spec:
 accessModes:
  - ReadWriteOnce
 resources:
   requests:
     storage: 2Gi

{% endhighlight %}

**问题：storageclass.storage.k8s.io "default" not found**

{% highlight c %}

apiVersion: v1
kind: PersistentVolumeClaim
metadata:
 name: gluster1
 annotations:
   volume.beta.kubernetes.io/storage-class: default # 不需要这个annotations，其实这样写会去找一个名称为default的storage-class，但是我们并没有定义名称为default的storage-class
spec:
 accessModes:
  - ReadWriteOnce
 resources:
   requests:
     storage: 5Gi

{% endhighlight %}

**验证pvc是否创建和绑定pv**

{% highlight c %}

# kubectl get pvc

NAME                   STATUS    VOLUME                                     CAPACITY   ACCESSMODES   STORAGECLASS     AGE
gluster-defalut-test   Bound     pvc-91647c21-d57d-11e7-bb8b-52540086d79e   2Gi        RWO           slow             13m
gluster1               Bound     pvc-3532129b-d565-11e7-b222-52540086d79e   5Gi        RWO           gluster-heketi   3h

# kubectl get pv 

NAME                                       CAPACITY   ACCESSMODES   RECLAIMPOLICY   STATUS    CLAIM                          STORAGECLASS     REASON    AGE
pvc-3532129b-d565-11e7-b222-52540086d79e   5Gi        RWO           Delete          Bound     default/gluster1               gluster-heketi             3h
pvc-91647c21-d57d-11e7-bb8b-52540086d79e   2Gi        RWO           Delete          Bound     default/gluster-defalut-test   slow                       12m

{% endhighlight %}

**参考文章** 

- https://github.com/gluster/gluster-kubernetes
- http://yoyolive.com/2017/03/09/Kubernetes-Deploy-GlusterFS/
- https://techdev.io/en/developer-blog/deploying-glusterfs-in-your-bare-metal-kubernetes-cluster
- https://www.google.co.uk/url?sa=t&rct=j&q=&esrc=s&source=web&cd=2&ved=0ahUKEwjF1JPIquXXAhXITrwKHUjWDOgQFggxMAE&url=https%3a%2f%2fkubernetes%2eio%2fdocs%2ftasks%2fadminister-cluster%2fchange-default-storage-class%2f&usg=AOvVaw0t1ZMz2MAzvyshogXfCmAw
- https://docs.openshift.org/latest/install_config/storage_examples/storage_classes_dynamic_provisioning.html#example2
- https://kubernetes.io/docs/tasks/administer-cluster/change-default-storage-class/#before-you-begin
- https://blog.lwolf.org/post/how-i-deployed-glusterfs-cluster-to-kubernetes/
