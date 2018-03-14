---
title:  "kubernetes 存储 NFS 部署"
date:   2018-03-11 23:47:00
categories: kubernetes-docker
---

**部署和配置NFS服务器**

- 查看是否安装nfs: yum list installed | grep  nfs
- 安装NFS相关软件: yum install nfs-utils
- 创建共享目录: mkdir -p /var/nfs/share && chmod 777 /var/nfs/share
- 修改NFS配置

{% highlight c %}
# vim /etc/exports

 /var/nfs/share *(rw,insecure,sync,no_subtree_check,no_root_squash)

{% endhighlight %}

**启动NFS相关服务**

{% highlight c %}
# systemctl enable rpcbind
# systemctl start rpcbind
# systemctl enable nfs-server
# systemctl start nfs-server
# exportfs

# cd /var/nfs/share
# touch nfs-test #创建测试文件
{% endhighlight %}

**在kubernetes node 安装 nfs client**

{% highlight c %}
# yum install nfs-utils
# mount -F nfs {nfs-server}:/var/nfs/share /tmp
# ls -la /tmp #可以看到测试文件nfs-test
# umount /tmp #卸载

# exportfs -r #更新配置后的刷新命令
{% endhighlight %}

**k8s yaml中使用NFS**

{% highlight c %}

  volumes:
    - name: gradlehome
      nfs:
        server: nfs-server-ip
        path: "/var/nfs/share"  

{% endhighlight %}

备注：也可以在k8s中创建pv和pvc来管理持久存储,后面介绍kubernetes-nfs动态存储

