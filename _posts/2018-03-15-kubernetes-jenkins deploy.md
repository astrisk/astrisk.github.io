---
title:  "容器化DEVOPS-弹性CI平台系列之部署和配置Jenkins master"
date:   2018-03-15 16:37:00
categories: kubernetes-docker
---

在之前的文章中，我们已经构建好了Jenkins的相关docker images,并且推送到私有register中。现在我们把Jenkins部署到kubernetes集群中。

**创建namespace**


{% highlight c %}
# vim jenkins-namespaces.yaml

apiVersion: v1
kind: Namespace
metadata:
   name: jenkins
   labels:
     name: jenkins

# vim jenkins-services.yaml

---
  kind: Service
  apiVersion: v1
  metadata:
    name: jenkins
    namespace: jenkins
  spec:
    type: NodePort
    selector:
      name: jenkins-leader
    ports:
      - protocol: TCP
        port: 8080
        targetPort: 8080
        nodePort: 38080
        name: ui
      - protocol: TCP
        port: 50000
        targetPort: 50000
        name: slaves


# vim jenkins-deployment.yaml

apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: jenkins-leader
  labels:
    name: jenkins-leader
  namespace: jenkins
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: jenkins-leader
    spec:
      containers:
      - name: jenkins-leader
        image: fangruo/jenkins-master-apline-base
        command:
        - /usr/local/bin/jenkins.sh
        ports:
        - name: ui
          containerPort: 8080
        - name: service
          containerPort: 50000
        volumeMounts:
        - mountPath: /var/jenkins_home
          name: jenkinshome
      volumes:
        - name: jenkinshome
          nfs:
            server: X.X.X.X # 把Jenkins的数据存储到nfs中
            path: "/home/nfs/jenkins/data" 

{% endhighlight %}


**部署Jenkins master**

{% highlight c %}

# kubectl create -f jenkins-namespaces.yaml
# kubectl create -f jenkins-services.yaml
# kubectl create -f jenkins-deployment.yaml

# kubectl get pods -n jenkins 

NAME                              READY     STATUS    RESTARTS   AGE
jenkins-leader-2258711727-sfq13   1/1       Running   1          90d

{% endhighlight %}

**登录Jenkins UI**
- 在浏览器输入 http://nodeip:38080/ 登录Jenkins master UI
- 后面会介绍Jenkins UI上进行相关配置和集成kubernetes,实现动态伸缩的分布式Jenkins cluster