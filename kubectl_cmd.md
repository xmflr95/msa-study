# Kubernetes, Kubectl 사용 명령어 
개인적으로 경험? 사용했던 kubectl 명령어 단순 정리

## kubectl
```bash
# kubectl
# pod 현재 상태보기
kubectl get pods 
kubectl get pod

# 라벨 보기
kubectl get pods --show-labels
# 좀더 상세 보기
kubectl get svc -o wide
# json view
kubectl get pods -o json
# yaml로 보기
kubectl get pods -o yaml

# json path 활용
kubectl get pods -o=jsonpath='{@}'
kubectl get pods -o=jsonpath="{.items[0].metadata.name}"
kubectl get pods -o=jsonpath="{.items[*]['metadata.name', 'metadata.namespace']}"

kubectl get pods -o=jsonpath="{range .items[*]}{.metadata.name}{'\t'}{.metadata.namespace}{'\t'}{.status.podIP}{'\n'}{end}"

kubectl get pods -o=jsonpath="{'name'}{'\t\t\t\t'}{'podIP'}{'\t\t'}{'namespace'}{'\n'}{range .items[*]}{.metadata.name}{'\t'}{.metadata.namespace}{'\t'}{.status.podIP}{'\n'}{end}"

# Pod, ReplicaSet, Deployment, Service 약어로 한번에 보기
kubectl get po,rs,deploy,svc
# 굉장히 상세하게 보는법
kubectl get po,rs,deploy,svc -o wide
# 더 빠르게 많이 보기
kubectl get all
kubectl get all -o wide

# 적용?
kubectl apply -f ${setting}.yaml
# 삭제?
kubectl delete -f ${setting}.yaml 

# 네임스페이스(namespace, ns) 확인
kubectl get namespaces
# ns로 정보 확인
kubectl get all --all-namespaces
# ns가 kube-system인 정보 모두 확인
kubectl get all -n kube-system

# PVC(PersistentVolumeClaim) 확인, 추가 공부 필요
kubectl get pvc

# endpoint 보기
kubectl get endpoints
kubectl get ep # 약어

# endpoint 상세
# kubectl describe ep/${service name}
kubectl describe ep/ibks-elk-e-svc

# 컨테이너 접속
kubectl exec -it ${pod_name} /bin/bash

# port-foword 명령어
nohup kubectl port-forward --address=0.0.0.0 service/service_name ${external_port}:${cluster_port}
# example
nohup kubectl port-forward --address=0.0.0.0 service/ibks-tda-eureka 18008:18000

# nohup으로 백그라운드 진행
# 1>/dev/null 2>&1 (표준출력 미사용) & (백그라운드 실행)
nohup kubectl port-forward --address=0.0.0.0 service/ibks-elk-e-svc -n ibks-elk 18000:9200 & nohup kubectl port-forward --address=0.0.0.0 service/ibks-elk-k-svc -n ibks-elk 18001:5601 1>/dev/null 2>&1 &

# 1개의 파드에 다중 컨테이너일때 컨테이너 접속법
kubectl exec -it ibks-tda-iam-pod-d95cbc95-jzknd -c filebeat-iam -- /bin/bash

# 대시보드 접속 토큰 확인
kubectl -n kube-system describe secret $(kubectl -n kube-system get secret | grep admin-user | awk '{print $1}')

```