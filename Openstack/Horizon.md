# Horizon

Horizon은 OpenStack의 웹 대시보드로, CLI 없이 브라우저에서 OpenStack 리소스를 관리할 수 있다.

## Horizon에서 관리할 수 있는 주요 기능

Horizon을 통해 OpenStack의 주요 서비스를 직접 관리할 수 있다.

| 기능 | 설명 |
|---|---|
| 사용자 및 프로젝트 관리 | 사용자 계정 생성 및 프로젝트(Role) 할당 |
| 인스턴스(Compute) 관리 | Nova 인스턴스(VM) 생성, 삭제, 스냅샷 생성 |
| 스토리지 관리 | Cinder 볼륨 생성, 삭제, 스냅샷 관리 |
| 네트워크 관리 | Neutron 네트워크 및 서브넷, Floating IP 설정 |
| 이미지 관리 | Glance 이미지 업로드 및 삭제 |
| 보안 그룹 및 키페어 관리 | 방화벽 규칙(Security Group) 및 SSH Key 등록 |

## 로그인

Horizon에 접속하면 로그인 화면이 나온다. admin 계정으로 로그인한다.

**멀티 도메인**

만약 도메인이 Default 말고도 있다면 로그인 창에 도메인 선택란도 나온다.

![alt text](/assets/Openstack/horizon1.png)

## 프로젝트 확인

로그인하면 오른쪽 상단에 현재 프로젝트 이름이 표시된다. Kolla-Ansible이 기본으로 만들어주는 프로젝트는 두 가지다.

```
admin    ← admin 유저용 프로젝트
service  ← nova, glance 등 서비스 계정용 프로젝트
```

admin으로 로그인하면 admin 프로젝트 안에 있는 상태다. 여기서 생성하는 모든 리소스는 admin 프로젝트 소속이 된다.

![alt text](/assets/Openstack/horizon2.png)

## 프로젝트 전환

오른쪽 상단 프로젝트 이름을 클릭하면 다른 프로젝트로 전환할 수 있다. 프로젝트가 바뀌면 보이는 리소스도 해당 프로젝트 소속으로 바뀐다.

## 프로젝트 생성

**인증(Identity) → 프로젝트(Projects) → 프로젝트 생성(Create Project)** 에서 새 프로젝트를 만들 수 있다.

![alt text](/assets/Openstack/horizon3.png)

멤버, 그룹, 역할을 지정할 수 있다. RBAC 기능 중 하나.

![alt text](/assets/Openstack/horizon4.png)

## 역할

**인증 → 역할** 메뉴에 여러 역할이 있는 것을 볼 수 있고, 생성할 수 있다. 역할별로 통제하려면 policy.yaml을 직접 수정해야 한다.

![alt text](/assets/Openstack/horizon5.png)

## 유저 생성

**인증(Identity) → 사용자(Users) → 사용자 생성(Create User)** 에서 유저를 만들 수 있다. Horizon에서 유저를 만들 때는 Primary Project와 Role을 함께 지정해야 한다. CLI와 다르게 역할 없이는 생성이 안 된다.

![alt text](/assets/Openstack/horizon6.png)
![alt text](/assets/Openstack/horizon7.png)

## VM 생성

**Compute → 인스턴스(Instances) → 인스턴스 생성(Launch Instance)** 에서 VM을 생성할 수 있다. VM은 반드시 프로젝트 안에서 생성되며, 프로젝트가 없으면 생성 자체가 불가하다.

이때 이미지가 필요하다.

## 이미지 생성

Compute 메뉴에 이미지를 누른다. 이미지는 OS를 깔 때 필요한 boot usb라고 보면 된다. 아래 사이트에서 원하는 이미지를 고르고 VM을 만들 때 사용해야 한다.

https://docs.openstack.org/image-guide/obtain-images.html

(내가 본 블로그에서는 CirrOS 라는 minimal Linux distribution으로 다운 받으셨다.)

그 후 이미지 생성 버튼을 누르고, 다운 받은 파일을 이미지 소스에 넣는다.

![alt text](/assets/Openstack/horizon8.png)

