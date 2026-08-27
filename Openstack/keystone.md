# Keystone
2026.08.27

Keystone은 당신은 누구(Authentication)고, 뭘 하러 왔는지 판단하고 권한을 파악(Authorization)하는 인증 서비스다.

![alt text](/assets/Openstack/keystone1.png)

사용자 인증, 권한 부여, API 접근 제어 등을 관리하는 서비스

## 주요 기능

- 인증 토큰 발급 및 관리
- 역할 기반 접근 제어(RBAC) 구현
- 서비스 카탈로그 제공
- 연계: OpenStack의 모든 서비스에 대해 인증 및 권한 관리 기능 제공

![alt text](/assets/Openstack/keystone2.png)

## Keystone 개념 활용

아래에 설명하는 개념들은 개념으로 끝나지 않고 실제로 쓰이는 명칭이다.

### OpenStack CLI 명령어

```
openstack domain list
openstack project list
openstack user list
openstack group list
openstack role list
openstack role assignment list
openstack token issue
openstack catalog list
```

### API URL

**v3**

Keystone의 API 버전

원래 v2.0이 있었는데, deprecated돼서 지금은 v3만 사용함

**업데이트 된 점**

- v2.0: Project와 User가 전역으로 공개
- v3: Domain 개념이 추가되어 Project, User와 함께 전역으로 공개

```
/v3/domains
/v3/projects
/v3/users
/v3/groups
/v3/roles
/v3/auth/tokens
```

## Keystone 세부 개념

### Domain

여러 조직을 논리적으로 격리하는 최상위 단위이다.

이때 조직은 별개의 사용자 집단이다. 예를 들어 현대오토에버가 Openstack 하나를 운영하고 두 팀에 제공한다고 하면, 팀 A와 팀 B가 Openstack을 쓰는 별개의 조직인 것이다.

초기 OpenStack은 프로젝트 사용자가 전역적으로 공개되어 조직 간 이름 충돌 문제가 있었는데, 이를 해결하기 위해 Domain이 도입됐다.

```
팀 A가 "dev"라는 프로젝트를 만듦
팀 B도 "dev"라는 프로젝트를 만들고자 함
→ 조직 이름 충돌로 못 만듦!!
```

조직을 분리하는 Domain을 도입해서 별개로 격리시켜 해결함. 서로의 리소스는 볼 수 없음.

```
[Domain: 팀 A]        |    [Domain: 팀 B]
   project: dev       |        project: dev
   user: chaeun        |       user: chaeun
```

Domain은 여러 조직을 동시에 수용하기 위해 필요한 개념이기 때문에, 소규모 환경에선 보통 Default 도메인 하나만 사용한다. Kolla-ansible은 조직이 하나뿐일 때를 대비해서 Default 도메인을 자동으로 만들어주고, 그 안에 admin 같은 기본 계정들을 제공한다.

**주의사항**

1. **새 도메인을 만들면 기본 계정이 없다.**
   새로운 도메인을 만들면 기본 계정은 없다. Default 도메인에만 Kolla-Ansible이 자동으로 admin, nova, glance, neutron 등 서비스 계정들을 넣어주고, 새로 생성한 도메인은 비어있어 직접 구성해야 한다.

2. **Default는 건드리지 말고 추가로 만들어야 한다.**
   만약 개발팀, 운영팀처럼 여러 조직에 도메인을 나눠주고 싶다면 Default는 건드리지 않고 도메인을 추가로 생성해야 한다.
   OpenStack 서비스 계정들(nova, glance 등)이 전부 Default 도메인에 묶여 있기 때문에 이름을 바꾸거나 건드리면 서비스 인증이 꼬일 수 있다. default라는 이름도 코드 곳곳에 하드코딩되어 있는 경우가 있어서 바꾸면 예상치 못한 곳에서 오류가 발생할 수 있다.

![alt text](/assets/Openstack/keystone3.png)

### Project

VM, 네트워크, 이미지 등 리소스를 그룹화하고 격리하는 단위이다. 초기에는 임차인을 뜻하는 Tenant라고 불렸지만 더 직관적인 project로 변경됐다.

**리소스**

OpenStack에서 관리 대상이 되는 모든 것

