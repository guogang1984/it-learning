# k8s.md

## TODO

## 知识点：自动化部署 kube
https://github.com/easzlab/kubeasz/blob/master/docs/setup/00-planning_and_overall_intro.md

## 知识点：Rancher2 部署

## 
## 1 kind: Deployment 、Service、 Pod 等等概念
## 2 volume： pv pvc 概念

## ruoyi 配置示例
````yaml
###### nacos服务
apiVersion: apps/v1                       #指定api版本
kind: Deployment                          #指定创建资源的角色/类型，Deployment表示控制器
metadata:                                 #定义资源的元数据/属性
  annotations:
    description: devops-nacos服务
  labels:
    appgroup: ''                          #定义资源的标签
  name: devops-nacos                      #资源的名字，在同一个namespace中必须唯一
  namespace: base-app            
spec:                                     #指定该资源的内容
  replicas: 1                             #指定副本数
  selector:                               #选择器
    matchLabels:                          #匹配标签
      app: devops-nacos                   #匹配模板名称
  template:
    metadata:
      labels:
        app: devops-nacos
    spec:                                   #定义容器模板
      containers:                           #定义容器信息
        - name: container-devops-nacos      #容器名，符合label名
          image: swr.cn-north-4.myhuaweicloud.com/chinasoft_csi_paas/devops-nacos:latest              #容器使用的镜像以及版本
          env:
            - name: MODE
              value: "standalone"
            - name: SPRING_DATASOURCE_PLATFORM
              value: "mysql"
            - name: MYSQL_SERVICE_HOST
              value: "devops-mysql57.default.svc.cluster.local"
            - name: MYSQL_SERVICE_PORT
              value: "3306"
            - name: MYSQL_SERVICE_DB_NAME
              value: "nacos"
            - name: MYSQL_SERVICE_USER
              value: "root"
            - name: MYSQL_SERVICE_PASSWORD
              value: "root"
          resources:
            limits:
              cpu: 250m
              memory: 2Gi
            requests:
              cpu: 250m
              memory: 2Gi
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: Always
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
      imagePullSecrets:
        - name: default-secret
        - name: default-secret
      affinity: {}
      schedulerName: default-scheduler
      tolerations:
        - key: node.kubernetes.io/not-ready
          operator: Exists
          effect: NoExecute
          tolerationSeconds: 300
        - key: node.kubernetes.io/unreachable
          operator: Exists
          effect: NoExecute
          tolerationSeconds: 300
      dnsConfig:
        options:
          - name: timeout
            value: ''
          - name: ndots
            value: '5'
          - name: single-request-reopen
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600

################

kind: Service
apiVersion: v1
metadata:
  name: devops-nacos
  namespace: base-app
  resourceVersion: ''
  labels:
    app: devops-nacos
    name: devops-nacos
spec:
  ports:
    - name: cce-service-0
      protocol: TCP
      port: 8848
      targetPort: 8848
      nodePort: 30848
  selector:
    app: devops-nacos
  clusterIP: ''
  type: NodePort
  sessionAffinity: None
  externalTrafficPolicy: Cluster
status:
  loadBalancer: {}

````
````yaml
###### ruoyi fontend服务

apiVersion: apps/v1                       #指定api版本
kind: Deployment                          #指定创建资源的角色/类型，Deployment表示控制器
metadata:                                 #定义资源的元数据/属性
  annotations:
    description: devops-fontend服务
  labels:
    appgroup: ''                          #定义资源的标签
  name: devops-fontend                    #资源的名字，在同一个namespace中必须唯一
  namespace: base-app            
spec:                                     #指定该资源的内容
  replicas: 1                             #指定副本数
  selector:                               #选择器
    matchLabels:                          #匹配标签
      app: devops-fontend                 #匹配模板名称
  template:
    metadata:
      labels:
        app: devops-fontend
    spec:                                   #定义容器模板
      containers:                           #定义容器信息
        - name: container-fontend           #容器名，符合label名
          image: swr.cn-north-4.myhuaweicloud.com/chinasoft_csi_paas/devops-frontend:base-app            #容器使用的镜像以及版本
          env:
            - name: DEVOPS_DISCOVERY_MIDDLEWARE_HOST
              value: devops-nacos.base-app.svc.cluster.local
          resources:
            limits:
              cpu: 250m
              memory: 2Gi
            requests:
              cpu: 250m
              memory: 2Gi
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: Always
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
      imagePullSecrets:
        - name: default-secret
        - name: default-secret
      affinity: {}
      schedulerName: default-scheduler
      tolerations:
        - key: node.kubernetes.io/not-ready
          operator: Exists
          effect: NoExecute
          tolerationSeconds: 300
        - key: node.kubernetes.io/unreachable
          operator: Exists
          effect: NoExecute
          tolerationSeconds: 300
      dnsConfig:
        options:
          - name: timeout
            value: ''
          - name: ndots
            value: '5'
          - name: single-request-reopen
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600

################

kind: Service
apiVersion: v1
metadata:
  name: devops-fontend
  namespace: base-app
  resourceVersion: ''
  labels:
    app: devops-fontend
    name: devops-fontend
spec:
  ports:
    - name: cce-service-0
      protocol: TCP
      port: 80
      targetPort: 80
      nodePort: 31888
  selector:
    app: devops-fontend
  clusterIP: ''
  type: NodePort
  sessionAffinity: None
  externalTrafficPolicy: Cluster
