
<details>
<summary>서비스 이론</summary>


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
| 모델 | 설명 | 예시 |
|---|---|---|
| 동기(Synchronous) | 호출자가 응답 기다림 | API Gateway, Function URL, ALB |
| 비동기(Asynchronous) | 호출자는 바로 리턴, Lambda는 큐에 넣고 나중에 처리, 실패 시 재시도 | S3 이벤트, SNS, EventBridge |
| 폴링 기반(Poll-based) | Lambda가 소스를 직접 폴링 | SQS, DynamoDB Streams, Kinesis |

### Lambda Function URL
- *그냥 함수 하나 빨리 호출할 때*
- Lambda 함수에 API Gateway 없이 직접 HTTPS 엔트포인트를 부여하는 기능
- 설정 간단, 추가 비용 없음(Lambda 요금만)
- 지원 기능: 기본 CORS, IAM 인증 또는 퍼블릭
- 없는 기능: 요청 스로틀링 세밀 제어, 커스텀 도메인 매핑, 캐싱, 요청/응답 변환, API 키 관리, 사용량 플랜, WAF 직접 연동
- 적합한 경우: 단일 Lambda 함수를 간단히 호출하고 싶을 때 (마이크로 API, 웹훅, 예측 결과 가져오는 간단한 API)

### API Gateway
- *제대로 된 API 게이트웨이 기능 필요할 때(인증, 스로틀링, 여러 리소스 관리)*
- 완전한 API 관리 서비스: 여러 Lambda/백엔드를 하나의 API로 묶어서 제공
- 기능: 요청 검증, API키 & 사용량 플랜, 캐싱, 요청/응답 변환, 커스텀 도메인, WAF 연동, Cognito/IAM/Lambda Authorizer 인증
- Rest API vs HTTP API 두 종류
    - REST API: 기능이 많음(캐싱, 요청 검증 등), 비용/지연 더 큼
    - HTTP API: 기능 단순, 더 빠름, 더 저렴
- 적합한 경우: 여러 엔드포인트/리소스를 묶은 정식 API, 세밀한 트래픽 제어, 다수 클라이언트에게 API 키 발급해야 할 때

### 동시성(Concurrency) 관리
- Reserved Concurrency: 특정 함수가 쓸 수 있는 최대 동시 실행 수 제한/보장
- Provisioned Concurrency: 미리 실행 환경을 웜업 상태로 유지 → 콜드 스타트 제거

---

### Hive
빅데이터를 SQL처럼 쿼리할 수 있게 해주는 데이터 웨어하우스 소프트웨어
- Hive Matastore: 테이블 스키마 정보를 저장하는 메타데이터 저장소 → Glue DataCatalog의 원형/호환 개념
- Amazon EMR(관리형 Hadoop/Spark 클러스터)에서 Hive를 실행하는 게 일반적

### DynamoDB 사용 모드
- 프로비저닝 모드: RCU/WCU를 미리 설정하는 방식. 설정한 용량을 초과하면 스로틀링 발생
- 온디맨드 모드: 트래픽에 따라 자동 스케일, 사용한 만큼 과금

### DynamoDB용 Athena 커넥터
- Athena가 Federated Query 기능으로 DynamoDB를 직접 조회할 수 있개 해주는 커넥터

### AWS Glue로 DynamoDB → S3 내보내기
- Glue Job이 DynamoDB 테이블 PITR 데이터를 읽어서 S3로 복사
    - DynamoDB 테이블의 PITR 백업 데이터를 기반으로 내보내는 방식이라 RCU/WCU를 소비하지 않음
- Glue Job이 S3로 내보낸 이후, Glue가 ETL/변환 → 지표 계산은 이 S3 사본에서 처리 → 라이브 테이블 건드리지 않음

---

### EBS Encryption by Default
- 리전 단위 설정 하나 켜두면, 그 리전에서 생성되는 모든 새 EBS 볼륨이 자동으로 암호화됨
- 볼륨 하나하나 수동으로 함호화 설정할 필요가 없어짐. → 가장 적은 운영 오버헤드 조건에 맞음
- 기본 키로 별 수도 있지만, 기본 암호화 키를 고객 관리형 키(CMK)로 지정하면 앞으로 생성되는 모든 볼륨이 그 CMK로 자동 암호화됨

#### HIPAA 규정준수 → CMK 필수

### IAM 역할과 CMK 권한
- EBS 볼륨이 CMK로 암호화되어 있으면, 그 볼륨을 생성·연결·마운트하는 주체(여기선 EKS/노드)가 그 CMK를 쓸 권한이 있어야 함
- KMS 키 자체에도 "이 IAM 역할은 이 키를 쓸 수 있다"는 키 정책이 필요
- 즉 "암호화를 켜는 것"과 "그 암호화된 볼륨을 실제로 다룰 권한을 주는 것"은 별개의 두 단계 

