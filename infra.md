# Infrastructure

## 1. 유일한 개발 환경: Docker

- Docker를 유일한 종속성으로 설정
    - 특정 버전 혹은 OS에 대한 요구사항 제거를 위해 컨테이너화된 개발자 워크스페이스 구축
- 개발 데이터베이스는 마이크로 서비스별로 RDS의 스키마를 나눠 설정
    - e.g., dev-stores, dev-items, …

## 2. Public Cloud: Amazon Web Service

![Salesync (3).png](https://prod-files-secure.s3.us-west-2.amazonaws.com/2b0c281b-9bce-4608-bd24-7d0b6fc0365d/3a972ee8-2108-4301-a723-a8e744e703a8/Salesync_(3).png)

### 2.1. Salesync 인프라 아키텍처 표준화 과정

### AS-IS

1. **현재 아키텍처 구성 요소에 대한 문서화**
- 현재 아키텍처에 대한 모든 구성 요소와 서비스의 상세한 문서화를 확인하거나 작성
- 각 컴포넌트의 설정, 연결 방식

[세부 구성 요소 및 설정](https://www.notion.so/9a0cd616806c470089710f58ae273fdb?pvs=21)

1. **베스트 프랙티스와 프레임워크 정의**
    - AWS Well-Architected Framework를 참조하여 표준화할 아키텍처의 베스트 프랙티스와 원칙을 정의

[AWS Well-Architected 설계 원칙](https://www.notion.so/AWS-Well-Architected-a492673705bb4e78ae7a6a558fc7fe29?pvs=21)

1. **갭 분석 (Gap Analysis)**
    - 현재 아키텍처와 표준화된 아키텍처 원칙 사이의 차이점을 식별
    - 각각의 아키텍처 요소를 베스트 프랙티스와 비교하여 미흡한 점이나 개선이 필요한 부분을 찾아냄

[갭 분석 (Gap Analysis)](https://www.notion.so/Gap-Analysis-d3737bca1c6f41d6b1b691fb1d31957e?pvs=21)

1. **표준화 로드맵 수립**
    - 각각의 갭을 해결하기 위한 구체적인 조치 사항을 계획하고 우선순위를 정하기
    - 단기적, 중기적, 장기적 행동 계획을 포함하는 로드맵을 수립

[표준화 로드맵 수립](https://www.notion.so/8cd011d3d4c14c32b31c34a21020ca8f?pvs=21)

---

### TO-BE

1. **구현 및 실행**
    - 표준화 로드맵에 따라 필요한 구성 변경, 인프라 업데이트, 코드 수정을 진행
    - 이 과정에서 인프라 코드화 및 자동화 도구(Terraform, CloudFormation 등)를 사용하여 변경 사항을 적용
    
2. **검증 및 테스팅**
    - 변경 사항을 적용한 후, 각각의 컴포넌트가 예상대로 작동하는지 철저히 검증
    - 보안, 성능, 비용 관점에서도 테스팅을 수행
    
3. **문서화 및 지식 공유**
    - 변경된 아키텍처와 관련한 모든 문서를 업데이트
    - 팀원과 관련된 이해 관계자에게 교육 및 지식 공유 세션을 제공
    
4. **지속적인 모니터링 및 개선**
    - 새로운 표준화된 아키텍처의 성능을 지속적으로 모니터링
    - 새로운 요구 사항이나 기술 발전에 따라 아키텍처를 지속적으로 개선하고 업데이트

### 2.2. Salesync 인프라 설계 전략

1. 데이터 백업, 복구 전략
2. 무정지 운영 (Zero Downtime Operations) 전략
3. DR (Disaster Recovery) 전략
4. IAM 관리 전략

[Salesync 인프라 전략 상세 보기](https://www.notion.so/Salesync-3faedceb628e4360bb759374ee17d715?pvs=21)

### 2.3. 코드형 인프라 (Infrastructure as a Code)

- Terraform을 사용해 인프라의 재현성, 확장성 증대
- Terraform 코드를 모듈화해서 작성

 

## 3. 컨테이너 오케스트레이션: Kubernetes

![Screenshot 2024-01-22 at 10.46.21.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/2b0c281b-9bce-4608-bd24-7d0b6fc0365d/4a187be4-be49-42ab-81fe-67ac542f039b/Screenshot_2024-01-22_at_10.46.21.png)

- 컨테이너 환경의 예측 가능하고 격리된 시스템 구축
- 컨테이너화된 애플리케이션의 오케스트레이션을 목적으로 AWS EKS 기반 클러스터 구축
    - 컨테이너의 배포, 확장, 관찰, 관리
    - 배포의 롤아웃, 롤백
    - 시크릿 관리, 로드 밸런싱, 트래픽 관리 등

### 3.1. Cluster Autoscaling (Scaling Out)

- Kubernetes의 Cluster Autoscaler 모듈 사용해 Autoscaling Group의 Desired capacity 변경
- 장점
    - 자동 스케일링
- 단점
    - 새 인스턴스가 생성되는 시간이 느림 → 급격한 트래픽에 즉각적으로 반응하지는 않음
    - 내부 네트워크 트래픽 증가

### 3.2. Scaling Up

- 로그 파이프라인 애플리케이션이 높은 메모리 사용량을 필요로 해 수직 확장이 필요해짐
- 수동으로 노드의 스펙 변경
    - e.g., t3.medium → t3.large
- 장점
    - 계산해보니 더 큰 노드를 운영하는 것이 비용 효율적일 때가 있음
    - 관리 복잡성 감소
- 단점
    - 수동 스케일링

### 3.3. Ingress

- 외부에서 BFF, 앞단 API로 들어오는 트래픽을 라우팅
- API Gateway의 관리형 기능은 필요하지 않아 Ingress 사용
- 인그레스를 관리하는 인그레스 컨트롤러 구현체는 ALB Controller 사용
