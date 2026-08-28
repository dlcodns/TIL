# LIVE 테스트 계정 및 리소스 정보
2026.08.27

> skyline에 로그인 할 때는 **{ID}@highcloud** 로 로그인 하세요.
> (현재는 다운된 서버)

**VM 이름: alpha1**
**OS:** Ubuntu-24.04-LTS-Cloud
**Flavor:** m1.small
**vCPU/RAM:** 1VCPU/2GiB
**IP:** 10.0.0.165
**keypair:** alpha-key

**VM 이름: alpha2**
**OS:** Ubuntu-22.04-LTS-Cloud
**Flavor:** m1.medium
**vCPU/RAM:** 2VCPU/4GiB
**IP:** 10.0.0.158
**keypair:** alpha-key

**VM 이름: beta1**
**OS:** Rocky-Linux-9-Cloud
**Flavor:** m1.nano
**vCPU/RAM:** 1VCPU/1GiB
**IP:** 10.0.0.124
**keypair:** beta-key

**VM 이름: beta2**
**OS:** Ubuntu-24.04-LTS-Cloud
**Flavor:** m1.small
**vCPU/RAM:** 1VCPU/4GiB
**IP:** 10.0.0.136
**keypair:** beta-key

**VM 이름: gamma1**
**OS:** Ubuntu-24.04-LTS-Cloud
**Flavor:** m1.small.bfv
**vCPU/RAM:** 1VCPU/2GiB
**Cinder Volume:** 20GiB
**IP:** 10.0.0.
**keypair:** gamma-key

**VM 이름: gamma2**
**OS:** Ubuntu-22.04-LTS-Cloud
**Flavor:** m1.medium.bfv
**vCPU/RAM:** 4VCPU/4GiB
**Cinder Volume:** 20GiB
**IP:** 10.0.0.
**keypair:** gamma-key

---

## 리소스 구성 이유

1. m1.large(8GB)는 프로젝트 램 쿼터 8GB를 다 먹어서 플젝 2개 생성 못함. 그래서 뺌.
2. alpha와 gamma의 차이점은 자체 disk볼륨을 쓰느냐, 아니면 cinder를 쓰느냐 차이. 다양성을 위해서.
3. beta에는 현재 기본적으로 있는 rocky os와 m1.nano를 유일하게 사용함.

| 프로젝트 | VM | 플레이버 | vCPU/RAM/디스크 | 이미지 | 디스크 방식 |
| --- | --- | --- | --- | --- | --- |
| alpha-20260617 | alpha1 | m1.small | 1 / 2GB / 20GB | Ubuntu 24.04 | ephemeral |
| alpha-20260617 | alpha2 | m1.medium | 2 / 4GB / 40GB | Ubuntu 22.04 | ephemeral |
| beta-20260617 | beta1 | m1.nano | 1 / 1GB / 10GB | Rocky 9 | ephemeral |
| beta-20260617 | beta2 | m1.medium | 2 / 4GB / 40GB | Ubuntu 24.04 | ephemeral |
| gamma-20260617 | gamma1 | m1.small.bfv | 1 / 2GB / 볼륨 | Ubuntu 24.04 | boot-from-volume |
| gamma-20260617 | gamma2 | m1.medium.bfv | 2 / 4GB / 볼륨 | Ubuntu 22.04 | boot-from-volume |

---

## 신청 내역
![alt text](/assets/Openstack/highcloud1.png)

## 신청 시트
![alt text](/assets/Openstack/highcloud2.png)

## CLI 확인 명령어
![alt text](/assets/Openstack/highcloud3.png)