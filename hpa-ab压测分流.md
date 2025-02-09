# hpa-ab压测

`ab` 是 Apache HTTP 服务器提供的一个基准测试工具，用来对Web服务器进行性能测试。

## ab常用选项

- `-n requests`：执行指定数量的请求。这个参数定义了整个测试过程中将要发出的请求数量。
- `-c concurrency`：同时发起的并发请求数。此参数决定了在任意给定时间点有多少个请求是并行处理的。
- `-t timelimit`：设置测试的最大持续时间（秒）。一旦达到这个时间限制，即使没有完成所有请求也会停止测试。
- `-s timeout`：设置每个连接的超时时间（秒）。默认情况下，如果服务器在一段时间内没有响应，`ab` 将认为请求失败。
- `-k`：启用HTTP KeepAlive特性。这意味着同一个TCP连接可以被用来发送多个HTTP请求，从而减少建立新连接所需的时间开销。
- `-S`：忽略SSL证书错误。对于HTTPS请求，如果你不想因为无效的SSL证书而终止测试，可以使用这个选项。

- `-l`：接受非幂等的HTTP响应码（如5xx）。默认情况下，`ab`会在遇到这类状态码时立即停止。
- `-V`：显示版本信息并退出。

运行`ab`命令后，它会返回一系列的结果，其中包括但不限于：

- **Complete requests**：成功完成的请求数。
- **Failed requests**：失败的请求数。
- **Non-2xx responses**：不是2xx系列的成功HTTP响应的数量。
- **Requests per second**：每秒处理的请求数，即吞吐率。
- **Time per request**：平均每个请求花费的时间（毫秒）。
- **Transfer rate**：传输速率（KB/sec），不包括HTTP头部。

