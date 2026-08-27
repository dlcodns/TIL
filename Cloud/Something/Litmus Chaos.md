# Litmus Chaos를 쓸 때 k6를 활용해야 하는 이유

> Introduction to k6 Load Chaos in LitmusChaos

Litmus로 파드 삭제, 노드 격리, 네트워크 지연 등의 장애는 발생시킬 수 있지만 트래픽과 이체, 조회 등의 실제 서비스 요청은 발생시킬 수 없음.

- **k6-loadgen**으로 트래픽을 일으키고
- **Litmus Chaos**로 장애를 발생시키고
- **Grafana**로 결과를 확인해야 함.

```
k6-loadgen 시작: 이체/인증/조회 요청 계속 발송
        ↓
Litmus Chaos로 pod-delete 시작: pay-pod 10초마다 삭제
        ↓
Grafana에서 확인:
  - 요청 성공률 변화
  - 잔액 정합성 유지 여부
  - 파드 재생성 시간
```

---

## 시나리오 1. (Private) PC_1 한 대 다운

**Litmus 주입**
```
node-drain 또는 물리 네트워크 차단
```

**기대 동작**
```
1. Nova가 PC_1 장애 감지
2. PC_1 위의 VM → PC_2로 자동 Evacuation
3. K8s가 해당 노드 파드 → 나머지 노드 재스케줄
4. Keepalived가 VIP를 PC_2로 인수
5. HAProxy가 PC_1을 백엔드 풀에서 자동 제거
6. k6-loadgen 요청 처리 지속
```

**검증 지표**
- 서비스 중단 시간 5분 이내
- 이체 중 잔액 유실 없음
- Grafana에서 노드 장애 → VIP 전환 → 복구 흐름 확인

**Alert 설정**

```yaml
# 2단계: 노드 다운 감지 즉시
- alert: NodeDown
  expr: up{job="node-exporter"} == 0
  for: 30s
  labels:
    severity: critical   # 2단계
  annotations:
    summary: "PC_1 노드 다운 감지"

# 3단계: VIP 인수 실패 (API 응답 없음)
- alert: OpenStackAPIDown
  expr: probe_success{job="openstack-api"} == 0
  for: 1m
  labels:
    severity: fatal      # 3단계
```

---

## 시나리오 2. (Public) 백엔드 파드 전체 다운

**장애 등급**
```
2단계 [위험] → 퍼블릭 워크로드 전체 중단
```

**Litmus 주입**
```
pod-delete (AWS 측 파드 전체, CHAOS_INTERVAL=5)
```

**기대 동작**
```
1. K8s ReplicaSet이 파드 재생성 시도
2. 재생성 완료 전까지 온프레미스로 트래픽 자동 전환
3. HAProxy 헬스체크 실패 → 퍼블릭 백엔드 풀 제거
4. 파드 복구 완료 → 퍼블릭 백엔드 풀 자동 재등록
5. S3에 저장된 처리 결과 정합성 유지
```

**검증 지표**
- 파드 재생성 완료 시간 측정
- 온프레미스 전환 중 요청 손실 없음
- Grafana 파드 수 변화 추이 확인
- S3 백업 정합성 확인

**Alert 설정**

```yaml
# 2단계: 파드 수 0개 감지
- alert: AllPodsDown
  expr: kube_deployment_status_replicas_available == 0
  for: 30s
  labels:
    severity: critical   # 2단계
  annotations:
    summary: "퍼블릭 백엔드 파드 전체 다운"
```

---

## 시나리오 3. (Public) 리소스 75% 초과

**장애 등급**
```
1단계 [경고] → 리소스 사용량 70% 돌파 (사전 경보)
2단계 [위험] → 75% 초과 (버스팅 트리거)
```

**주입 방법**
```
k6-loadgen stages 고부하 단계로 설정
또는 stress-ng Litmus fault
```

**기대 동작**
```
[1단계 - 70% 도달]
1. AlertManager → Slack 경고 알림 발송
2. 운영자 인지 및 모니터링 강화

[2단계 - 75% 초과]
1. 버스팅 트리거 자동 실행
2. 신규 워크로드 → AWS EC2로 자동 전환
3. 온프레미스 부하 분산
4. Grafana에서 온프레미스/AWS 부하 분산 확인
```

**검증 지표**
- 70% 알림 발송 시간 측정
- 75% 초과 → 버스팅 전환 소요 시간
- 버스팅 전환 중 응답 지연 변화
- 온프레미스 CPU 정상화 확인

**Alert 설정**

```yaml
# 1단계: CPU 70% 경고
- alert: CPUWarning
  expr: avg(rate(node_cpu_seconds_total[5m])) > 0.70
  for: 1m
  labels:
    severity: warning    # 1단계
  annotations:
    summary: "리소스 사용량 70% 돌파 - 버스팅 대기"

# 2단계: CPU 75% 버스팅 트리거
- alert: CPUCritical
  expr: avg(rate(node_cpu_seconds_total[5m])) > 0.75
  for: 30s
  labels:
    severity: critical   # 2단계
  annotations:
    summary: "리소스 75% 초과 - 버스팅 전환 실행"
```

---

## 시나리오 4. (ALL) 트래픽 폭주

