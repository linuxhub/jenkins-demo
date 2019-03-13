# k8s部署jenkins与打包部署java应用

## 阿里云Docker镜像仓库认证信息配置
### 推送镜像-使用
名称: `jenkins-docker-cfg`
```bash
docker login --username=<帐号> --password=<密码> <镜像仓库地址>
kubectl create secret generic jenkins-docker-cfg --from-file=/root/.docker/config.json
```
 
### 拉取镜像-使用
名称: `docker-registry`
```bash
kubectl create secret docker-registry docker-reg-auth --docker-server=<镜像仓库地址> --docker-username=<帐号> --docker-password=<密码>
```
