# workspace-manifests

**Workspace Booking** Kubernetes 매니페스트 저장소

```
Docker Compose로 운영하던 서비스를 Kubernetes로 이관하며 만들었다. 
목표는 고가용성이 아니라 "compose 구성을 K8s 오브젝트로 최대한 그대로 옮기는 것"이라,
Helm·Kustomize·멀티 replica는 쓰지 않았다.
```

## 구조

```
apps/      Argo CD가 감시하는 실제 K8s 리소스 (Deployment, StatefulSet, Service)
argocd/    Argo CD Application 정의
```

Ingress와 Cloudflare Tunnel 토큰 Secret은 Rancher에서 직접 관리한다(이 저장소가 다루는 GitOps 범위 밖).

## 배포 흐름

1. `workspace-backend`/`workspace-frontend`의 CI가 이미지를 빌드해 GHCR에 올린다.
2. CI가 이 저장소의 `apps/*-deployment.yaml` 이미지 태그를 커밋한다.
3. Argo CD가 이 저장소를 감시하다 변경을 감지해 자동 sync(`prune`, `selfHeal`)한다(pull 방식 배포).

## 설정 근거

- **MySQL 볼륨 복제본 2** — 공유 StorageClass의 기본값(3)이 워커 노드 수(2)보다 많아, 해당 볼륨만 개별적으로 복제본 2로 지정했다.
- **backend replicas 1** — `@Scheduled` 폴링·재시도가 인스턴스 간 분산 락 없이 동작해, 중복 발송을 막기 위해 단일 인스턴스로 운영한다.
- **cloudflared Tunnel → ingress-nginx-controller** — `frontend` Service가 아니라 `ingress-nginx-controller`를 목적지로 잡아, 기존 K8s Ingress 라우팅을 그대로 활용한다.
