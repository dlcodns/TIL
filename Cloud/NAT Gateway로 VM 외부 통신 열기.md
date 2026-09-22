# [SCP] NAT Gateway로 Private VM 외부 통신 열기

> Bastion 구조에서 공인 IP가 없는 Private VM(Bvm1)이 `apt update`를 할 수 있도록 NAT Gateway를 붙여 본 실습

---

## 1. 구성

```
                 ┌──▶ ✔️.✔️.✔️.✔️ (Public NAT IP) ◀──▶ Avm1 (X.X.X.3, subnet-A)
인터넷 ◀── IGW ──┤
                 └──◀ ⭕.⭕.⭕.⭕ (NAT Gateway)   ◀─── Bvm1 (X.X.X.67, subnet-B)
                                                  (나가기만 가능)
```

| | Avm1 (Bastion) | Bvm1 (Private) |
|---|---|---|
| 공인 IP | ✔️.✔️.✔️.✔️ (Public NAT, 1:1) | 없음 |
| 외부 → VM | 가능 | 불가 |
| VM → 외부 | 가능 | **NAT Gateway ⭕.⭕.⭕.⭕로 가능** |

---

## 2. 시작 상태: Bvm1은 밖으로 못 나감

<img height="158" alt="image" src="https://github.com/user-attachments/assets/d98a5562-ec10-473a-b00f-aa39b51bbe1f" />

```bash
ubuntu@bvm1:~$ sudo apt update
Ign:1 ... Err:1 ...
Could not connect to ...:80 (X.X.X.95), connection timed out
W: Failed to fetch ...
W: Some index files failed to download.
```

### apt update 로그 읽는 법
| 표시 | 의미 |
|---|---|
| `Hit:` | 성공 (변경 없음) |
| `Get:` | 성공 (새로 받음) |
| `Ign:` | 넘어감 (재시도 중) |
| `Err:` | 실패 |

- 성공하면 마지막에 `Fetched ... kB` 가 찍힘
- `All packages are up to date`는 새 목록을 못 받아서 그냥 예전 목록 기준으로 판단한 것
- 도메인이 IP로 바뀌어 있으면 **DNS는 정상** (SCP 기본 DNS)
- IPv6 `Network is unreachable`는 IPv6 미사용이라 무시

---

## 3. Public IP 예약

- 구분: **Internet Gateway** 선택
  - VPC가 실제로 쓰는 게이트웨이 타입과 맞아야 통신 가능
  - Secured Internet Gateway는 보안 기능이 추가된 별도 게이트웨이 → vpc3에서는 통신 안 됨
- Avm1의 ✔️.✔️.✔️.✔️는 재사용 불가 → **공인 IP 하나는 한 리소스에만 연결 가능**
  - 떼면 Avm1 Bastion 접속이 끊김
- Bvm1의 NAT Gateway는 새로 만들어서 ⭕.⭕.⭕.⭕ 예약

### 💫Public IP / Public NAT / NAT Gateway
| 메뉴 | 정체 | 비유 |
|---|---|---|
| Public IP 예약 | 공인 IP **주소 자체** | 전화번호 개통 |
| VM의 Public NAT | VM **한 대 전용** 연결 (1:1, 양방향) | 개인 직통번호 |
| NAT Gateway | **여러 VM이 나갈 때 공유** (N:1, 나가기 전용) | 회사 대표번호로 발신만 |

---

## 4. NAT Gateway 생성

subnet-B(Private)에  공인 IP ⭕.⭕.⭕.⭕ 사용해서 생성 → `NAT_GW_subnet-B`

---

## 5. NAT Gateway가 Active여도 실패한 이유 - outbound 규칙 만들기

**안 된 이유**

```
Bvm1 ──[SG2_lcu Outbound]──▶ NAT GW ──[FW_IGW_vpc3 Outbound]──▶ 인터넷
          🔒 없음                           🔒 없음
```

### SG2_lcu (Bvm1)
| 방향 | 대상 | 포트 |
|---|---|---|
| Outbound | `0.0.0.0/0` | TCP 80, 443 |

### FW_IGW_vpc3
| 출발지 | 목적지 | 포트 | 방향 | 동작 |
|---|---|---|---|---|
| `X.X.X.64/26` (subnet-B) | `0.0.0.0/0` | TCP 80, 443 | Outbound | 허용 |

- 출발지를 **사설 대역**으로 넣고 성공 → SCP 방화벽은 **NAT 변환 전 사설 IP 기준**으로 판단
- `apt`는 HTTP/HTTPS를 쓰므로 80, 443이면 충분

### 왜 목적지가 전체(0.0.0.0/0)일까?
- 우분투 저장소 도메인 하나에 IP가 여러 개고, **언제든 바뀌거나 늘어남**
- 그래서 "목적지는 전체, **포트는 80/443만**"으로 제한하는 게 일반적

> 보안이 중요한 환경(공공 등)은 `0.0.0.0/0` 대신 **프록시 서버**나 **내부 미러 저장소**를 사용해 나가는 곳을 통제

---

## 6. 성공

<img width="500" alt="image" src="https://github.com/user-attachments/assets/97983ecf-6199-4003-87a7-536bedbe7b9b" />

<img width="500" alt="image" src="https://github.com/user-attachments/assets/a6444084-dc9c-402e-9528-78857d488246" />


```bash
ubuntu@bvm1:~$ sudo apt update
Get:1 http://security.ubuntu.com/ubuntu noble-security InRelease [126 kB]
Hit:2 http://kr-west1-b.clouds.archive.ubuntu.com/ubuntu noble InRelease
...
Fetched 37.1 MB in 14s (2750 kB/s)
316 packages can be upgraded.
```

### 어느 IP로 나가는지 확인
```bash
ubuntu@avm1:~$ curl ifconfig.me   #결과 ✔️.✔️.✔️.✔️ (Public NAT)
ubuntu@bvm1:~$ curl ifconfig.me   #결과 ⭕.⭕.⭕.⭕ (NAT Gateway)
```
→ 두 서버가 **다른 길로 나감**

---

## 7. 정리
- Private 서브넷의 기준은 NAT GW 유무가 아니라 **외부에서 VM으로 직접 들어올 수 있냐**
  - NAT GW 없음: 완전 폐쇄망 (외부 ↔ VM 모두 불가)
  - NAT GW 있음: 나가기만 가능 (패치, 외부 API 호출용) → 실무에서 일반적
- NAT GW는 나갈 때 출발지를 공인 IP로 바꾸고(SNAT), **응답만** 돌려보냄. 외부에서 먼저 들어오는 건 버림
- 통신이 되려면 **경로(NAT GW) + SG Outbound + 방화벽 Outbound** 셋 다 필요

## 8. 다음에 해볼 것
- [ ] SG2_lcu에 ICMP Outbound 추가 후 `ping` 확인
- [ ] Avm1에 nginx, Bvm1에 DB를 올려 2-tier 구성
- [ ] 실습 후 NAT Gateway, Public IP 반납