이미지 공유 가시성은 이미지를 모든 유저가 사용할 수 있게 할 것인지, 특정 유저 혹은 프로젝트만 사용할 수 있게할 지 지정하는 선택란이다.

## Flavor

AWS에서는 과금을 방지하기 위한 하드웨어 분류를 Flavor라고 한다. (t3.medium) OpenStack의 이 메뉴에서는 하드웨어 분류를 할 수 있다.

![alt text](/assets/Openstack/horizon9.png)

이미지의 용량에 따라서 잘 설정해야 한다. 아래 사진의 경우는 초경량 cirros이기 때문에 vCPU 1개, RAM 1GB, Root 디스크 10GB로 설정한 것을 볼 수 있다.

### vCPU

VM이 사용하는 논리적 CPU.

```
물리 서버
 └── CPU: 8코어 (+ 하이퍼스레딩 → 16스레드)
      └── OpenStack이 쪼개서 vCPU로 할당
           ├── VM-1: vCPU 2개
           ├── VM-2: vCPU 4개
           └── VM-3: vCPU 2개
```

### 오버커밋

물리 코어 8개 × 오버커밋 비율 16 = vCPU 128개까지 할당 가능하게 하는 경우가 많다.

VM들이 동시에 CPU를 100% 쓰는 경우가 드물기 때문에 가능한 방식이고, AWS도 이렇게 운영한다고 한다.

비슷한 케이스로 VMware 운영 상에서 VM을 생성할 때 'vCPU : 총 할당 vCPU'의 비율을 1:3 정도로 배정하는 것이 좋다고 한다. 1대1 할당이 원칙이라고 할 때는 절대로 고려할 수 없는 비율이지만, 1:3 수치가 가능한 것은 할당된 VM이 항상 모든 CPU를 100% 사용하고 있지 않기 때문이다.

![alt text](/assets/Openstack/horizon10.png)

## 네트워크

### 네트워크 토폴로지

현재 구성된 네트워크 전체를 시각적 다이어그램으로 보여주는 화면이다.

![alt text](/assets/Openstack/horizon11.png)

### 네트워크

가상 네트워크와 서브넷을 생성하고 관리하는 메뉴이다. 네트워크는 크게 두 종류로 나뉜다.

- **외부 네트워크(External Network)**: 인터넷과 연결되는 네트워크로, admin만 생성할 수 있다. Floating IP 할당의 기반이 되며, 외부에서 VM에 접근할 때 사용한다.
- **내부 네트워크(Internal Network)**: 프로젝트 내부 VM끼리 통신하는 네트워크로, 일반 사용자도 생성할 수 있다. 생성 시 서브넷(IP 대역)을 함께 설정한다.

### 라우터

내부 네트워크와 외부 네트워크를 연결하는 가상 라우터를 생성하고 관리하는 메뉴이다. 라우터가 없으면 VM이 외부와 통신할 수 없다. 라우터에 외부 네트워크를 게이트웨이로 설정하고, 내부 네트워크를 인터페이스로 추가하면 VM이 외부와 통신할 수 있는 경로가 만들어진다.

### 보안 그룹

VM 단위로 적용되는 가상 방화벽. 인바운드/아웃바운드 트래픽을 보안 규칙으로 제어한다.

**기본 규칙**

기본 규칙은 제한적이기 때문에 보안 규칙을 우리가 생성해야 한다!!

- Inbound: 들어오는 트래픽 → 전부 차단
- Outbound: 나가는 트래픽 → 전부 허용

**규칙 구성 요소**

| 항목 | 설명 |
|---|---|
| Direction | Ingress(인바운드) / Egress(아웃바운드) 선택 |
| Protocol | TCP / UDP / ICMP |
| Port | 허용할 포트 번호 |
| CIDR | 허용할 IP 대역 |

**외워두면 좋은 규칙**

```
SSH 접속       → TCP, 22번,  0.0.0.0/0
HTTP 서비스    → TCP, 80번,  0.0.0.0/0
HTTPS 서비스   → TCP, 443번, 0.0.0.0/0
핑(ping) 허용   → ICMP, -1,   0.0.0.0/0
```