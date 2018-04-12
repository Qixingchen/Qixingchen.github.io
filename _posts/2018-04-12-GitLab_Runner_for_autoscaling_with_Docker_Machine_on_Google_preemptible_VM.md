---
layout: post
title: GitLab Runner for autoscaling with Docker Machine on Google preemptible VM
---

## 前言

### 目前状态

目前我们的 runner 跑在 Aliyun 与 开发者实体机上,Aliyun 服务器无法向 Google Play 提交应用,而开发者实体机上运行时会阻碍其他程序运行.而且在临近发版时,由于 Runner 数量不足,会造成编译阻塞影响发版节奏.

### 云服务

由于编译 Android 应用需要内存不少,主流云服务商报价都不低,例如 digitalocean 一台 2vCPU 4G 内存的机器需要 $20/mo,Google Cloud 上 2vCPU 4G 内存 机器报价为 $12.96/mo,Aliyun ECS 计算型 c5 2vCPU 4G 内存 每年报价为 ￥2393.40(约 $31.76 每月)

### 竞争实例与自伸缩

大部分云服务商都对"竞争实例"或者"抢占式实例"有大幅优惠.  
"竞争实例"的意义是,在服务商有机器设备空闲时,你才能申请到使用权,而且当服务器资源紧张时,你的设备会被强制收回.  
比较适合能够分布式处理或者运行时长较小的服务运行.  
GCE 的抢占式实例价格为正常实例的 20%,而且对于小于 10 分钟内就被 GCE 收回的时长不收取费用.

Google 的文档上声称 即使在最繁忙的时段,还是有部分抢占式实例可供申请,只要你的需求可以使用这种随时会被关闭机器的服务来完成,选择抢占式实例能够节约大量成本.Google 的数据表明 每个项目每7天的平均抢占率在5％和15％之间变化.我在运行时只遇到过一次抢占.

而 Gitlab 的 Runner 支持使用 Docker-Machine 来运行自伸缩实例,当有任务时在 GCE 上新建一个VM 机器,并在任务完成时删除机器.

## 那么怎么才能弄一个自伸缩的 runner 呢

首先呢 你要有一个用来管理扩展出来的runner的服务器,当然最好和runner在同一个内网.所以最好的解决方案就是直接在 GCE 上建一个实例啦.而且根据 GCE “始终免费” 政策,可以免费运行一个 f1-micro (1 vCPU 0.6G RAM 突发性能实例),完全足够作为管理服务器了.

按照 https://docs.gitlab.com/runner/executors/docker_machine.html 的引导,在这个管理机上安装 "GitLab Runner for autoscaling with Docker Machine" 顺带可以把 代理容器注册服务和缓存服务 ("proxy container registry and cache server")一起装在这个设备上.

下面是 `config.toml` (gitlab runner 配置文件)在使用 GCE 的示例  

``` yaml
[[runners]]
  name = "gce-main-scal"
  url = ""
  token = ""
  executor = "docker+machine"
  # 最大伸缩量
  limit = 3
  [runners.docker]
    tls_verify = false
    image = "ubnutu:16.04"
    privileged = false
    disable_cache = false
    volumes = ["/cache"]
    shm_size = 0
    # 缓存服务配置
  [runners.cache]
     Type = "s3"
     ServerAddress = "10.138.0.2:9005" #your minio-instance
     AccessKey = "" # minio AccessKey
     SecretKey = "" # minio Secret
     BucketName = "cache"
     Insecure = true
     Shared = true

  [runners.machine]
  # 配置见 https://docs.gitlab.com/runner/configuration/advanced-configuration.html#the-runners-machine-section
    IdleCount = 0
    IdleTime = 120
    MaxBuilds = 3
    MachineDriver = "google"
    MachineName = "ci-as-w-%s"
    MachineOptions = [
      # docker machine 配置 见 https://docs.docker.com/machine/drivers/gce/
      "google-project=jwb-android",
      "google-zone=us-west1-c",
      "google-machine-type=custom-2-5120",
      "google-machine-image=ubuntu-os-cloud/global/images/ubuntu-1604-xenial-v20180405",
      "google-tags=gitlab-ci-runner-worker,project-noip",
      # 抢占式实例
      "google-preemptible=true",
      "google-use-internal-ip=true",
      # 代理容器注册服务
      "engine-registry-mirror=http://10.138.0.2:6000"
    ]
```

