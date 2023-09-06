# nginx-rtmp服务器+rtmp鉴权服务器部署脚本
## 运行环境
任何一个k8s集群
## 使用方法
### 修改部署节点
修改`rtmp_server_deployment.yaml`中的nodeSelector字段，指定部署节点。
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: rtmp-server-deployment
spec:
  selector:
    matchLabels:
      app: rtmp-server
  template:
    metadata:
      labels:
        app: rtmp-server
    spec:
      nodeSelector:
        kubernetes.io/hostname: debian-digitalocean # 修改为你自己的节点名
```
### 修改鉴权服务器的用户名和密码
修改`rtmp_server_deployment.yaml`中的环境变量`USERNAME`和`PASSWORD`，分别为用户名和密码。
```yaml
env:
  - name: USERNAME
  value: huajuan # 修改为你自己的用户名
  - name: PASSWORD
  value: 123456 # 修改为你自己的密码
```
### 部署
```bash
kubectl apply -f rtmp_server_deployment.yaml
```
### 推流

由于使用的是NodePort方式暴露服务，所以rtmp服务器的端口号默认为`31935`，而不是`1935`可以在`rtmp_server_deployment.yaml`中修改。
```yaml
apiVersion: v1
kind: Service
metadata:
  name: rtmp-server-service
spec:
  type: NodePort
  selector:
    app: rtmp-server
  ports:
  - port: 1935 
    targetPort: 1935
    nodePort: 31935 # 修改为你自己喜欢的端口号，只能是30000～32767之间的数字
    name: rtmp
  - port: 80
    targetPort: 80
    nodePort: 30080
    name: http
```
### 推流
推流时需要在推流地址中加入用户名和密码，例如：`rtmp://host-of-rtmp-server:31935/live/stream?username=huajuan&password=123456`。其中用户名和密码就是在`rtmp_server_deployment.yaml`中设置的用户名和密码。stream为流名，可以自己定义。

### 拉流
和推流一样，拉流也需要在拉流地址中加入用户名和密码，例如：`rtmp://host-of-rtmp-server:31935/live/stream?username=huajuan&password=123456`。其中用户名和密码就是在`rtmp_server_deployment.yaml`中设置的用户名和密码。stream为流名，可以自己定义。
