# [SCP] VPC·서브넷 구성부터 Bastion(Jump Host) SSH 접속

> SCP에서 VPC를 직접 설계하고 Public/Private 서브넷을 나눠 Bastion 구조로 SSH 접속해 본 실습 기록

---

## 1. 최종 구성도

'''
aa
'''

---

## 2. CIDR / 서브넷 설계

### VPC 대역
- 'X.X.X.0/24' → IP 256개
- 기존 VPC, 연결될 VPC와 **대역이 겹치지 않게** 잡아야 함 (Peering, Transit Gateway 연결 시 문제)

### 비트로 보는 CIDR
'''
X        .X        .X        .0
XXXXXXXX .XXXXXXXX .XXXXXXXX .00000000
'''
- '/24' → 앞 24비트 고정, 뒤 8비트가 호스트
- '/26' → 마지막 옥텟 중 **앞 2비트는 서브넷 구분**, **뒤 6비트는 호스트**(64개)

| 앞 2비트 | 서브넷 | 범위 |
|---|---|---|
| 00 | X.X.X.0/26 | .0 ~ .63 |
| 01 | X.X.X.64/26 | .64 ~ .127 |
| 10 | X.X.X.128/26 | .128 ~ .191 |
| 11 | X.X.X.192/26 | .192 ~ .255 |

- 프리픽스 +1 → 서브넷 수 ×2, 서브넷당 IP ÷2
  - /24: 1개 × 256 / /25: 2 × 128 / /26: 4 × 64 / /27: 8 × 32
- '/32' = 호스트 비트 0개 → **IP 딱 1개** (SG에 "내 IP만 허용"할 때 사용)
- '0.0.0.0/0' = 전체 IP

### 서브넷 안에서 못 쓰는 IP
| IP | 역할 |
|---|---|
| 첫 번째 (호스트 비트 전부 0) | Network address |
| 두 번째 (.1) | Gateway (SCP 기본값) |
| 마지막 (호스트 비트 전부 1) | Broadcast address |
| 그 외 앞쪽 일부 | 클라우드 내부 예약 → '.2'와 '.66'이 "이미 등록된 IP"로 떴음 |

### 적용한 서브넷
| 서브넷 | 유형 | 구분 | 대역 | Gateway |
|---|---|---|---|---|
| subnet-A | Public | Primary | X.X.X.0/26 | X.X.X.1 |
| subnet-B | Private | Primary | X.X.X.64/26 | X.X.X.65 |

> **Primary vs Secondary**
> Secondary는 부모 Primary 서브넷의 유형을 그대로 물려받고, 같은 브로드캐스트 도메인을 공유함.
> → Public/Private처럼 **격리가 목적이면 별도 Primary**로 생성. Secondary는 IP 대역을 확장할 때 사용.

---

## 3. 작업 순서

### ① VPC 생성
- 'vpc3' / 'X.X.X.0/24'

### ② 서브넷 생성
- subnet-A (Public), subnet-B (Private)
- IP 할당 범위: IP 대역 전체
- DNS Name Server: 미사용 → SCP 기본 DNS가 자동 적용됨
- 호스트 경로: 미사용

### ③ Internet Gateway 생성
- vpc3에 연결 + **방화벽 사용** 체크 → 'FW_IGW_vpc3' 자동 생성

### ④ Security Group 생성
**SG_lcu (Bastion용)**
| 방향 | 대상 | 포트 |
|---|---|---|
| Inbound | '내 공인IP/32' | TCP 22 |

**SG2_lcu (Private VM용)**
| 방향 | 대상 | 포트 |
|---|---|---|
| Inbound | **Security Group: SG_lcu** | TCP 22 |

- SG의 Inbound "대상 주소" = **출발지**를 넣는 칸. VM 내부 대역을 넣으면 안 됨.
- 내 공인 IP는 'ipconfig'가 아니라 'curl ifconfig.me'로 확인. ('ipconfig'의 X.X.0.x는 공유기 내부 사설 IP)
- Bvm1용 SG의 대상을 IP 대신 **SG_lcu로 지정** → "SG_lcu가 붙은 VM에서 오는 트래픽 허용". Bastion IP가 바뀌어도 규칙 수정 불필요.
- CIDR로 넣는다면 Avm1의 **사설 IP**('X.X.X.3/32')를 넣어야 함. Avm1 → Bvm1은 VPC 내부 통신이라 NAT를 안 거치기 때문.

### ⑤ Keypair 생성
- 'A1': Bastion(Avm1)용
- 'B1': Private VM(Bvm1)용
- 키를 분리하면 Bastion 키가 유출돼도 내부 서버까지 바로 뚫리지 않음

### ⑥ Virtual Server 생성
| 항목 | Avm1 | Bvm1 |
|---|---|---|
| Image | Ubuntu 24.04 | Ubuntu 24.04 |
| 서버 타입 | s1v1m2 (vCPU 1 / 2GB) | 동일 |
| Subnet | subnet-A | subnet-B |
| IP | X.X.X.3 | X.X.X.67 |
| Public NAT | 사용 | 불가 (Private 서브넷) |
| SG | SG_lcu | SG2_lcu |
| Keypair | A1 | B1 |

- 서버 타입 이름 규칙: 'v' 뒤 = vCPU 수, 'm' 뒤 = 메모리(GB)
- 디스크: 실습용이면 **SSD_Provisioned 말고 일반 SSD** (Provisioned는 IOPS 보장형이라 비쌈) → 이번엔 실수로 Provisioned로 생성함
- **Public NAT 체크를 빼먹어서** 생성 후 상세 화면의 "Public NAT IP 수정"에서 추가함
- 목록에 있던 미사용 Public IP는 다른 사람이 예약한 것일 수 있어서 **신규 생성**으로 할당

