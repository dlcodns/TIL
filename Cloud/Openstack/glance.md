# Glance
2026.08.27

Glance는 OpenStack의 이미지 서비스다. VM을 생성할 때 사용할 OS 이미지를 저장하고 관리하는 역할을 한다.

![alt text](/assets/Openstack/glance1.png)

## 주요 기능

VM 생성에 필요한 OS 이미지를 저장하고 관리하며, RAW/QCOW2/VMDK 등 다양한 포맷을 지원한다. Nova 요청 시 적절한 포맷으로 변환해서 제공하고, 이미지 이름/크기/체크섬 같은 메타데이터도 저장·검색할 수 있다. 동일 프로젝트 내 이미지 공유 및 다른 클라우드 환경으로 복제도 가능하다.

## 이미지 포맷

### 1. 디스크 포맷 (VM의 OS/데이터 저장 방식)

이미지 파일 자체가 어떤 구조로 저장되어 있는지를 뜻한다.

| 포맷 | 설명 |
|---|---|
| RAW | 변환 없는 기본 포맷, 용량 크지만 빠름 |
| QCOW2 | QEMU 기반, 스냅샷·압축 지원. OpenStack 표준 |
| VHD | Hyper-V용 |
| VMDK | VMware용 |
| VDI | VirtualBox용 |
| ISO | CD/DVD 이미지 |

- **QEMU(Quick EMUlator)**: VM을 만들 때 KVM과 같이 씀. 소프트웨어로 CPU, 메모리 등 하드웨어 전체를 흉내냄.

```
Nova
 └── libvirt (드라이버)
      └── QEMU + KVM
               └── VM 실행
```

- **Hyper-V**: Microsoft가 만든 Windows 서버용 하이퍼바이저
- **하이퍼바이저**: 물리 서버 위에서 VM을 만들고 관리하는 소프트웨어 (KVM, Hyper-V, VMware ESXi)

### 2. 컨테이너 포맷 (디스크 이미지를 감싸는 방식)

그 이미지 파일을 어떤 컨테이너로 감싸고 있는지를 뜻한다.

| 포맷 | 설명 |
|---|---|
| bare | 컨테이너 없이 디스크 이미지만 |
| OVF | 가상 머신 배포 표준 포맷 |
| Docker | Docker 컨테이너 이미지 |

## 주요 구성 요소

- **Glance API**: 핵심 서비스. REST API로 이미지 업로드/다운로드/삭제 처리. 내부적으로 Database, Storage와 연계해서 동작.
- **Database**: 이미지 메타데이터(이름, 크기, 포맷, 체크섬) 저장·관리. 삭제된 이미지 메타데이터도 남아서 로그 역할 수행.
- **Storage**: 실제 이미지 파일 저장소. 로컬 파일시스템, Swift, S3, NFS 등 백엔드 선택 가능.

## VM 생성 시 Glance 동작 흐름

```
사용자가 Glance에 이미지 업로드
      ↓
Nova가 인스턴스 생성 요청 수신
      ↓
Nova API → Glance API 호출하여 이미지 메타데이터 조회
      ↓
Nova-compute가 Glance에서 이미지 파일 다운로드
      ↓
KVM+QEMU 하이퍼바이저로 인스턴스 부팅
      ↓
Nova가 Placement·Neutron과 연계하여 네트워크·자원 할당
      ↓
VM 정상 실행
```

**Placement**: Nova의 리소스 스케줄링을 담당하는 OpenStack 컴포넌트. 리소스 빈 공간을 찾아서 VM을 어느 컴퓨트 노드에 올릴지 결정한다.