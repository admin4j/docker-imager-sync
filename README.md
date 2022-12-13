# docker-image-syncer

无论是在学习k8s还是正式环境部署k8s中,第一步安装k8难倒了各大英雄好汉。原因是k8s 各种组件镜像在谷歌服务器上(k8s.gcr.io)，而我们有墙的存在，所以会经常性的下载失败。解决办法是搭梯子，或者是使用其他镜像源。

本仓库使用  [aliyun image-syncer](https://github.com/AliyunContainerService/image-syncer) 配合 github action 同步 k8s docker镜像(k8s.gcr.io)  到 dockerhub。提高k8s docker镜像(k8s.gcr.io)成功率，解决云原生第一大难题。

docker-image-syncer 运行原理

1. `docker pull` 下拉所需镜像

   > 由于github action 运行再国外的github服务器的，没有qiang一说，docker pull 是很方便的

2. `docker tag` 修改镜像tag

3. `docker push` 推送镜像到相应docker register

# Getting Started

## 1. 方式一

提交 PR ，和合并到 main 分支之后，自动执行github action同步到 dockerhub

![image-20221213101819817](https://img-blog.csdnimg.cn/img_convert/4f42a4d7ef8e167aeeb386b530e1bf33.png#pic_center)

## 2. 方式二

2.1、fork 这个仓库, 创建你自己的docker register 账号密码:


- 1. Settings
- 2. Secrets
- 3. New Repository Secrets
- 4. Add your DOCKER_USERNAME and DOCKER_PASSWORD key values.

![image-20221213102110118](https://img-blog.csdnimg.cn/img_convert/de478aaf77041569c82f17cd34834926.png#pic_center)

2.2、修改`images.json` 文件，改成你需要的

```
{
  "quay.io/coreos/kube-rbac-proxy": "admin4j/kube-rbac-proxy",
  "k8s.gcr.io/metrics-server/metrics-server": "admin4j/metrics-server",
  "k8s.gcr.io/ingress-nginx/controller": "admin4j/ingress-nginx-controller",
  "k8s.gcr.io/git-sync/git-sync": "admin4j/git-sync",
  "gcr.io/kaniko-project/executor:debug,latest": "admin4j/kaniko-executor",
  "k8s.gcr.io/kube-state-metrics/kube-state-metrics": "admin4j/kube-state-metrics",
  "k8s.gcr.io/sig-storage/nfs-subdir-external-provisioner": "admin4j/nfs-subdir-external-provisioner",
  "k8s.gcr.io/prometheus-adapter/prometheus-adapter": "admin4j/prometheus-adapter",
  "k8s.gcr.io/kube-apiserver": "admin4j/kube-apiserver",
  "k8s.gcr.io/kube-controller-manager": "admin4j/kube-controller-manager",
  "k8s.gcr.io/kube-scheduler": "admin4j/kube-scheduler",
  "k8s.gcr.io/kube-proxy": "admin4j/kube-proxy",
  "k8s.gcr.io/pause": "admin4j/pause",
  "k8s.gcr.io/etcd": "admin4j/etcd",
  "k8s.gcr.io/coredns/coredns": "admin4j/coredns"
}
```

修改`auth.json`，可以添加其他 docker register源

```yaml
{
  "registry.hub.docker.com": {
    "username": "${USERNAME}",
    "password": "${PASSWORD}"
  }
}
```

2.3、检查 action logs

![](https://img-blog.csdnimg.cn/img_convert/b72068c934fbc4394675e66e0b28e8c8.png#pic_center)

# Action 执行完后,检查成果

dockerhub

![image-20221213102633683](https://img-blog.csdnimg.cn/img_convert/7d01e06938461c646c1354b2bdc3f383.png#pic_center)

# k8s使用镜像

1. 方式一

```
# 在安装kubernetes集群之前，必须要提前准备好集群需要的镜像，所需镜像可以通过下面命令查看
[root@master ~]# kubeadm config images list

# 下载镜像
# 此镜像在kubernetes的仓库中,由于网络原因,无法连接，下面提供了一种替代方案
images=(
    kube-apiserver:v1.23.15
	kube-controller-manager:v1.23.15
	kube-scheduler:v1.23.15
	kube-proxy:v1.23.15
	pause:3.6
	etcd:3.5.1-0
	coredns/coredns:v1.8.6
)

for imageName in ${images[@]} ; do
	docker pull admin4j/$imageName
	docker tag admin4j/$imageName 		k8s.gcr.io/$imageName
	docker rmi admin4j/$imageName
done

```

2. 方式二

   直接修改 yml 部署文件的 image 属性
