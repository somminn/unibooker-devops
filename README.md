# UniBooker DevOps (Kubernetes / GitOps / CI/CD)

UniBooker 서비스를 운영/배포 관점에서 정리한 DevOps 레포입니다. 

애플리케이션 소스코드는 포함하지 않고, 클러스터 구성(Ansible+kubeadm), 배포 매니페스트(Kustomize/Argo Rollouts), CI (Jenkins / GitHub Actions), GitOps-based CD (ArgoCD), 모니터링(Prometheus/Grafana) 설정을 중심으로 구성했습니다.

<br>

## What’s inside

* **Ansible + kubeadm**: 멀티 노드 쿠버네티스 클러스터 프로비저닝 자동화
* **CI (Jenkins / GitHub Actions)**: 컨테이너 이미지 빌드 및 GitOps 레포 태그 업데이트
* **GitOps CD (ArgoCD)**: Pull 기반 배포 자동화
* **Argo Rollouts**: 백엔드 Blue/Green 배포 전략 적용
* **Networking**: ingress-nginx 및 Envoy Gateway(Gateway API) 구성
* **Monitoring**: kube-prometheus-stack 기반 클러스터/애플리케이션 모니터링
* **Storage**: Kaniko cache PVC 등 빌드 최적화 구성
* **Kustomize**: Kubernetes 매니페스트 구조화 및 환경 관리

<br>

## Repository structure
```text
unibooker-devops/

├── ansible/
│
│   # kubeadm 기반 클러스터 구성 자동화
│
│   ├── playbooks/
│   │   ├── 01-prereq.yaml
│   │   └── 02-kubeadm-cluster.yaml
│   │
│   ├── inventory/
│   └── site.yaml
│
│
├── ci/
│
│   # CI 파이프라인 정의
│
│   ├── github-actions/
│   │   ├── backend-ci-cd.yaml
│   │   └── frontend-ci-cd.yaml
│   │
│   └── jenkins/
│       ├── jenkinsfile-backend.yaml
│       └── jenkinsfile-frontend.yaml
│
│
├── cd/
│
│   └── argocd/
│
│       # GitOps CD (ArgoCD)
│
│       ├── apps/
│       │   ├── backend.yaml
│       │   ├── frontend.yaml
│       │   └── infra.yaml
│       │
│       └── application.yaml
│
│
├── k8s/
│
│   # Kubernetes 매니페스트
│
│   ├── apps/
│   │
│   │   ├── backend/
│   │   │   ├── kustomization.yaml
│   │   │   ├── rollout-bluegreen.yaml
│   │   │   ├── service-active.yaml
│   │   │   └── service-preview.yaml
│   │   │
│   │   └── frontend/
│   │       ├── deployment.yaml
│   │       ├── service.yaml
│   │       └── kustomization.yaml
│   │
│   └── platform/
│       ├── monitoring/
│       │   └── values.yaml
│       │
│       ├── networking/
│       │   ├── ingress-nginx/
│       │   │   └── ingress-nginx.yaml
│       │   │
│       │   └── envoy-gateway/
│       │       ├── gateway.yaml
│       │       ├── httproute.yaml
│       │       └── kustomization.yaml
│       │
│       └── storage/
│           └── kaniko-cache-pvc.yaml
```

<br>


## Kubernetes cluster (kubeadm) provisioning
* `01-prereq.yaml`: OS 사전 설정(네트워크 sysctl, 모듈, containerd/kubeadm 설치 등)
* `02-kubeadm-cluster.yaml`: master init + worker join

<br>

## Networking

두 가지 방식을 같이 정리했습니다.

### ingress-nginx

* `k8s/platform/networking/ingress-nginx/ingress-nginx.yaml`

### Envoy Gateway (Gateway API)

* `k8s/platform/networking/envoy-gateway/gateway.yaml`
* `k8s/platform/networking/envoy-gateway/httproute.yaml`

> 개인 확장 과정에서 Gateway API 기반 라우팅으로 전환했습니다.

<br>

## Deployments

### Backend (Blue/Green with Argo Rollouts)

* `k8s/apps/backend/rollout-bluegreen.yaml`
* `k8s/apps/backend/service-active.yaml`
* `k8s/apps/backend/service-preview.yaml`
* `k8s/apps/backend/kustomization.yaml`

### Frontend (Deployment)

* `k8s/apps/frontend/deployment.yaml`
* `k8s/apps/frontend/service.yaml`
* `k8s/apps/frontend/kustomization.yaml`

<br>

## CI

두 가지 방식을 같이 정리했습니다.

### Jenkins

* `ci/jenkins/jenkinsfile-backend.yaml`
* `ci/jenkins/jenkinsfile-frontend.yaml`

### GitHub Actions

* `ci/github-actions/backend-ci-cd.yaml`
* `ci/github-actions/frontend-ci-cd.yaml`

> 개인 확장 과정에서 동일 파이프라인을 GitHub Actions로 재구성하여 Self-Hosted CI와 SaaS CI 환경을 비교했습니다.

<br>

## CD

### ArgoCD

`cd/argocd/` 아래에 ArgoCD Application 정의가 있습니다.

* `cd/argocd/apps/backend.yaml`
* `cd/argocd/apps/frontend.yaml`
* `cd/argocd/apps/infra.yaml`
* `cd/argocd/application.yaml`

<br>

## Storage

* `k8s/platform/storage/kaniko-cache-pvc.yaml`
> Kaniko 빌드 속도 개선을 위해 cache PVC를 사용했습니다.

<br>

## Monitoring (Prometheus + Grafana)

`k8s/platform/monitoring/values.yaml`은 kube-prometheus-stack 설치 커스터마이징 값입니다.

```bash
kubectl create namespace monitoring

helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update

helm upgrade --install kube-prom-stack prometheus-community/kube-prometheus-stack \
  -n monitoring \
  -f k8s/platform/monitoring/values.yaml
```

<br>

## CI/CD Flow
![final](https://github.com/user-attachments/assets/05eff2f7-85f9-4d58-8818-ac9d79d06bc9)



## Notes

* 이 레포는 “애플리케이션 기능 개발”이 아니라 **운영/배포 자동화 관점의 구성**을 보여주기 위한 목적입니다.
* 환경/도메인/IP/레지스트리 명은 공개를 위해 일부 일반화될 수 있습니다.
