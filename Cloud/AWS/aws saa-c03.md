

### Athena
- 서버리스
- 비용 효율성: RedShift보다 가성비 있음
- S3에 저장된 파일(CSV, JSON, Parquet)을 그 자리에서 바로 SQL로 분석할 수 있게 함

### AWS Glue
1. Glue Crawler: S3에 있는 파일을 보고 스키마를 파악함
2. Data Catalog: 크롤러가 파악한 스키마를 구조 메타데이터를 저장
3. Athena: Data Catalog를 보고 SQL 쿼리를 실행 가능