- VM (인스턴스)
- 네트워크
- 서브넷
- 이미지
- 볼륨 (블록 스토리지)
- 플로팅 IP
- 보안 그룹
- 키페어

**리소스는 반드시 특정 프로젝트에 속한다.**

OpenStack에서 VM을 만들면 그 VM은 반드시 어느 프로젝트의 소속이다. 만약 team-dev와 team-ops가 있을 때, team-dev의 VM은 team-ops에서 보이지 않는다.

**프로젝트는 사용자를 직접 소유하지 않는다.**

도메인이 사용자를 소유하고, Keystone의 Role Assignment를 통해 사용자가 특정 프로젝트에 대한 접근 권한을 갖게 된다. 만약 chaeun이라는 유저가 도메인 A에 들어있고, B, C라는 프로젝트 두 개가 있다고 하자. chaeun은 B에서는 member, C에서는 admin 역할을 가질 수 있는 것이다.

##### CSP 프로젝트에서의 쓰임
```
포털 백엔드 → Keystone API로 프로젝트 자동 생성
POST /v3/projects  { name: "user-chaeun-01" }

해당 유저에게 그 프로젝트의 member 역할 부여
PUT /v3/projects/{id}/users/{user_id}/roles/{role_id}

Nova API로 VM 생성 (해당 프로젝트 소속으로)

관리자는 프로젝트 단위로 쿼터 설정, 자원 회수 가능
```

![alt text](/assets/Openstack/keystone4.png)


### Actors (User and User Groups)

실제로 역할을 부여받는 주체다.

도메인을 설명할 때 도메인이 다르면 같은 이름의 유저가 존재할 수 있다고 했는데, 내부적으로는 UUID로 구분하기 때문에 충돌이 없다. v2.0에서도 UUID는 존재했지만, 개발자는 UUID로 이름을 찾지 않고 이름으로 찾기 때문에 문제가 생기는 것이다.

#### User

클라우드를 직접 사용하는 개인이다. OpenStack에 로그인하고 VM을 만들고 네트워크를 구성하는 실제 사람 또는 서비스를 의미한다.

- **서비스**: OpenStack 서비스들끼리 서로 API 호출할 때 쓰는 계정이다.
  예를 들어 VM을 만들 때 Nova는 이미지를 가져오려고 Glance API를 호출해야 한다. 이때 Nova가 Keystone한테 Nova인 것을 인증받아야 하는데, 그때 쓰는 계정이 서비스 계정이다.
- `openstack user list` 명령어: nova, glance, neutron, cinder, heat, placement와 같은 서비스 계정이 나온다.

#### Group

사용자들의 집합이다. 역할을 부여할 때 유저 한 명씩 설정하는 게 아니라 그룹 단위로 한 번에 처리할 수 있다.

```
Group: dev-team
  └─ chaeun
  └─ A
  └─ B

dev-team 그룹에 member 역할 부여
→ chaeun, A, B 전원에게 자동 적용
```

만약 위 그룹(member 역할)에서 A에 admin 역할을 부여하면 그룹에서 A만 빠지지 않고, 그룹 내에서 A만 admin, chaeun과 B는 member인 채로 있는다.

C 유저가 그룹에 추가되면 그룹 자체는 member 역할이기 때문에 member를 부여받는다.

**역할은 유저 생성부터 무조건 배정될까?**

- Horizon에서 만들면 역할 배정이 필수
- CLI에서 만들면 역할 없이 생성 가능

### Roles

권한 부여의 단위다.

Role은 이름표일 뿐이고, Keystone이 admin이라는 이름표를 붙여줬다고 해서 바로 권한이 생기는 게 아니다. 각 서비스가 policy.yaml을 보고 admin 이름표를 가진 사람이 할 수 있는 일을 알 수 있는 구조이다.

**기본 제공 역할**

| 역할 | 설명 |
|---|---|
| admin | 모든 걸 할 수 있는 최고 권한. 유저 생성, 프로젝트 삭제, 쿼터 수정 등 관리자 작업 전부 가능. |
| member | 일반 사용자 역할. 본인 프로젝트 안에서 VM 생성, 네트워크 구성 등 실제 작업은 할 수 있지만 관리자 작업은 불가. |
| reader | 읽기 전용. 조회만 가능하고 생성/수정/삭제는 불가. |