**장애 등급**
```
1단계 [경고] → 초당 요청 수 임계값 초과
2단계 [위험] → 응답 지연 급증, 큐 적체 시작
```

**주입 방법**
```
k6-loadgen stages 폭증 단계 (target: 500+)
```

**기대 동작**
```
[온프레미스]
1. HAProxy가 요청 분산
2. K8s HPA가 파드 자동 스케일아웃
3. Redis 캐시로 DB 부하 흡수

[한계 도달 시]
4. 온프레미스 CPU 75% 초과 감지
5. 신규 요청 AWS로 자동 버스팅
6. 트래픽 정상화 시 AWS 워크로드 자동 회수

[데이터 레이어]
7. 이체 요청 동시성 제어 (FOR UPDATE)
8. 잔액 정합성 유지
```

**검증 지표**
- 폭증 구간 응답 시간 500ms 이내 유지
- 이체 중 잔액 음수 발생 없음
- Redis 캐시 히트율 80% 이상 유지
- 버스팅 전환 소요 시간

**Alert 설정**

```yaml
# 1단계: 응답 지연 경고
- alert: HighLatency
  expr: http_request_duration_seconds > 0.3
  for: 1m
  labels:
    severity: warning    # 1단계

# 2단계: 응답 지연 위험
- alert: CriticalLatency
  expr: http_request_duration_seconds > 0.5
  for: 30s
  labels:
    severity: critical   # 2단계
```

---

## 시나리오 5. (Public) Queue/Cache 서버 다운

**장애 등급**
```
Queue 서버 다운 → 2단계 [위험]
Cache 서버 다운 → 1단계 [경고] → 2단계 [위험]
```

**Litmus 주입**
```
pod-delete (RabbitMQ 파드 또는 Redis 파드)
```

**Queue 서버 다운 시 기대 동작**
```
1. 큐에 대기 중인 메시지 Persistent Storage 보존
2. RabbitMQ 클러스터 나머지 노드로 Failover
3. 메시지 유실 없이 처리 재개
4. 복구 후 대기 메시지 순서대로 처리
5. S3에 큐 스냅샷 백업
```

**Cache 서버 다운 시 기대 동작**
```
1. Redis 다운 감지
2. 조회 요청 → DB로 자동 fallback
3. DB 부하 급증 모니터링
4. Redis 복구 후 캐시 자동 워밍업
5. 캐시 히트율 정상화 확인
```

**Alert 설정**

```yaml
# 1단계: Redis 다운 (Cache fallback 시작)
- alert: RedisDown
  expr: redis_up == 0
  for: 30s
  labels:
    severity: warning    # 1단계
  annotations:
    summary: "Redis 다운 - DB fallback 전환"

# 2단계: RabbitMQ 다운 (메시지 유실 위험)
- alert: RabbitMQDown
  expr: rabbitmq_up == 0
  for: 30s
  labels:
    severity: critical   # 2단계
  annotations:
    summary: "RabbitMQ 다운 - 큐 메시지 보존 확인 필요"
```

---

## 시나리오 6. (ALL) Public/Private 각각 다운 시 데이터 싱크

**장애 등급**
```
한쪽 다운     → 2단계 [위험]
양쪽 동시 다운 → 3단계 [망했다!]
```

**주입 방법**
```
Tailscale VPN 인터페이스 차단
또는 온프레미스 전체 네트워크 격리
```

**Private 다운 시 기대 동작**
```
1. AWS 단독으로 서비스 지속
2. 온프레미스 복구 후 S3 → 온프레미스 DB 동기화
3. 싱크 완료 전까지 AWS를 단독 Source of Truth로 운영
4. 데이터 정합성 검증 후 정상 운영 전환
```

**Public 다운 시 기대 동작**
```
1. 온프레미스 단독으로 서비스 지속
2. VPN 복구 후 온프레미스 → S3 자동 백업 재개
3. S3 미전송 구간 재동기화
4. 정합성 검증 후 정상 운영 전환
```

**검증 지표**
- VPN 복구 후 S3 동기화 완료 시간
- 복구 후 온프레미스 DB ↔ S3 레코드 수 일치 확인
- 싱크 구간 데이터 유실 없음

**Alert 설정**

```yaml
# 2단계: VPN 단절
- alert: VPNDisconnected
  expr: tailscale_connection_status == 0
  for: 1m
  labels:
    severity: critical   # 2단계
  annotations:
    summary: "온프레미스-AWS VPN 단절 - 데이터 싱크 중단"

# 3단계: 양쪽 동시 장애
- alert: HybridTotalFailure
  expr: tailscale_connection_status == 0 and node_up == 0
  for: 30s
  labels:
    severity: fatal      # 3단계
  annotations:
    summary: "하이브리드 전체 장애 - 즉시 대응 필요"
```

---

## 등급별 Alert 채널 요약

```
1단계 [경고]  → Slack #infra-warning 채널
               운영자 확인 후 대응

2단계 [위험]  → Slack #infra-critical 채널
               + 담당자 DM
               즉시 대응 필요

3단계 [망했다!] → Slack #infra-fatal 채널
               + 전체 팀 호출
               + 비상 대응 절차 실행
```