---

### AWS CloudFormation
- Infrastructure as Code(IaC) 서비스: 인프라 구성을 코드(YAML/JSON 템플릿)으로 정의해서 반복 가능하게 배포
- 템플릿 하나로 EC2, ASG, NLB, Aurora 등 여러 리소스를 하나의 스택으로 묶어서 관리
- 같은 템플릿을 여러 리전/여러 환경에서 재사용 배포 가능

### AWS Systems Manager Automation
- 운영 작업(패치, 인스턴스 재시작, 스냅샷 생성)을 runbook 형태로 자동화하는 도구
- 처음부터 인프라 전체를 프로비저닝하는 IaC 도구가 아니라 이미 존재하는 리소스에 대한 운영 작업을 자동화하는 도구임

### AWS Config
- 리소스의 구성 변경 이력을 추적하고 규정 준수 규칙에 어긋하는지 평가하는 서비스
- Remediation: 규칙 위반이 감지되면 자동으로 수정 조치를 실행하는 기능
- 새 인프라를 처음부터 프로비저닝하는 IaC가 아니라 이미 있는 리소스가 규칙을 지키는지 감시하고 고치는 도구

### AWS Elastic Beanstalk 
- 애플리케이션 코드를 올리면 AWS가 알아서 EC2, 로드밸런서, Auto Scaling 등을 구성해주는 PaaS형 배포 플랫폼
- ALB를 씀
- 자체 방식으로 인프라를 추상화가기 때문에 프로토타입에서 이미 구성한 정확한 아키텍처를 그대로 재현하기 어려움

---

### EBS 스냅샷
- EBS 볼륨의 특정 시점 백업, S3에 저장됨
- 스냅샷을 삭제하면 바로 사라짐

### EBS 스냅샷 Recycle Bin
- 설정한 보존 기간(1일~1년)동안 휴지통에 보관하는 안전망
- 리전/계정 전체 스냅샷에 태그 기반으로 적용 가능

### Cross Region 스냅샷 복사
- 스냅샷을 다른 리전으로 복사해두는 것: 원래는 리전 재해 복구(DR) 목적(해당 리전 전체가 다운됐을 때 대비)
- 문제는 매일 새 복사본을 또 만들어야 하고, 그 복사본들도 결국 스케줄링/보존관리/모니터링을 직접 개발해야 함 → "최소 개발 노력" 조건과 안 맞음
- 실수로 삭제되는 문제 자체를 막아주지도 않음 (원본이 삭제돼도 복사본은 남지만, 그 복사본 관리 체계를 처음부터 만들어야 하는 부담)


### S3 스토리지 클래스 전체 정리
| 클래스 | 특징 | 적합한 경우 |
|---|---|---|
| S3 Standard | 높은 가용성/내구성, 즉시 접근 | 자주 접근하는 데이터 |
| S3 Intelligent-Tiering | 접근 패턴 자동 분석해서 최적 티어로 자동 이동 | 접근 패턴이 불규칙/예측 불가할 때 |
| S3 Standard-IA (Infrequent Access) | 저장 비용↓, 검색(retrieval) 비용 있음, 즉시 접근 가능 | 가끔 접근하지만 빠른 접근은 필요한 데이터 |
| S3 One Zone-IA | Standard-IA와 비슷하지만 단일 AZ에만 저장 (더 저렴, 내구성↓) | 재생성 가능한 데이터, 백업의 백업 |
| S3 Glacier Instant Retrieval | 아카이브지만 밀리초 단위 즉시 조회 가능 | 분기별 1회 정도 접근하는 아카이브 |
| S3 Glacier Flexible Retrieval | 조회에 몇 분~몇 시간 소요 | 연 1~2회 접근하는 아카이브 |
| S3 Glacier Deep Archive | 가장 저렴, 복원에 12시간 이상 | 규제 준수용 장기 보관(7~10년), 거의 접근 안 함 |


</details>

<br>
<br>

<details>
<summary>오답 노트</summary>

