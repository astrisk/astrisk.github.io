---
title:  "Kubernetes Packages Manager Tools - Helm install "
date:   2018-03-13 22:37:00
categories: example
---


**在k8s master服务器进行以下操作，完整后，在master会安装helm的客户端，在k8s中安装server：tiller**


**download**

{% highlight c %}
# wget https://storage.googleapis.com/kubernetes-helm/helm-v2.7.0-linux-amd64.tar.gz
# tar -zxvf helm-v2.7.0-linux-amd64.tar.gz
# mv linux-amd64/helm /usr/local/bin/helm 

{% endhighlight %}

**configure rbac in cluster**

由于kubernetes 1.6.0 并且开启了RBAC功能，需要配置helm的RBAC

{% highlight c %}

# vim rbac-config.yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: tiller
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1beta1
kind: ClusterRoleBinding
metadata:
  name: tiller
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: tiller
    namespace: kube-system
    
# kubectl create -f rbac-config.yaml

{% endhighlight %}

**install Tiller**

在缺省配置下， Helm 会利用 "gcr.io/kubernetes-helm/tiller" 镜像在Kubernetes集群上安装配置 Tiller；并且利用 "https://kubernetes-charts.storage.googleapis.com" 作为缺省的 stable repository 的地址。由于在国内可能无法访问 "gcr.io", "storage.googleapis.com" 等域名，阿里云容器服务为此提供了镜像站点。所以最后helm的安装命令如下

{% highlight c %}
# helm init --service-account tiller --upgrade -i registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:v2.7.0 --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts
 
{% endhighlight %}

**install socat in all nodes**

{% highlight c %}
# helm version
Client: &version.Version{SemVer:"v2.7.0", GitCommit:"08c1144f5eb3e3b636d9775617287cc26e53dba4", GitTreeState:"clean"}
E1127 15:59:30.945567   17321 portforward.go:331] an error occurred forwarding 12490 -> 44134: error forwarding port 44134 to pod 958ca220447367061f361bb83969264591bdbadb6232e665aee1573743a6bc5d, uid : unable to do port forwarding: socat not found.


# wget http://ftp.tu-chemnitz.de/pub/linux/dag/redhat/el7/en/x86_64/rpmforge/RPMS/socat-1.7.2.4-1.el7.rf.x86_64.rpm 
# rpm -ivh socat-1.7.2.4-1.el7.rf.x86_64.rpm


## set ENV HELM_HOST=127.0.0.1


E0413 08:37:13.656896   18541 portforward.go:209] Unable to create listener: Error listen tcp6 [::1]:22972: socket: address family not supported by protocol
Client: &version.Version{SemVer:"v2.2.2", GitCommit:"1b330722aafcb8123114ae51f69d1e884a326f3e", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.2.2", GitCommit:"1b330722aafcb8123114ae51f69d1e884a326f3e", GitTreeState:"clean"}

{% endhighlight %}

**verfily(in k8s master)**

{% highlight c %}

# helm version

Client: &version.Version{SemVer:"v2.7.0", GitCommit:"08c1144f5eb3e3b636d9775617287cc26e53dba4", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.5.1", GitCommit:"7cf31e8d9a026287041bae077b09165be247ae66", GitTreeState:"clean"}

{% endhighlight %}

**upgrading Tiller**

{% highlight c %}

# export TILLER_TAG=v2.7.0        # Or whatever version you want
# kubectl --namespace=kube-system set image deployments/tiller-deploy tiller=registry.cn-hangzhou.aliyuncs.com/google_containers/tiller:$TILLER_TAG



# helm version
Client: &version.Version{SemVer:"v2.5.0", GitCommit:"08c1144f5eb3e3b636d9775617287cc26e53dba4", GitTreeState:"clean"}
Error: cannot connect to Tiller

{% endhighlight %}

- 未找到原因，之前用2.5的版本，卸载后换成2.7的。问题解决

**uninstall**

{% highlight c %}

$ helm delette --purge chart #完全删除
$ helm delete chart

{% endhighlight %}


**参考文章**

 - https://docs.helm.sh/using_helm/#service_account
 - https://github.com/kubernetes/helm/blob/master/docs/rbac.md
 - https://yq.aliyun.com/articles/159601