status:
  loadBalancer: {}


````

````yaml
###### ruoyi 网关配置示例

apiVersion: apps/v1                       #指定api版本
kind: Deployment                          #指定创建资源的角色/类型，Deployment表示控制器
metadata:                                 #定义资源的元数据/属性
  annotations:
    description: devops-gateway服务
  labels:
    appgroup: ''                          #定义资源的标签
  name: devops-gateway                    #资源的名字，在同一个namespace中必须唯一
  namespace: base-app            
spec:                                     #指定该资源的内容
  replicas: 1                             #指定副本数
  selector:                               #选择器
    matchLabels:                          #匹配标签
      app: devops-gateway                 #匹配模板名称
  template:
    metadata:
      labels:
        app: devops-gateway
    spec:                                 #定义容器模板
      containers:                         #定义容器信息
        - name: container-gateway         #容器名，符合label名
          image: swr.cn-north-4.myhuaweicloud.com/chinasoft_csi_paas/devops-cloud:latest  #容器使用的镜像以及版本
          env:
            - name: DEVOPS_DISCOVERY_MIDDLEWARE_HOST
              value: devops-nacos.base-app.svc.cluster.local
            - name: DIST_JAR
              value: /opt/ruoyi-gateway-2.0.0.jar
          resources:
            limits:
              cpu: 250m
              memory: 2Gi
            requests:
              cpu: 250m
              memory: 2Gi
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: Always
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
      imagePullSecrets:
        - name: default-secret
        - name: default-secret
      affinity: {}
      schedulerName: default-scheduler
      tolerations:
        - key: node.kubernetes.io/not-ready
          operator: Exists
          effect: NoExecute
          tolerationSeconds: 300
        - key: node.kubernetes.io/unreachable
          operator: Exists
          effect: NoExecute
          tolerationSeconds: 300
      dnsConfig:
        options:
          - name: timeout
            value: ''
          - name: ndots
            value: '5'
          - name: single-request-reopen
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600

################ service

kind: Service
apiVersion: v1
metadata:
  name: devops-gateway
  namespace: base-app
  resourceVersion: ''
  labels:
    app: devops-gateway
    name: devops-gateway
spec:
  ports:
    - name: cce-service-0
      protocol: TCP
      port: 8080
      targetPort: 8080
      nodePort: 31080
  selector:
    app: devops-gateway
  clusterIP: ''
  type: NodePort
  sessionAffinity: None
  externalTrafficPolicy: Cluster
status:
  loadBalancer: {}

````

````yaml
###### ruoyi auth服务配置示例

apiVersion: apps/v1                       #指定api版本
kind: Deployment                          #指定创建资源的角色/类型，Deployment表示控制器
metadata:                                 #定义资源的元数据/属性
  annotations:
    description: devops-auth服务
  labels:
    appgroup: ''                          #定义资源的标签
  name: devops-auth                       #资源的名字，在同一个namespace中必须唯一
  namespace: base-app            
spec:                                     #指定该资源的内容
  replicas: 1                             #指定副本数
  selector:                               #选择器
    matchLabels:                          #匹配标签
      app: devops-auth                 #匹配模板名称
  template:
    metadata:
      labels:
        app: devops-auth
    spec:                                   #定义容器模板
      containers:                           #定义容器信息
        - name: container-auth           #容器名，符合label名
          image: swr.cn-north-4.myhuaweicloud.com/chinasoft_csi_paas/devops-cloud:latest             #容器使用的镜像以及版本
          env:
            - name: DEVOPS_DISCOVERY_MIDDLEWARE_HOST
              value: devops-nacos.base-app.svc.cluster.local
            - name: DIST_JAR
              value: /opt/ruoyi-auth-2.0.0.jar
          resources:
            limits:
              cpu: 250m
              memory: 2Gi
            requests:
              cpu: 250m
              memory: 2Gi
          terminationMessagePath: /dev/termination-log
          terminationMessagePolicy: File
          imagePullPolicy: Always
      restartPolicy: Always
      terminationGracePeriodSeconds: 30
      dnsPolicy: ClusterFirst
      securityContext: {}
      imagePullSecrets:
        - name: default-secret
        - name: default-secret
      affinity: {}
      schedulerName: default-scheduler
      tolerations:
        - key: node.kubernetes.io/not-ready
          operator: Exists
          effect: NoExecute
          tolerationSeconds: 300
        - key: node.kubernetes.io/unreachable
          operator: Exists
          effect: NoExecute
          tolerationSeconds: 300
      dnsConfig:
        options:
          - name: timeout
            value: ''
          - name: ndots
            value: '5'
          - name: single-request-reopen
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxUnavailable: 0
      maxSurge: 1
  revisionHistoryLimit: 10
  progressDeadlineSeconds: 600

################ service

kind: Service
apiVersion: v1
metadata:
  name: devops-auth
  namespace: base-app
  resourceVersion: ''
  labels:
    app: devops-auth
    name: devops-auth
spec:
  ports:
    - name: cce-service-0
      protocol: TCP
      port: 9200
      targetPort: 9200
  selector:
    app: devops-auth
  clusterIP: ''
  type: ClusterIP
  sessionAffinity: None
status:
  loadBalancer: {}

````