### ⑦ IGW 방화벽 규칙 추가 ('FW_IGW_vpc3')
| 출발지 | 목적지 | 포트 | 방향 | 동작 |
|---|---|---|---|---|
| 내 공인 IP | X.X.X.3 (Avm1 사설 IP) | TCP 22 | Inbound | 허용 |

- 방화벽은 위에서부터 순서대로 매칭 → 규칙 위치 주의
- "추가" 후 **저장까지** 해야 반영됨

---

## 4. MobaXterm 접속

### Avm1 직접 접속
| 항목 | 값 |
|---|---|
| Remote host | Avm1 **공인 NAT IP** |
| Username | 'ubuntu' (SCP Ubuntu 이미지 기본 계정) |
| Use private key | A1.pem |

- 처음 접속 시 호스트 키 확인 창 → Accept (호스트 키를 저장해 두고 다음부터 서버 위조 여부를 검증)
- VM 안에서 보면 IP는 사설 IP(X.X.X.3)만 보임 → NAT는 IGW에서 일어남

### Bvm1 접속 (Jump Host)
| 위치 | 값 |
|---|---|
| Remote host | 'X.X.X.67' (Bvm1 사설 IP) |
| Username | 'ubuntu' |
| Advanced SSH settings → private key | **B1.pem** |
| Network settings → SSH gateway host | Avm1 **공인 NAT IP** |
| SSH gateway username / key | 'ubuntu' / **A1.pem** |

'''
내 PC ──▶ Avm1 공인 IP (A1 키) ──▶ X.X.X.67 Bvm1 (B1 키)
'''

- B1.pem을 Bastion에 **복사하지 않음**. 키는 전부 내 PC에만 있고 Avm1은 터널로만 사용.
- Bvm1 세션은 Avm1 세션과 **독립적** → Avm1 창을 닫아도 동작. 한 세션 안에서 "Avm1 경유 → Bvm1 접속"이 자동으로 일어남.
- 두 VM 모두 username이 'ubuntu'지만 서버마다 별개 계정이라 겹쳐도 문제없음.

---

## 5. 개념 정리

### 공인 IP vs 사설 IP vs NAT
- **Public IP**: SCP 리전의 공인 IP 풀에서 할당받는 주소 (직접 고를 수 없음)
- **NAT**: 그 공인 IP를 VM의 사설 IP로 1:1 변환해 주는 방식 (IGW에서 수행)
- 전화 비유
  - 공인 IP = 대표번호 / 사설 IP = 내선번호 / IGW = 교환기
  - 밖에서 걸면 교환기를 거쳐 내선으로 연결 → **공인 IP 사용**
  - 회사 안에서 내선끼리 걸면 교환기 안 거침 → **사설 IP 사용**
- 결론: **사설 IP로는 VPC 안에 있는 서버만 접속할 수 있다.**

### SG vs IGW 방화벽
| | Security Group | IGW Firewall |
|---|---|---|
| 적용 대상 | VM (네트워크 포트) | VPC ↔ 인터넷 구간 |
| 목적지 | 이미 정해져 있음 (붙은 VM) | 직접 지정 |
| 외부 → VM | 필요 | 필요 |
| VPC 내부 VM ↔ VM | 필요 | 안 거침 |

→ 외부에서 SSH가 되려면 **둘 다** 열려 있어야 함. 하나만 열면 타임아웃.

### Bastion은 "역할"이다
- Avm1 혼자 있을 때 → 그냥 공인 IP가 있는 VM
- 뒤에 공인 IP 없는 Bvm1이 생기고, **Bvm1에 가려면 반드시 Avm1을 거쳐야** 할 때 → Avm1이 Bastion
- 목적지가 자기 자신이 아니라 **뒤에 있는 서버들로 가는 경유지**라는 점이 핵심
- 외부 노출을 Bastion 하나로 줄여서 IP 제한, 접속 로그, 계정 관리를 한 곳에 집중할 수 있음

---

## 6. 트러블슈팅 메모
| 증상 | 원인 | 해결 |
|---|---|---|
| IP 'X.X.X.2' "이미 등록된 IP" | SCP 내부 예약 | IP 자동 할당 또는 다른 IP 사용 |
| MobaXterm에 사설 IP 입력 | VPC 밖에서는 사설 IP로 접근 불가 | 공인 NAT IP 입력 |
| Public NAT IP가 '-' | 생성 시 NAT 체크 누락 | VM 상세 → Public NAT IP 수정 |
| SG 대상에 서브넷 대역 입력 | 대상 = 출발지 칸인데 목적지를 넣음 | '내 공인IP/32'로 수정 |
| Jump host 타임아웃 (예상) | SG_lcu Outbound가 막혀 Avm1 → Bvm1 불가 | SG_lcu Outbound: SG2_lcu, TCP 22 추가 |

---

## 7. 다음에 해볼 것
- [ ] Private VM에서 'sudo apt update' → 실패 확인 → **NAT Gateway** 연결해서 Outbound만 허용하는 구조 실습
- [ ] Public NAT IP(1:1, 양방향) vs NAT Gateway(N:1, Outbound 전용) 비교
- [ ] 서브넷 간 'ping' 테스트 (ICMP 규칙)
- [ ] Avm1에 nginx, Bvm1에 DB를 올려 2-tier 구성
- [ ] 실습 끝나면 VM, Public IP 반납 (미사용 공인 IP도 과금될 수 있음)
