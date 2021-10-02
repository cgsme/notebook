# Deployment

无状态应用部署，比如微服务，提供多副本功能。

## scale 扩缩容

## 自动修复、故障转移

## 滚动更新

  #set image：修改指定的deploy中的镜像。--record：记录这次更新
  kubectl set image deployment/xxx-dep nignix=nginx:1.16.1 --record

  #查看更新记录
  kubectl rollout history deployment/xxx-dep

## 版本回退

  #可以先查看更新记录
  kubectl rollout history deployment/xxx-dep

  #回退到指定的版本(也是一个滚动更新的过程)
  kubectl rollout undo deploy/my-dep --to-revision=1