![image-20250124145132278](https://github.com/user-attachments/assets/c7182519-c5c2-406d-ac26-9d2a86cb5f51)


ab显示参数：

`server software`：服务器端提供的服务

`server hostname`：提供服务的主机

`server port`：服务端口

`document path`：访问资源所在路径

`document length`： 响应http请求的内容大小（单位:byte）

`concurrency level`：并发数

`time taken for tests`：表示完成所有指定请求数所需的总时间（s）

`complate requests`：完成的请求数

`fail requests`：失败的请求数

`Total transferred`：表示在测试期间从服务器传输到客户端的总数据量（单位：byte）

`HTML transferred`：表示在HTTP响应中传输的HTML文档的大小

`Requests per second`：表示在给定的时间段内，系统能够处理的HTTP请求的数量（ms）

`Time per request` 通常会以两种形式展示：

**单个请求时间 (Time per request)**：

- **55.987 [ms] (mean)**：这是从客户端视角来看，每个请求平均花费的时间，即如果忽略并发性，每个请求大约需要55.987毫秒才能完成。这个值反映了在没有其他并发请求的情况下，服务器处理单个请求所需的平均时间。

**跨所有并发请求的时间 (Time per request, across all concurrent requests)**：

- **0.560 [ms] (mean, across all concurrent requests)**：这个值考虑了并发性，表示当多个请求同时进行时，每个请求实际经历的平均等待时间。由于并发请求共享资源和处理时间，所以从个体请求的角度来看，它们似乎更快地得到了响应。

`Transfer rate`：表示在测试期间从服务器传输到客户端的数据速率。这个数值通常以每秒千字节（Kbytes/sec）为单位

`Connection Times (ms)`：提供了关于每个HTTP请求连接和处理时间的详细统计信息。这些数据以毫秒（ms）为单位

**Connect**：

- **min**: 最短连接时间是0毫秒。
- **mean[+/-sd]**: 平均连接时间为4毫秒，标准差为3.4毫秒。
- **median**: 中位数连接时间为4毫秒。
- **max**: 最长连接时间是33毫秒。

这个阶段表示客户端与服务器建立TCP连接所需的时间

**Processing**：

- **min**: 最短处理时间是3毫秒。
- **mean[+/-sd]**: 平均处理时间为51毫秒，标准差为39.9毫秒。
- **median**: 中位数处理时间为62毫秒。
- **max**: 最长处理时间是187毫秒。

这个阶段涵盖了从服务器接收到请求到开始发送响应之间的时间。它反映了服务器处理请求并生成响应的速度，包括任何内部操作如数据库查询、业务逻辑执行

**Waiting**：

- **min**: 最短等待时间是1毫秒。
- **mean[+/-sd]**: 平均等待时间为50毫秒，标准差为39.9毫秒。
- **median**: 中位数等待时间为61毫秒。
- **max**: 最长等待时间是186毫秒。

等待时间是指从客户端发出请求后到接收到第一个字节响应的时间。这个时间点标志着服务器已经开始返回数据，但可能还没有完成整个响应的传输

**Total**：

- **min**: 最短总时间是9毫秒。
- **mean[+/-sd]**: 平均总时间为56毫秒，标准差为38.4毫秒。
- **median**: 中位数总时间为67毫秒。
- **max**: 最长总时间是187毫秒。

总时间是从客户端发起请求到完全接收到所有响应数据的时间，综合了连接、处理和等待三个阶段的时间

## 测试案例

### 安装metrics组件进行hpa压测是进行观测资源状态

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
    rbac.authorization.k8s.io/aggregate-to-admin: "true"
    rbac.authorization.k8s.io/aggregate-to-edit: "true"
    rbac.authorization.k8s.io/aggregate-to-view: "true"
  name: system:aggregated-metrics-reader
rules:
- apiGroups:
  - metrics.k8s.io
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
rules:
- apiGroups:
  - ""
  resources:
  - nodes/metrics
  verbs:
  - get
- apiGroups:
  - ""
  resources:
  - pods
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server-auth-reader
  namespace: kube-system
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: extension-apiserver-authentication-reader
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server:system:auth-delegator
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:auth-delegator
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  labels:
    k8s-app: metrics-server
  name: system:metrics-server
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: system:metrics-server
subjects:
- kind: ServiceAccount
  name: metrics-server
  namespace: kube-system
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  ports:
  - name: https
    port: 443
    protocol: TCP
    targetPort: https
  selector:
    k8s-app: metrics-server
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    k8s-app: metrics-server
  name: metrics-server
  namespace: kube-system
spec:
  selector:
    matchLabels:
      k8s-app: metrics-server
  strategy:
    rollingUpdate:
      maxUnavailable: 0
  template:
    metadata:
      labels:
        k8s-app: metrics-server
    spec:
      containers:
      - args:
        - --cert-dir=/tmp
        - --secure-port=4443
        - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
        - --kubelet-use-node-status-port
        - --metric-resolution=15s
        - --kubelet-insecure-tls  #增加该段白名单
        image: registry.cn-hangzhou.aliyuncs.com/hong-space/metrics:v0.6.2  #注意镜像的拉取
        imagePullPolicy: Never
        livenessProbe:
          failureThreshold: 3
          httpGet:
            path: /livez
            port: https
            scheme: HTTPS
          periodSeconds: 10
        name: metrics-server
        ports:
        - containerPort: 4443
          name: https
          protocol: TCP
        readinessProbe:
          failureThreshold: 3
          httpGet:
            path: /readyz
            port: https
            scheme: HTTPS
          initialDelaySeconds: 20
          periodSeconds: 10
        resources:
          requests:
            cpu: 100m
            memory: 200Mi
        securityContext:
          allowPrivilegeEscalation: false
          readOnlyRootFilesystem: true
          runAsNonRoot: true
          runAsUser: 1000
        volumeMounts:
        - mountPath: /tmp
          name: tmp-dir
      nodeSelector:
        kubernetes.io/os: linux
      priorityClassName: system-cluster-critical
      serviceAccountName: metrics-server
      volumes:
      - emptyDir: {}
        name: tmp-dir
---
apiVersion: apiregistration.k8s.io/v1
kind: APIService
metadata:
  labels:
    k8s-app: metrics-server
  name: v1beta1.metrics.k8s.io
spec:
  group: metrics.k8s.io
  groupPriorityMinimum: 100
  insecureSkipTLSVerify: true
  service:
    name: metrics-server
    namespace: kube-system
  version: v1beta1
  versionPriority: 100
```

验证：

```yaml
#安装完成之后执行如下命令：
kubectl top node 
#如果正确返回则说明安装成功，如果显示没有该资源或者命令则是插件安装有问题。
```

### 后端服务pod

pod资源分配（为方便测试）：

cpu：200m

memory: 200Mi

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ab-ng
  labels:
    app: ab-ng
spec:
  replicas: 1
  selector:
    matchLabels:
      app: ab-ng
  template:
    metadata:
      labels:
        app: ab-ng
    spec:
      containers:
      - name: ab-ng
        image: registry.cn-hangzhou.aliyuncs.com/hong-space/nginx:1.24.0 #注意镜像拉取
        imagePullPolicy: Never
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 200m
            memory: 200Mi
          requests:
            cpu: 200m
            memory: 200Mi
---
apiVersion: v1
kind: Service
metadata:
  name: ab-svc
  labels:
    app: svc
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  selector:
    app: ab-ng
---
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: ab-hpa
  labels:
    app: hpa
spec:
  maxReplicas: 10
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: ab-ng
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 30
      policies:
      - type: Pods
        value: 1
        periodSeconds: 15
   #behavior:
   #scaleDown:
   #  stabilizationWindowSeconds: 300 # 缩容后观察稳定的时间窗口为300秒，等待300来观测缩容是稳定的
   #  policies:
   #  - type: Pods
   #    value: 4 # 每次缩容最多减少4个Pod
   #    periodSeconds: 60 # 每60秒检查一次是否满足缩容条件
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ab-ingress
spec:
  ingressClassName: ab-ingress
  rules:
  - host: www.ab.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: ab-svc
            port:
              number: 80
```

获取svc绑定host解析

`kubectl get svc`

![image-20250124165225274](https://github.com/user-attachments/assets/6704654a-ecf0-492a-8674-7c6d0cc6a5a4)

写入hosts解析

![image-20250124165313671](https://github.com/user-attachments/assets/aa9dcabb-2dd9-4452-876a-28e56c7d55e3)


不进行hosts解析会导致http请求到不了目标客户端

获取后端pod状态

`kubectl top pod`

![image-20250124164851751](https://github.com/user-attachments/assets/291dece7-0ba9-4247-a33c-daaa17916b38)


`watch kubectl top pod` #持续观测pod资源使用状态

![image-20250124170004293](https://github.com/user-attachments/assets/5a81b8b4-a899-4c0b-b64b-e3de413fee0e)


#### 进行压测（单个pod自动扩缩容）

ab -n 1000 -c 10 http://192.168.114.129/

![image-20250131165943833](https://github.com/user-attachments/assets/95113efe-d9d3-4cb3-bb9f-b095657fca8f)

pod资源状态

![image-20250131170016670](https://github.com/user-attachments/assets/23011c81-8765-4b54-8d21-4d782c3c427f)

ab -n 10000 -c 10 http://192.168.114.129/

![image-20250131170109631](https://github.com/user-attachments/assets/8669a6a7-8482-4fbf-a702-bd70ef9440fa)

pod资源状态

![image-20250131170231982](https://github.com/user-attachments/assets/e2d8df4d-66b5-4028-aff6-f48e3296729b)

![image-20250131170240649](https://github.com/user-attachments/assets/11d58bea-86cc-463d-aba8-ac39e43dc85f)

ab -n 100000 -c 10 http://192.168.114.129/

![image-20250131170735614](https://github.com/user-attachments/assets/da499e31-4b00-4e15-9d7d-e2e71b64077e)

pod资源状态

![image-20250131170637154](https://github.com/user-attachments/assets/14684384-feec-404f-9eeb-186a4c0b84ef)

![image-20250131170647217](https://github.com/user-attachments/assets/ef4cd81e-debc-48aa-9725-8585977e449c)

通过上诉3个例子可以很清楚的看到95%之后的请求时间大幅度增加，当前访问请求量来到10000以上时，传输速率基本稳定在2.17k

#### 多个副本(2)

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: ng-container
        image: registry.cn-hangzhou.aliyuncs.com/hong-space/nginx:1.24.0
        imagePullPolicy: Never
        ports:
        - containerPort: 80
        resources:
          limits:
            cpu: 200m
            memory: 200Mi
          requests:
            cpu: 200m
            memory: 200Mi
---
apiVersion: v1
kind: Service
metadata:
  name: rc-svc
  labels:
    app: svc
spec:
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
  selector:
    app: nginx
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: rc-ingress
spec:
  ingressClassName: rc-ingress
  rules:
  - host: www.rc.com
    http:
      paths:
      - path: /etc/nginx/html
        pathType: Prefix
        backend:
          service:
            name: rc-svc
            port:
              number: 80
```

ab -n 1000 -c 10 http://10.101.202.108/

![image-20250131174714672](https://github.com/user-attachments/assets/d3fe1de2-1f63-42e1-bf40-c7d8176359f2)

pod资源状态

![image-20250131174729441](https://github.com/user-attachments/assets/e9f0d47b-a456-41a1-ab27-3d582429cf7c)

![image-20250131174804202](https://github.com/user-attachments/assets/65e75f4a-7013-4613-b934-cc20bb60bf2f)

ab -n 10000 -c 10 http://10.101.202.108/

![image-20250131174830159](https://github.com/user-attachments/assets/feed0260-d7f6-4109-bd93-327dd6c0de71)

pod资源状态

![image-20250131174850616](https://github.com/user-attachments/assets/07bcf34a-7a1f-4f71-82b5-7fa7084e5f6b)

![image-20250131174909589](https://github.com/user-attachments/assets/3c214a0d-42f6-4184-b2d3-dd2de96838ff)

ab -n 100000 -c 10 http://10.101.202.108/

![image-20250131175540352](https://github.com/user-attachments/assets/6f717962-b6d0-4897-87ef-4d661d6a9641)

pod资源

![image-20250131175521621](https://github.com/user-attachments/assets/4273eb3d-33ec-4633-80e7-8c05013dab36)

![image-20250131175530487](https://github.com/user-attachments/assets/338743b7-8046-41e4-ac73-03054b113331)

通过这3次对比我可以很清楚的发现在资源量0.2c200m的情况下2个副本所能够承受的极限访问请求在99928，1000请求总量请求总时间为0.25s，10000请求总量为2.35s而到了100000请求总量时可以清楚的发现那怕资源依旧拥有赋予的情况下，能处理的请求依旧到极限来到44s，传输速率更是从正常的3.3k/s降低到1.8k/s

### hpa与固定副本数对比结论

单个pod在服务端被请求时访问压力明显比多个pod的访问压力要大的多，这一点从两者的传输速率上就可以明显的看差别。在拥有多个副本集后端服务可以进行轮询（svc默认策略是轮询）访问速率明显快于单个pod进行扩容后容乃请求，因此后端服务器资源还具有一定富裕时，可以通过给后端pod增加自扩容手段来面对超过预期的访问请求压力来缓解在此情况下出现的服务响应慢，不进行响应，甚至是后端服务pod直接被过量的请求压力压死的情况出现。

## ingress流量分流

#### 基于不同路径下的分流

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: host-in
  labels:
    app: in
spec:
  replicas: 1
  strategy:
    type: RollingUpdate
    rollingUpdate:
      maxSurge: 2
      maxUnavailable: 1
  selector:
    matchLabels:
      app: in
  template:
    metadata:
      labels:
        app: in
    spec:
      hostNetwork: true
      restartPolicy: Always
      nodeSelector:
        nginx: "schedule"
      tolerations:
      - key: "key"
        operator: "Equal" #Exists
        value: "nginx"
        effect: "NoExecute"
      #- key: "key"
      #  operator: "Exists"
      #  effect: "NoExexute" PreferNoSchedule,NoSchedule
      dnsPolicy: None
      dnsConfig:
        nameservers: ["114.114.114.114"]
        searches:
        - default.svc.cluster.local
        options:
        - name: ndots
          value: "2"
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: nginx
                operator: In
                values:
                - "schedule"
          preferredDuringSchedulingIgnoredDuringExecution:
            - weight: 1
              preference:
                matchExpressions:
                - key: nginx
                  operator: In
                  values:
                  - "schedule"
      hostAliases:
      - ip: "127.0.0.1"
        hostnames:
        - "boo.local"
        - "far.local"
      - ip: "10.92.208.226"
        hostnames:
        - "fwyq.local"
      containers:
      - name: host-ng
        image: registry.cn-hangzhou.aliyuncs.com/hong-space/nginx:1.24.0
        imagePullPolicy: Never
        ports:
        - containerPort: 80
        env:
        - name: "type_data"
          value: "int"
        envFrom:
        - configMapRef:
            name: cm-ref
        - secretRef:
            name: sec-ref
        lifecycle:
          postStart:
            exec:
              command: ["/bin/bash","-c","sleep 1s"]
          preStop:
            exec:
              command: ["/bin/bash","-c","sleep 5s"]
        resources:
          limits:
            cpu: 200m
            memory: 200Mi
          requests:
            cpu: 200m
            memory: 200Mi
        livenessProbe:
          httpGet:
            port: 80
            path: /
            host: 192.168.114.129
            scheme: HTTP
          initialDelaySeconds: 10
          periodSeconds: 10
        readinessProbe:
          exec:
            command: ["touch","/healthz"]
          initialDelaySeconds: 10
          periodSeconds: 10
        startupProbe:
          tcpSocket:
            port: 80
          initialDelaySeconds: 10
          periodSeconds: 10
          failureThreshold: 10
          successThreshold: 1
        volumeMounts:
        - name: logs
          mountPath: /var/log/nginx
          subPath: log
        - name: html
          mountPath: /etc/nginx/html
          subPath: html
        - name: conf
          mountPath: /etc/nginx/nginx.conf
          subPath: nginx.conf
      volumes:
      - name: logs
        hostPath:
          path: /my_data
          type: Directory
      - name: html
        persistentVolumeClaim:
          claimName: html-pvc
      - name: conf
        configMap:
          name: conf
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-ref
  labels:
    app: cm-in
data:
  default_data: "0"
  max_Length: "3"
---
apiVersion: v1
kind: Secret
metadata:
  name: sec-ref
  labels:
    app: sec-in
type: Opaque
data:
  default_user: ""
  default_pass: ""
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: conf
  labels:
    app: conf-in
data:
  nginx.conf: |-
    user  nginx;
    worker_processes  auto;

    error_log  /var/log/nginx/error.log notice;
    pid        /var/run/nginx.pid;

    events {
        worker_connections  1024;
    }

    http {
        include       /etc/nginx/mime.types;
        default_type  application/octet-stream;

        log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                          '$status $body_bytes_sent "$http_referer" '
                          '"$http_user_agent" "$http_x_forwarded_for"';

        access_log  /var/log/nginx/access.log  main;

        sendfile        on;
        tcp_nopush     on;
        tcp_nodelay    on;
        keepalive_timeout  65;
        types_hash_max_size 2048;

        include       /etc/nginx/conf.d/*.conf;

        server {
            listen       80;
            server_name  localhost;

            location / {
                root   /usr/share/nginx/html;
                index  index.html index.htm;

                # 开启目录列表功能
                autoindex on;
                autoindex_exact_size off; # 显示更友好的文件大小，默认是on，显示精确大小
                autoindex_localtime on;   # 显示本地时间而非GMT时间，默认是off
            }
        }
    }
---
apiVersion: v1
kind: Service
metadata:
  name: host-svc
  labels:
    app: svc
spec:
  selector:
    app: in
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    nodePort: 30039
  type: NodePort
---
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: host-hpa
  labels:
    app: hpa
spec:
  maxReplicas: 10
  minReplicas: 1
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: host-in
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 60
  behavior:
    scaleDown:
      stabilizationWindowSeconds: 60
      policies:
      - type: Pods
        value: 1
        periodSeconds: 30
--- #此处统一管理的ingress写在第一个deploy中，可以分开写看个人喜好
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-ingress
spec:
  ingressClassName: host-ingress
  rules:
  - host: www.host.com
    http:
      paths:
      - path: / #访问www.host.com/下跳转到host-svc代理的后端pod
        pathType: Prefix
        backend:
          service:
            name: host-svc
            port:
              number: 80
      paths:
      - path: /api #这里是deploy的api响应路径，访问www.host.com/api/resource/下跳转到svc2代理的后端pod
        pathType: Prefix
        backend:
          service:
            name: svc2
            port:
              number: 80
#deployment2
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deploy
  labels:
    app: node2
spec:
  replicas: 1
  selector:
    matchLabels:
      app: node2
  template:
    metadata:
      labels:
        app: node2
    spec:
      containers:
      - name: test-node2
        image: t-py:v2
        imagePullPolicy: Never
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: svc2
  labels:
    app: svc
spec:
  type: ClusterIP
  ports:
  - port: 80
    protocol: TCP
    targetPort: 80
  selector:
    app: node2
```

##### deploy2的api响应

```py
from flask import Flask, jsonify

app = Flask(__name__)

@app.route('/api/resource', methods=['GET']) #api响应路径及方式
def get_resource():
    return jsonify({"message": "This is a response from the API service."}), 200

if __name__ == '__main__':
    app.run(host='0.0.0.0', port=80)
```

##### 生成响应image

```Dockerfile
FROM registry.cn-hangzhou.aliyuncs.com/hong-space/python-3.9:3.9-slim

WORKDIR /app #此出为设置工作目录非api响应路径

COPY . /app

RUN pip install -i https://pypi.tuna.tsinghua.edu.cn/simple --no-cache-dir flask

CMD ["python", "app.py"]
```

##### 获取两个deployment的对应的svc写入host解析

![image-20250204215644335](https://github.com/user-attachments/assets/199f10b0-c628-4fa0-abba-076e621e1563)

##### 访问验证

跳转host-svc

`curl http://www.host.com/`

![image-20250204215729059](https://github.com/user-attachments/assets/f70eff02-c551-4332-9749-f7141c1fb5eb)

![image-20250204215809477](https://github.com/user-attachments/assets/133fd107-1a5e-4097-b493-acb18a326364)

页面演示

![image-20250204220756426](https://github.com/user-attachments/assets/7d2d05fb-2bdc-46c1-8a73-133b50761240)

跳转svc2

`curl http://www.host.com:80/api/resource/`

![image-20250204220214521](https://github.com/user-attachments/assets/60d15075-5ed8-4872-a3e8-7abbafcf8ec1)

![image-20250204220259110](https://github.com/user-attachments/assets/4d692ec3-f73f-4a46-b8b9-1aee394d8b65)

页面演示

![image-20250204221223404](https://github.com/user-attachments/assets/ada9abc7-bd2a-465f-84a6-4138a74ccd15)

#### 基于不同域名的分流

```yaml
#更改域名部分
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: host-ingress
spec:
  ingressClassName: host-ingress
  rules:
  - host: www.host.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: host-svc
            port:
              number: 80
  - host: www.svc.com
    http:
      paths:
      - path: /api
        pathType: Prefix
        backend:
          service:
            name: svc2
            port:
              number: 80
```

##### 获取两个deploy对应的svc写入解析

![image-20250204221735935](https://github.com/user-attachments/assets/ee71de77-0b4c-4c2b-83e0-094f8d73c760)

##### 访问验证

跳转host-svc

![image-20250204221931091](https://github.com/user-attachments/assets/81721bbb-2265-4c96-9561-70aa5385c9a1)

![image-20250204221956860](https://github.com/user-attachments/assets/9bad4fd4-403b-4718-b849-9767d66150f6)

页面演示

![image-20250204222041566](https://github.com/user-attachments/assets/c1532f74-f3a8-4dab-92cd-10b833f15a12)

跳转svc2

![image-20250204222216225](https://github.com/user-attachments/assets/a5d309be-7879-46e7-84da-8ec31b7bb903)

![image-20250204222149014](https://github.com/user-attachments/assets/98177583-f2e4-47e4-b6b5-8867138821c4)

页面演示

![image-20250204222303266](https://github.com/user-attachments/assets/cb24bde2-ed10-4c78-83a7-e1a6bd02c455)
