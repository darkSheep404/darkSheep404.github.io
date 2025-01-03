---
title: Sealos使用
date: 2024-09-23 15:20:18
createdTime: 24-02-23 (星期五) 13:39
updateTime: 25-01-03 (星期五) 17:17
---
https://cloud.sealos.io/

### 使用sealos选择nginx docker镜像部署

修改k8s部署的nginx image内部的nginx.conf来添加proxy_pass

需要在外部编写一个nginx.conf,覆盖掉镜像内部的nginx.conf

**在k8s中,可以使用把编写的nginx.conf放到 configMap内 随后用configMap的内容挂载到nginx容器内覆盖掉configMap**

> gui `新增配置文件`创建后 实际生成了`configMap`

`configmap.yaml`

```yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-test
data:
  etcvn-nginxvn-nginxvn-conf: |-
    user  nginx;
    worker_processes  auto;

    error_log  /var/log/nginx/error.log notice;
    pid        /var/run/nginx.pid;
...省略其他配置内容
```



`deployment.yml`

```yml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-test
  labels:
    app: nginx-test
spec:
  replicas: 1
  revisionHistoryLimit: 1
  selector:
    matchLabels:
      app: nginx-test
...
    spec:
      automountServiceAccountToken: false
      containers:
        - name: nginx-test
          image: nginx
      ...
          volumeMounts: #容器挂载配置
            - name: etcvn-nginxvn-nginxvn-conf #对应vloume名称
              mountPath: etc/nginx/nginx.conf #挂载到容器的的路径
              subPath: ./etc/nginx/nginx.conf #只挂载vloume中特定文件,而非整个目录 
      volumes:
        - name: etcvn-nginxvn-nginxvn-conf #volume 定义的卷的名称
          configMap: #vloume
            name: nginx-test #configMap的名称
            items:
              - key: etcvn-nginxvn-nginxvn-conf #configMap中数据项的key
                path: ./etc/nginx/nginx.conf #数据项在卷中路径
```

> 原gui 新建的名字即为挂载到容器中的路径,需要修改为etc/nginx/nginx.conf

`kubectl edit deployment nginx-test` 进入编辑模式

按`Insert` 进入编辑

`Esc` 退出编辑,进入命令模式 `:wq` 保存并退出 `:q` 不保存退出

