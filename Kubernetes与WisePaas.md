# Kubernetes与WisePaas

​             

k8s与wisepaas的使用笔记

| 修订人     | 版本 | 时间       | 备注 |
| ---------- | ---- | ---------- | ---- |
| jinwei.wei | v1.0 | 2021-08-13 | 初始 |



## kubectl的安装

[安装工具 | Kubernetes](https://kubernetes.io/zh/docs/tasks/tools/)  参考官网

从wise-paas上下载配置文件，并放到.kube目录中，关于多个kubeConfig的切换和使用参考如下链接

[配置对多集群的访问 | Kubernetes](https://kubernetes.io/zh/docs/tasks/access-application-cluster/configure-access-multiple-clusters/)

常用命令

```shell
export KUBECONFIG=~/.kube/config1:~/.kube/config2  载入多个配置（Linux）

$Env:KUBECONFIG="$Env:KUBECONFIG;$HOME\.kube\config" 载入多个配置（Windows）
  
kubectl config current-context     当前使用的环境

kubectl config get-contexts   可以使用的环境

kubectl config use-context [需要使用的context]

kubectl config view  查看配置
  
```



## kubectl常用命令

参考：
[命令说明 | Kubernetes](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands)


其最基本的语法格式为kubectl [command] [TYPE] [NAME] [flags]，其中各部分的简要说明如下。

- command：对资源执行相应操作的子命令，例如get、create、delete、run等；常用的核心子命令如表2-3所示。

- TYPE：要操作的资源类型，例如pods、services等；类型名称大小写敏感，但支持使用单数、复数或简写格式。

- NAME：要操作的资源对象名称，大小写敏感；省略时，则表示指定TYPE的所有资源对象；同一类型的资源名称可于TYPE后同时给出多个，也可以直接使用TYPE/NAME的格式来为每个资源对象分别指定类型。

- flags：命令行选项，例如-s或--server等；另外，get等命令在输出时还有一个常用的标志-o <format>用于指定输出格式

  

### kubectl get 

作用：查询信息

```shell
$ kubectl get [(-o|--output=)json|yaml|name|go-template|go-template-file|template|templatefile|jsonpath|jsonpath-as-json|jsonpath-file|custom-columns-file|custom-columns|wide] (TYPE[.VERSION][.GROUP] [NAME | -l label] | TYPE[.VERSION][.GROUP]/NAME ...) [flags]
```

常用type：pod,svc,ing,deploy,all

常用flags：

-o wide 资源附加信息

-o yaml 以yaml方式输出信息（更详细）

-o json  类似

-n [空间名称] 命名空间（重要，很多时候获取不到是没指定这个）

-w 持续监控

示例:

```
kubectl get pod -n e100 -w  #查看命名空间e100下的pod 持续监控
kubectl get ing -n e100   #查看命名空间e100下的ingress
kubectl get all -n e100   #查看命名空间e100下的pod/svc/deploy
```



### kubectl describe

作用:详细描述信息，一般是部署信息

```shell
$ kubectl describe (-f FILENAME | TYPE [NAME_PREFIX | -l label] | TYPE/NAME)
```

常用type：pod

常用flags：-n [空间名称] 命名空间

示例：

```
kubectl describe pod ihcc-system-vk-b49bbb979-659c8 -n e100 #查看pod的详细描述
```



### kubectl logs

作用:日志查看

```sh
$ kubectl logs [-f] [-p] (POD | TYPE/NAME) [-c CONTAINER]
```

常用type：pod

常用flags：

-c [容器名称] 指定pod终端容器，如果pod中只有一个容器可以不写

-n [空间名称] 指定命名空间

-f 持续监控日志

示例：

```
kubectl logs ihcc-system-vk-b49bbb979-659c8 -n e100
```



### kubectl exec

作用：进入容器

```sh
$ kubectl exec (POD | TYPE/NAME) [-c CONTAINER] [flags] -- COMMAND [args...]
```

常用type：pod

常用flags：

-it 进入容器（Pass stdin to the container，Stdin is a TTY）

-c [容器名称]：多容器时使用

常用COMMAND：-- /bin/sh

示例：

```sh
kubectl exec -it ihcc-system-vk-b49bbb979-659c8 -n e100 -- /bin/sh  #进入到容器，想退出时输入exist并回车
```



### kubectl delete

作用：删除资源，请慎用

```sh
$ kubectl delete ([-f FILENAME] | [-k DIRECTORY] | TYPE [(NAME | -l label | --all)])
```

 常用方式：

-f 指定depoly文件

示例：

```
kubectl apply -f nacos/  #应用本地nacos文件内的所有deploy文件创建资源
kubectl delete -f nacos/ #删除对应deploy文件创建的资源
```

### kubectl apply/create

作用：创建资源

```sh
$ kubectl apply (-f FILENAME | -k DIRECTORY)
```

常用方式：

建议apply方式，可以用于创建和更新。

示例

```sh
kubectl apply -f nacos/  #应用本地nacos文件内的所有deploy文件创建资源
kubectl apply -f deployment.yaml  #使用deployment.yaml文件创建资源
```

### kubectl top

作用：资源占用查看

常用type：pod，node

常用flags：-n [空间名称] 命名空间

示例：

```sh
kubectl top pod -n e100 #查看e100空间下的pod资源占用情况
```

## docker镜像打包

常用命令

```sh
docker build -t vkwei/ihcc_file:v1 .   #使用当前目录的Dockerfile创建镜像
docker pull vkwei/ihcc_file:v1       #拉取镜像
docker push vkwei/ihcc_file:v1       #推送镜像到dockerHub，注意tag是有规则限制的vkwei是账户名，其它类似harbor需要用host名
docker tag ubuntu:15.10 runoob/ubuntu:v3  #为镜像打标签
```

Dockfile示例1

```yaml
FROM java:8
ADD ./target/*.jar /app.jar
EXPOSE 9300
ENTRYPOINT ["java","-Xms512m","-Xmx1024m","-jar","/app.jar"]  
#注意，在镜像资源有限时，尽量配置Java堆参数，否则容易oom。
```

Dockfile示例2

```sh
FROM nginx
COPY dist/ /usr/share/nginx/html/
COPY nginx/default.conf /etc/nginx/conf.d/default.conf
#拷贝本地nginx目录下的配置文件替换镜像的默认配置文件
```





## k8s的yaml文件

示例与说明：

示例1

示例中将所有配置放在同一个yaml中，也可以分开配置，建议同一个文件管理。

示例中的pod包含两个容器，nginx和springboot，两个容器公用一个存储卷。

```yaml
#svc配置部分
---
apiVersion: v1  #固定
kind: Service   #固定
metadata:
  name: ihcc-file-vk #名称
  namespace: e100   #命名空间
spec:
  type: ClusterIP  #类型，基本以这个为主
  selector:    #选择标签，可以用多个，需要跟deploy配对，这里用了两个
    component: web-file  
    author: vk-wei
  ports:
    - port: 9300  #映射端口
      name: up-api #配置多个端口时需填写，一个端口时可以不填写
      targetPort: 9300 #目标端口
    - port: 80
      name: nginx
      targetPort: 80
---
#deploy配置部分
apiVersion: apps/v1 #固定
kind: Deployment    #固定
metadata:
  name: ihcc-file-vk #名称
  namespace: e100    #命名空间
spec:
  replicas: 1        #集群实例个数
  revisionHistoryLimit: 0 #定义保留的升级记录数，配合回滚
  selector:          #标签选择器，需要跟template设置匹配
    matchLabels:
      component: web-file
      author: vk-wei
  template:
    metadata:     
      labels:    #pod的标签
        component: web-file
        author: vk-wei
    spec:
      volumes:   #挂载卷，在wisePaas中配置pv和pvc后可用
        - name: volume
          persistentVolumeClaim:
            claimName: ihcc-file-store  #需与wisePaas中设置的相同
            readOnly: false             #可读写，否则只能读
      containers:
        - name: nginx     #容器1，
          image: vkwei/nginx:v2   #镜像tag
          ports:
            - containerPort: 80   #映射端口
          volumeMounts:           #挂载卷，name需和volumes中定义的对应
            - name: volume
              mountPath: /usr/share/nginx/html  #挂载到容器的目录
          resources:           
            limits:              #资源限制
              cpu: 500m
              memory: 2Gi
              ephemeral-storage: 256Mi
            requests:
              cpu: 200m
              memory: 1Gi
        - name: ihcc-file-vk  #容器2
          image: vkwei/ihcc_file:v2 #镜像tag
          imagePullPolicy: Always   #拉取策略，在dev环境下每次都重新拉取镜像
          volumeMounts:             #挂载卷，name需和volumes中定义的对应
            - name: volume
              mountPath: /opt/ihcc-file/uploadFile/ #挂载到容器的目录
          ports: 
            - containerPort: 9300   #映射端口
              protocol: TCP         #默认TCP，可以省略
          resources:
            limits:                 #资源限制，注意不要小于docker中为jvm配置的堆大小
              cpu: 500m
              memory: 3Gi
              ephemeral-storage: 256Mi
            requests:
              cpu: 200m
              memory: 2Gi
---
#ingress路由配置
apiVersion: extensions/v1beta1   #固定
kind: Ingress                    #固定
metadata:
  name: ihcc-file-vk             
  namespace: e100
    # annotations:           #一些ingress的路由规则，某些场景下需要
    # kubernetes.io/ingress.class: nginx
  # nginx.ingress.kubernetes.io/rewrite-target: /$1
spec:
#这里配置两个域名分给两个容器，一个是nginx的文件内容查看，另一个是文件上传服务，如果某个容器不需要外部访问，可以去掉。
  rules:
    - host: ihcc-file-vk.e100.ensaas.en.internal   #内部域名1，需要遵守wisePaas的规则
      http:
        paths:
          - path: /
            backend:
              serviceName: ihcc-file-vk    #svc的名称，注意要匹配
              servicePort: 9300
    - host: ihcc-file-niginx-vk.e100.ensaas.en.internal #内部域名2，需要遵守wisePaas的规则
      http:
        paths:
          - path: /
            backend:
              serviceName: ihcc-file-vk
              servicePort: 80
```

示例2

前端yaml



```yaml
---
#svc配置
kind: Service  #固定
apiVersion: v1 #固定
metadata:
  name: ihcc-web-nginx-vk
  namespace: e100
  labels:
    app: ihcc-web-nginx
spec:
  selector:            #标签选择器
    component: ihcc-web-nginx-vk  
    author: vk-wei
  type: ClusterIP
  ports:
    - port: 80
      targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ihcc-web-nginx-vk
  namespace: e100
spec:
  replicas: 1
  selector:
    matchLabels:
      component: ihcc-web-nginx-vk
      author: vk-wei
  template:
    metadata:
      labels:
        component: ihcc-web-nginx-vk
        author: vk-wei
    spec:
      volumes:     #挂载卷
       - name: volume
         persistentVolumeClaim:
           claimName: ihcc-file-store #需与wisepaas配置匹配
           readOnly: false
      containers:
      - name: ihcc-web-nginx-vk
        image: vkwei/ihcc_web:v6  #拉取的镜像
        imagePullPolicy: Always
        volumeMounts:
          - name: volume
            mountPath: /opt/ihcc-file/uploadFile/
        ports:
        - containerPort: 80
        resources:
          requests:
            memory: 1Gi
            cpu: 200m
          limits:
            memory: 2Gi
            cpu: 500m
            ephemeral-storage: 500Mi
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:   
  annotations:  #路由规则，将以/prod-api/开头的请求都转发的gateway服务上，其它的请求转发到web服务上
    nginx.ingress.kubernetes.io/configuration-snippet: |
      rewrite /prod-api/(.*)  /$1 break;
  name: ihcc-web-nginx-vk
  namespace: e100
spec:
  rules:
    - host: ihcc-web.e100.ensaas.en.internal
      http:
        paths:
          - path: /
            backend:
              serviceName: ihcc-web-nginx-vk  #与svc名称匹配
              servicePort: 80
          - path: /prod-api/
            backend:
              serviceName: ihcc-gateway-vk   #与是v从名称匹配
              servicePort: 8080

```

## k8s与harbor

在k8s中配置密钥：

```sh
kubectl create secret docker-registry [密钥名称] \ #harbor-secret
  --docker-server=<你的镜像仓库服务器> \  #https://harbor.arfa.wise-paas.com
  --docker-username=<你的用户名> \       #jinwei.wei
  --docker-password=<你的密码> \         #123456
  --docker-email=<你的邮箱地址>\          #jinwei.wei@advantech.com.cn
  --namespace=<wisePaas中的使用的命名空间> #e100 注意一定要一致，不然拉取不到
```

在deployment.yaml中配置

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: private-reg
spec:
  containers:
  - name: private-reg-container
    image: <your-private-image>
  imagePullSecrets:             #配置密钥
  - name: harbor-secret         #和k8s密钥名称匹配
```

