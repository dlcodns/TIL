# [SCP] Bastion 경유 SSH가 안 붙을 때 - SG Outbound 누락

## 상황
```
내 PC → Avm1 (Bastion, X.X.X.3) → Bvm1 (Private, X.X.X.67)
          SG_lcu                        SG2_lcu
```
- Avm1 직접 접속은 정상
- Bvm1 접속(Jump Host)은 빈 화면에서 멈춤

## 원인 확인
Avm1에서 Bvm1의 22번 포트 확인
```bash
nc -zv X.X.X.67 22
```
- 멈춤 / timed out → 네트워크(SG) 차단
- succeeded → 네트워크는 정상, MobaXterm 설정 문제

**원인**
SG_lcu에 규칙이 **Inbound(내 IP → 22) 1개뿐**이고 Outbound가 없었음

```
Avm1 [SG_lcu Outbound 22] ──▶ [SG2_lcu Inbound 22] Bvm1
          ❌ 없음                      ✅ 있음
```

## 해결
SG_lcu에 Outbound 규칙 추가

| 대상 | 프로토콜/포트 | 방향 |
|---|---|---|
| Security Group: SG2_lcu | TCP 22 | Outbound |

```bash
ubuntu@avm1:~$ nc -zv X.X.X.67 22
Connection to X.X.X.67 22 port [tcp/ssh] succeeded!
```

## 배운 점
- VM 간 통신은 **보내는 쪽 Outbound + 받는 쪽 Inbound** 둘 다 열려야 함
- 내 PC → Avm1은 Outbound 없이도 됐음 → 들어온 연결의 **응답은 자동 허용**
- Avm1 → Bvm1으로 가는 Outbound 필요
