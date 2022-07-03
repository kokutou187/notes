- kubectl cluster-info: 查看集群信息
- kubectl get nodes/services/deployment: 查看所有节点/服务
- kubectl describe pods: 查看pod的详细信息，包括IP，容器等
    - -l 使用label来过滤你要查找的对象
- kubectl logs $POD_NAME: 查看pod的日至
- kubectl exec \$POD_NAME -- COMMAND: 在pod中执行命令
    - -ti \$POD_NAME --bash: 使用容器的bash。-t 将标准输入控制台作为容器的控制台，-i 将输入送入容器
- kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080: 将这个deployment作为服务在8080端口暴露出去。这个8080端口是针对这个pod来说，要知道在node上的port需要使用如下命令:
    - export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')

- kubectl label pods $POD_NAME xxx=xxx: 给pod添加label
- kubectl delete serveice -l xx=xx
- kubectl scale deployments/xxx --replicas=4: 缩放副本数到4
- kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2: 更新应用
- kubectl rollout undo deployments/kubernetes-bootcamp: 撤销新的部署动作

```shell
curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME
```