### 문제 1
한 글로벌 게임 회사가 AWS에서 멀티플레이어 온라인 게임 플랫폼을
운영합니다. 이 플랫폼은 여러 시간대의 게임 피크 시간에 수백만 명의
동시 접속 플레이어를 처리합니다. 회사는 점수, 업적, 아이템 구매와 같은
수백만 건의 플레이어 활동 이벤트를 여러 내부 분석 애플리케이션과
공유하기 위한 확장 가능한 준실시간 솔루션이 필요합니다. 플레이어
이벤트는 리더보드 서비스가 빠르게 조회할 수 있도록 NoSQL
데이터베이스에 저장되기 전에 개인 식별 정보(PII)를 마스킹하도록 처리되어야
합니다. 이러한 요구 사항을 충족하려면 솔루션 아키텍트는 무엇을 권장해야
합니까?
 
<br>

A. 플레이어 이벤트 데이터를 Amazon DynamoDB에 저장한다. DynamoDB에
   쓰기 시 각 이벤트에서 PII를 제거하는 규칙을 설정한다. DynamoDB
   Streams를 사용하여 플레이어 이벤트 데이터를 다른 애플리케이션과
   공유한다.
 
B. 플레이어 이벤트 데이터를 Amazon Kinesis Data Firehose로 스트리밍하여
   Amazon DynamoDB와 Amazon S3에 저장한다. Kinesis Data Firehose의
   AWS Lambda 통합을 사용하여 PII를 제거한다. 다른 애플리케이션은
   Amazon S3에 저장된 데이터를 소비할 수 있다.
 
C. 플레이어 이벤트 데이터를 Amazon Kinesis Data Streams로 스트리밍한다.
   AWS Lambda 통합을 사용하여 각 이벤트에서 PII를 제거한 다음 플레이어
   이벤트 데이터를 Amazon DynamoDB에 저장한다. 다른 애플리케이션은
   Kinesis 데이터 스트림에서 플레이어 이벤트 데이터를 소비할 수 있다.
 
D. 배치된 플레이어 이벤트 데이터를 파일로 Amazon S3에 저장한다. AWS
   Lambda를 사용해 각 파일을 처리하고 PII를 제거한 뒤 Amazon S3의 파일을
   업데이트한다. 그런 다음 Lambda 함수가 데이터를 Amazon DynamoDB에
   저장한다. 다른 애플리케이션은 Amazon S3에 저장된 이벤트 파일을 소비할
   수 있다.



### 문제 2
한 의료 기관이 ① 환자 등록, 진료 예약, 의료 기록, 청구 시스템을 포함하는
여러 상호 연결된 서비스로 구성된 환자 관리 시스템을 운영하고 있습니다.
현재 온프레미스 아키텍처는 환자 등록량이 300% 증가하는 ② 피크 시간
(오전 8시-10시 및 오후 2시-4시)에 잦은 서비스 장애가 발생합니다. 시스템은
서비스 간 통신에 REST API를 사용하지만, ③ 개별 서비스가 과부하될 때
트랜잭션이 자주 손실됩니다. 조직은 AWS Cloud로 마이그레이션하여
④ 15,000건의 동시 환자 등록 피크 부하를 처리하면서 ⑤ 트랜잭션 손실을
0으로 보장하고 시스템 응답성을 유지해야 합니다. 솔루션은 ⑥ 자동 확장
기능을 제공하고 서비스를 분리하여 연쇄 장애를 방지해야 합니다. 어떤
솔루션이 이러한 요구 사항을 충족하며 운영 효율성이 가장 높습니까?


<br>

A. Amazon API Gateway를 사용하여 환자 요청을 각 서비스 계층의 AWS
   Lambda 함수로 라우팅한다. 서비스 간 비동기 통신을 처리하기 위해
   Amazon Simple Queue Service(Amazon SQS)를 구현한다.
 
B. 과거 CloudWatch 지표를 분석하여 서비스 장애 중 피크 사용 패턴을
   식별한다. 각 서비스 계층의 Amazon EC2 인스턴스를 15,000개의 동시
   요청이라는 최대 예상 부하를 처리할 수 있도록 scale up한다.
 
C. Auto Scaling group의 Amazon EC2 인스턴스에 서비스를 배포하고 서비스
   간 메시징에 Amazon Simple Notification Service(Amazon SNS)를 사용한다.
   CloudWatch가 SNS 메시지 볼륨을 모니터링하고 스케일링 작업을 트리거하도록
   구성한다.
 
D. Auto Scaling group 내 Amazon EC2 인스턴스에서 서비스를 실행하고 서비스
   간 통신에 Amazon Simple Queue Service(Amazon SQS)를 구현한다.
   CloudWatch를 사용해 SQS queue depth를 모니터링하고 통신 병목이
   발생하면 자동으로 확장한다.
 

