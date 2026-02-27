# Tomcat运行不起来，排查思路是怎样的？（基础中间件运行不起来先查环境！环境变量、文件权限、端口占用、僵尸进程等）
- 先观察启动时的信息输出，若是权限问题则检查bin目录下的启动脚本是否有执行权限。然后检查基础环境，排查端口占用问题和Java环境问题。基础环境没问题后排查日志，标准输出日志文件catalina.out,服务崩溃则排查catalina.xxxx-xx-xx.log。


# 假设容器内的服务启动不了或突然崩溃，你是如何排查问题的？
## 排查思路是：状态 → 日志 → 内部环境 → 资源 → 外部依赖。先快速定位问题类型，再针对性深入。
- docker ps -a 查看所有容器（包括已退出），关注 STATUS 列和退出码（如 137、139、143）可提示是被 OOM Kill 还是进程主动退出。
- docker logs 容器日志中通常包含服务启动失败的具体错误（如配置文件错误、端口冲突、依赖服务未就绪）。
- docker exec -it <容器名> ps aux    # 查看容器内运行的进程
- docker top <容器名>                # 更简洁的进程查看
- docker exec -it <容器名> /bin/bash 看应用日志、配置文件
- docker inspect <容器名> | grep -i memory   # 查看内存限制
- docker stats <容器名>                      # 查看实时CPU、内存、网络IO   检查容器资源限制以及使用情况
- 检查宿主机资源
- 容器挂载卷、权限、网络配置、端口映射

- Kubernetes 环境：
- kubectl get pods -n <命名空间>                 # 查看Pod状态
- kubectl describe pod <pod名> -n <命名空间>     # 查看事件、错误原因
- kubectl logs <pod名> -n <命名空间>              # 查看容器日志
- kubectl exec -it <pod名> -n <命名空间> -- /bin/bash  # 进入容器

# 假设docker容器突然崩溃或宕机了，如何定位问题原因？
## 排查思路是：先看状态（docker ps -a 退出码）、再看日志、然后看宿主机资源、最后看配置（资源限制、挂载、网络）