然后在这个管理机上[安装 gcloud SDK](https://cloud.google.com/sdk/downloads) 并登入有创建 VM 实例权限的账号.

然后就要弄一个能编译所需项目的 docker 啦.嗯, Gitlab 的 autoscaling 只有 Docker Machine 模式,不能在创建出的 VM 上直接编译.如果不需要加入什么个性化的配置文件就能编译那就太好了,直接在 项目的 `.gitlab-ci.yml` 中的 image 上填入所需的镜像就可以开始了.  
然而我的项目需要加入一些保密的配置文件才能运行,在对比了几家私有 docker image 托管服务商之后我还是选择了 Google Container Registry ,一是和要使用的服务器处在同一个内网,下载更快(大概吧),另一个是 GCR 只需要支付很廉价的储存价格和出站流量(内网并不需要233)价格就可以了,不像其他托管商按照镜像数量收费,基本都是 $10/mo 起.  
按照 https://cloud.google.com/endpoints/docs/openapi/docker-image 推送了 docker 镜像,然后编译.结果遇到了 无权 pull  docker image 的错误.按照 https://docs.gitlab.com/ee/ci/docker/using_docker_images.html#define-an-image-from-a-private-container-registry ,将 docker login 信息在 gitlab 项目页面的 CI  secret variable 配置中添加 DOCKER_AUTH_CONFIG 即可.(是的 我用了一天的时间试图在 创建出来的 VM 上登入有权限的 Gcloud 用户 然后才发现可以直接放入 docker login 信息)

## 其他

如果在创建 Google VM 实例时申请了静态 IP 地址,记得在删除实例后去网络管理中把不需要的静态 IP 地址释放了.GCE 并不会在删除实例时自动释放,而且 IP 地址很贵(我的 75% 账单来自一个忘记释放的 IP 地址 ORZ )

如果你需要在编译的 Docker 放什么全局配置文件,有个小提示, Docker 运行时的默认登入用户是 Root.

## 结束

目前设定最多同时会有3个 runner 进行编译,目前平均每天相关费用不到 $0.05.  
后续跟进下 Google container builder,每天提供了 120 分钟的免费基础编译时间,看看能不能与目前的开发模式相容.

### 参考

gitlab 官方迁移到 GCP 相关 ISSUR:  
https://gitlab.com/gitlab-org/gitlab-runner/issues/3023  
https://gitlab.com/gitlab-com/infrastructure/issues/3131  

Gitlab 相关文档:  
[Runners autoscale configuration](https://docs.gitlab.com/runner/configuration/autoscale.html)  
[Install your own container registry and cache server](https://docs.gitlab.com/runner/install/registry_and_cache_servers.html)  
[Install and register GitLab Runner for autoscaling with Docker Machine](https://docs.gitlab.com/runner/executors/docker_machine.html)  
[Define an image from a private Container Registry](https://docs.gitlab.com/ee/ci/docker/using_docker_images.html#define-an-image-from-a-private-container-registry)  
[Using Docker Build](https://gitlab.com/help/ci/docker/using_docker_build.md)  
[GitLab 8.16 released with auto deploy on Google Container Engine and Prometheus monitoring as part of GitLab](https://about.gitlab.com/2017/01/22/gitlab-8-16-released/)  
  
    
docker 文档:  
[Docker Machine drivers:Google Compute Engine](https://docs.docker.com/machine/drivers/gce/)  
[Quickstart for Container Registry](https://cloud.google.com/container-registry/docs/quickstart)  
  
  
  
  
Google Cloud 文档:  

[Building a Docker Image](https://cloud.google.com/endpoints/docs/openapi/docker-image)  
[Container Registry 价格](https://cloud.google.com/container-registry/pricing)  
[Container Registry Advanced Authentication Methods](https://cloud.google.com/container-registry/docs/advanced-authentication)  
[Google Compute Engine 价格](https://cloud.google.com/compute/pricing)  
[Google Cloud SDK install](https://cloud.google.com/sdk/downloads)  
[Creating and Starting a Preemptible VM Instance](https://cloud.google.com/compute/docs/instances/create-start-preemptible-instance)  
[CONTAINER BUILDER](https://cloud.google.com/container-builder/?hl=zh-cn)
[Cloud Container Builder Automating Builds using Build Triggers](https://cloud.google.com/container-builder/docs/running-builds/automate-builds)  
[Container Registry Pushing and Pulling Images](https://cloud.google.com/container-registry/docs/pushing-and-pulling)  
[Preemptible VMs now up to 33% cheaper](https://cloudplatform.googleblog.com/2016/08/Preemptible-VMs-now-up-to-33-percent-cheaper.html)  

其他文章:  

[GitLab CI Runner auf Google Compute Engine betreiben](https://blog.marvin-menzerath.de/artikel/gitlab-ci-runner-auf-google-compute-engine-betreiben/)  
[Auto-scaling Gitlab Runner in GCE](https://github.com/jerryjj/gitlab-runner-gce)  
[Autoscaling Gitlab-CI builds on preemptible Google-Cloud instances](https://webnugget.de/autoscaling-gitlab-ci-builds-on-preemptible-google-cloud-instances-2/)