**policy.yaml**

Nova, Glance, Neutron 등 서비스마다 각자 policy.yaml을 가지고 있고, 거기서 역할별로 허용/거부를 정의한다.

```yaml
# 예시
"os_compute_api:servers:create": "role:member"  # member 이상만 VM 생성 가능
"os_compute_api:servers:delete": "role:admin"   # admin만 VM 삭제 가능
```

역할 생성이 가능하다.

```
openstack role create vm-only
```

```yaml
# Nova policy.yaml
"os_compute_api:servers:create": "role:vm-only"  # vm-only 역할만 VM 생성 가능
```

![alt text](/assets/Openstack/keystone5.png)

### Assignment

Keystone이 "누가, 어디서, 무슨 역할"을 가지는지 저장하는 단위가 Assignment다. Actor + Target + Role 세 가지가 반드시 있어야 성립한다.

**Targets**: 역할이 유효한 범위로 Project와 Domain을 말한다.

```
chaeun(Actor) + project-dev(Target) + member(Role)
→ Assignment: chaeun은 project-dev에서 member다.
```

**grant와 revoke**

아래 코드는 Actor + Target + Role 세 가지로 grant와 revoke하는 코드이다.

```
# grant
openstack role add member --user chaeun --project project-dev

# revoke
openstack role remove member --user chaeun --project project-dev
```

revoke를 하면 chaeun은 project-dev에 접근 불가하다!!!! 🙀

### Token

인증 성공 후 발급되는 인증토큰이다. OpenStack의 모든 API 요청은 이 토큰을 헤더에 담아서 보낸다.

**토큰에 담긴 정보**

```
사용자 정보          ← 누가 인증했는지
issued_at            ← 언제 발급됐는지
expires_at           ← 언제 만료되는지
프로젝트 정보         ← 어떤 프로젝트(타겟)에 유효한지
서비스 카탈로그       ← 각 서비스 URL 목록
```

**생성 - Fernet Token 방식**

현재 OpenStack이 사용하는 토큰 생성 및 검증 방식이다. Kilo 릴리즈에서 도입되어 Ocata 릴리즈부터 기본 토큰 방식으로 지정됐다.

암호화와 복호화에 동일한 키를 사용하는 대칭키 방식이다. 토큰 자체는 DB에 저장하지 않고 사용자에게 전달하고 끝이다. 나중에 그 토큰이 API 요청에 담겨오면 `/etc/keystone/fernet-keys/`에 저장된 키 파일로 복호화해서 유효한지 확인한다. 서버가 토큰을 저장하지 않는 Stateless 방식이다.

이전 방식인 UUID Token은 발급할 때마다 DB에 저장했기 때문에 토큰이 많아질수록 DB가 무거워지는 문제가 있었다.

**인증 - Bearer Token 방식**

토큰을 가진 사람이면 누구든 사용할 수 있는 인증 방식이다. 토큰을 탈취당하면 탈취한 사람도 그 토큰으로 API를 호출할 수 있다는 뜻이다. 그래서 토큰에 만료 시간이 있고, 기본값은 1시간이다.

### Catalog

OpenStack 서비스들의 엔드포인트 목록이다. 토큰 발급 시 함께 반환되며, 클라이언트는 이걸 보고 각 서비스에 어디로 요청을 보낼지 결정한다.

카탈로그가 없으면 클라이언트는 Nova가 어디 있는지, Glance가 어디 있는지 알 수 없다. VM 생성 요청을 어디로 보낼지 모르는 거다.

**엔드포인트 구분**

각 서비스의 엔드포인트는 세 가지 URL로 나뉜다.

| 구분 | 설명 |
|---|---|
| admin | 관리자용. 관리자 작업에 사용 |
| internal | 서비스끼리 내부 통신에 사용 |
| public | 외부 사용자가 접근하는 URL |

Nova, Glance, Neutron 등 서비스마다 이 세 가지 URL이 카탈로그에 등록되어 있다.

```
openstack catalog list
```

---

참고: [YouTube 강의 영상](https://www.youtube.com/watch?v=8KNYUWaY_08&list=PL1fB0Roqrr0vPh2dRfSYEpBWCyKFMGoQ9&index=16)