### 문제 3
패션 소매에 특화된 전자상거래 플랫폼이 온라인 마켓플레이스의 상품 이미지
최적화를 제공하려고 합니다. 이 플랫폼은 수천 개의 의류와 액세서리를
호스팅하며, 판매자가 업로드하는 고해상도 상품 사진은 데스크톱 브라우저,
모바일 앱, 태블릿 인터페이스에 표시되어야 합니다. 시스템은 업로드된 상품
이미지에서 ① 여러 이미지 형식(② thumbnail: 150x150px, mobile: 400x400px,
desktop: 800x800px)을 자동으로 생성해야 합니다. 이 플랫폼은 Black Friday
같은 판매 이벤트 동안 ③ 최대 300%의 계절적 트래픽 급증을 겪습니다.
솔루션은 ④ 피크 시간에 높은 성능을 유지하면서 ⑤ 트래픽이 적은 기간에는
비용 효율적이어야 합니다. 이러한 요구 사항을 충족하려면 솔루션스
아키텍트는 무엇을 권장해야 합니까?
 

 <br>


A. Amazon S3 정적 호스팅에 웹 애플리케이션을 배포하고, 이미지 처리를 위해
   AWS Lambda 함수를 트리거하며, 최적화된 이미지를 크기별로 구성된 별도
   S3 버킷에 저장한다.
 
B. Amazon CloudFront 배포를 사용하는 웹 애플리케이션을 설정하고, AWS Step
   Functions 워크플로를 호출하여 이미지를 처리하며, 결과를 이미지
   메타데이터와 함께 Amazon DynamoDB에 저장한다.
 
C. 전용 Amazon EC2 인스턴스에 웹 애플리케이션을 구축하고, 사용자 지정
   이미지 처리 서비스가 이미지를 동기적으로 리사이즈한 뒤 Amazon S3
   스토리지에 저장하도록 한다.
 
D. Auto Scaling이 적용된 Amazon ECS의 컨테이너화된 웹 애플리케이션을
   구현하고, 이미지 처리 작업을 Amazon SQS에 큐잉하며, 전용 EC2 워커
   인스턴스로 처리 작업을 수행한다.
 
 

 
### 문제 4
한 금융 서비스 회사는 모바일 뱅킹 애플리케이션에서 대량의 ① 트랜잭션 로그
데이터를 수집하여 Amazon S3에 저장합니다. 데이터에는 ② 일일 활성 사용자
200만 명이 생성하는 사용자 상호작용, API 호출, 시스템 지표가 포함됩니다.
이 회사는 포괄적인 사기 탐지 파이프라인에 데이터를 넣기 전에, ③ S3에
저장된 트랜잭션 로그 데이터를 빠르게 분석하여 ④ 의심스러운 패턴과 사기
징후를 식별해야 합니다. 솔루션은 관리 복잡성과 운영 작업을 최소화해야
합니다. ⑤ 최소 운영 오버헤드로 이러한 요구 사항을 충족하는 솔루션은
무엇입니까?
 

<br>

A. Spark 카탈로그에 외부 테이블을 생성합니다. 트랜잭션 로그 데이터를
   쿼리하도록 AWS Glue 작업을 구성합니다.
 
B. AWS Glue 크롤러가 트랜잭션 로그 데이터를 크롤링하도록 구성합니다.
   Amazon Athena가 데이터를 쿼리하도록 구성합니다.
 
C. Hive metastore에 외부 테이블을 생성합니다. 트랜잭션 데이터를 쿼리하도록
   Amazon EMR의 Spark 작업을 구성합니다.
 
D. AWS Glue 크롤러가 트랜잭션 데이터를 크롤링하도록 구성합니다. Amazon
   Kinesis Data Analytics를 사용하여 SQL을 사용해 데이터를 쿼리하도록
   구성합니다.
 
 
 

### 문제 5
한 온라인 서비스가 ① AWS WAF를 통합하면서 가용성을 높여야 합니다.
② 여러 AZ에 걸쳐 로드 밸런서 뒤에서 실행하고 ③ WAF를 적절하게 연결해야
합니다. 어떤 솔루션이 이러한 요구 사항을 충족합니까?
 
<br>

A. 두 AZ에 걸친 ASG를 ALB 뒤에 배치하고 AWS WAF를 ALB에 연결합니다.
 
B. 클러스터 배치 그룹을 사용하고 ALB가 인스턴스를 대상으로 하며 WAF를
   배치 그룹에 연결합니다.
 
C. 두 AZ에 걸친 두 개의 EC2 인스턴스를 ALB 뒤에 배치하고 WAF를 ALB에
   연결합니다.
 
D. 두 AZ에 걸친 ASG를 ALB 뒤에 배치하고 AWS WAF를 ASG에 연결합니다.








</details>