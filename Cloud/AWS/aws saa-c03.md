

### Athena
- 서버리스
- 비용 효율성: RedShift보다 가성비 있음
- S3에 저장된 파일(CSV, JSON, Parquet)을 그 자리에서 바로 SQL로 분석할 수 있게 함

### AWS Glue
1. Glue Crawler: S3에 있는 파일을 보고 스키마를 파악함
2. Data Catalog: 크롤러가 파악한 스키마를 구조 메타데이터를 저장
3. Athena: Data Catalog를 보고 SQL 쿼리를 실행 가능

---

### AWS Global Accelerator
- Anycast IP 기반 전 세계 AWS 엣지 로케에 진입점을 두고 가장 가까운 리전으로 트래픽 라우팅
- 캐싱X, 네트워크 경로를 최적화해주는 서비스임
- TCP UDP
- 게임, VoIP, 금융 거래같은 저지연/비HTTP 프로토콜에 적합

### CloudFront
- CDN 캐싱 서비스
- HTTP/HTTPS
- 전 세계 AWS 엣지 로케에 진입점을 두고 가장 가까운 리전으로 트래픽 라우팅
- S3 버킷, ALB, EC2, API Gateway랑 짝꿍
- Cache Behavior: 경로별 캐싱 정책을 다르게 설정 가능
- OAC/OAI: S3를 CloudFront 전용으로만 열어두는 설정
- WAF 연동: CloudFront 앞단에 WAF 붙여서 SQL Injection, XSS 같은 공격 필터링
- 필드 레벨 암호화, Signed URL/Signed Cookie: 특정 사용자에게만 콘텐츠 접근 제한
- 지역 제한(Geo-restriction): 특정 국가 접근 차단/허용

### Route 53 지연 시간 기반 라우팅(Latency-Based Routing)
- 사용자 요청을 지연 시간이 가장 낮은 리전의 엔드포인트로 라우팅
- 여러 리전에 동일 서비스가 배포되어 있을 때 사용
- Global Accelerator와 결합하면 글로벌 저지연+다중 프로토콜

---

### AMI(Amazon Machine Image)
- EC2 인스턴스의 상태를 그대로 담은 이미지(OS, 설정, 애플리케이션을 포함)
- stateless 인스턴스에 적합: 도화지에 AMI 받으면 동일한 인스턴스가 되니까
- Auto Scaling 그룹의 시작 템플릿으로도 바로 활용
- EBS 스냅샷과 달리 인스턴스 전체를 백업하는 개념 (EBS 스냅샷은 볼륨 단위)

### Amazon RDS 자동 백업 (Automated Backups)
- RDS 인스턴스를 매일 자동으로 스냅샷 뜨고, 트랜잭션 로그도 5분 단위로 S3에 저장함
- 백업 보존 기간(1일~35일) 설정 가능
- PITR(Point-In-Time Recovery)의 기반이 되는 기능

### PITR(Point In Time Recovery)
- 자동 백업+트랜잭션 로그를 조합해서 특정 시점으로 DB를 복원하는 기능
- RPO(복구 지점 목표)를 맞출 수 있음. (ex. 90분 RPO 요구사항)
- stateful 데이터베이스 계층에 쓰이는 백업 전략

### EBS 스냅샷
- EC2에 연결된 EBS 볼륨의 특정 시점 백업
- 인스턴스에 지속적 로컬 데이터가 없으면 (stateless) 볼륨 스냅샷은 의미 없음
- 선지에 "90분마다 EBS 스냅샷"이 나오면 EBS의 데이터가 stateless인지 체크하기

### DLM(Data Lifecycle Manager)
- EBS 스냅샷 생성/보관/삭제를 자동 스케줄링해주는 서비스
- 스냅샷 자체가 불필요한 상황이면 DLM 역시 불필요함 (EBS의 데이터가 stateless인지 체크하기 / stateless이면 불필요)

---

### ECS on Fargate
- 컨테이너 오케스트레이션 서비스(ECS)를 서버리스로 실행하는 방식
- EC2 인스턴스를 직접 프로비저닝/스케일링 할 필요 없음 → 운영부담 최소화
- 컨테이너 단위로 모듈식 확장 가능 → 모놀리스를 서비스로 분리하는 마이크로 서비스 전환에 적합
- EKS는 컨트롤 플레인/노드 관리 오버헤드가 더 큼 → 저운영이 목표면 ECS Fargate가 유리함


### RDS Multi-AZ
- 기본 DB 인스턴스+다른 가용 영역의 대기 복제본을 자동 유지
- 장애 시 자동 페일오버로 고가용성 확보
- 관계형 스키마를 유지하면서 관리형으로 운영 → EC2에 DB 설치하는 것보다 운영 부담 낮음

### S3
- 비정형 객체(이미지, 파일)을 저장하는 데 최적화된 완전관리형 스토리지
- 거의 무제한 확장, 서버 관리 불필요 → "이미지 저장+저운영"
- RDS 같은 관계형 DB에 이미지를 직접 넣는 건 안티패턴

### DynamoDB
- NoSQL 키-값/문서 스토어 → 관계형 스키마를 요구하는 제품 데이터에는 부적합

---
### Amazon SageMaker
- 완전 관리형, ML지식이 필요한 머신러닝 플랫폼
- 데이터 전처리, 알고리즘 선택, 하이퍼파라미터 설정, 모델 학습, 배포까지 직접 구상해야 함
- ML 전문지식이 없으면 진입장벽이 있음
- SageMaker Endpoint: 학습된 SageMaker 모델을 실시간 추론할 수 있게 배포하는 API 엔드포인트

### Amazon Forecast
- 시계열 예측(수요, 재고, 리소스 요구량 예측 등)에 특화된 완전관리형 서비스 (노코드/로우 코드)
- S3에 있는 과거 데이터를 넣으면 AWS가 알아서 최적 알고리즘 선택 → 모델학습 → 예측생성 자동화
- Forecast Predictor: 모델 학습에 해당하는 단계

---
## Lambda

### 기본 개념
- 서버리스 컴퓨트 서비스: 서버 프로비저닝/관리 없이 코드만 업로드하면 실행됨
- 이벤트 기반(Event-driven): 뭔가 트리거가 발생해야 실행됨 (요청, 파일 업로드, 스케줄 등)
- 실행 후 자동으로 스케일 인/아웃, 안 쓰면 과금 없음 (호출당 과금)
- 최대 실행 시간: 15분 (그 이상 걸리는 작업엔 부적합 → Step Functions, ECS/Batch로)
- 메모리: 128MB ~ 10,240MB (메모리 늘리면 CPU도 비례해서 증가)

### 실행 환경/구조
- 핸들러(Handler): 이벤트 발생 시 호출되는 진입점 함수
- 실행 역할(Execution Role): Lambda가 다른 AWS 서비스에 접근할 때 쓰는 IAM 역할 (S3 읽기, DynamoDB 쓰기 등 권한 부여)
- 레이어(Layers): 공통 라이브러리/의존성을 별도로 패키징해서 여러 함수에서 재사용 (배포 패키지 크기 줄임)
- 환경 변수: 코드 밖에서 설정값 주입 (DB 엔드포인트, API 키 등)
- 콜드 스타트(Cold Start): 오랜만에 호출되면 컨테이너를 새로 띄우느라 지연 발생. Provisioned Concurrency로 미리 웜업 가능

### 호출 방식(트리거)





### Lamdba 함수 URL
- API Gateway 없이 직접 HTTPS 엔드포인트를 부여하는 기능
