# AWS

## 클라우드 서비스

### 클라우드 서비스 모델

- Iaas(Intrastructure as a Service)
  - 인프라(CPU, 메모리, 네트워크 등 서버 자체)를 빌려주는 형태
  - 예) AWS EC2
- Paas(Platform as a Service)
  - 개발자가 서버, 스토리지, 네트워크 등 하드웨어 자원에 대해 걱정하지 않고, 애플리케이션 코드를 배포
  - 실행 환경, 데이터베이스, 웹서버 등의 기술적인 측면은 클라우드 서비스에서 제공
  - 예) AWS Elastic Beanstalk, RDS
- Saas(Software as a Service)
  - 소프트웨어를 서비스 형태로 제공 받는 형태
  - 예) Gmail, Dropbox

### 클라우드 배포 모델

- 퍼블릭 클라우드: AWS, Azure, GCP와 같은 공개 서비스
- 프라이빗 클라우드: 기업 내부 클라우드
- 하이브리드 클라우드: 프라이빗 클라우드(사내 데이터 센터)와 퍼블릭 클라우드(외부 클라우드 서비스 제공 업체가 제공하는 클라우드)를 결합

## 리전 및 영역

AWS VM(EC2)는 전 세계의 여러 곳에서 호스팅되고 있으며, AWS 리전, 가용 영역, Local Zones, AWS Outposts, Wavelength Zone으로 구성

### 리전

- 별도의 지리적 영역
- 각 리전은 다른 리전에서 격리되도록 설계 -> 강력한 내결함성 및 안정성 확보
- 인스턴스를 특정 고객과 가까운 위치에 배치 또는 법률 이나 기타 요구 사항을 충족하는 리전을 선택
- 지정한 리전에 연결된 리소스만 표시, 리전이 격리되어 있기 때문에 리소스의 자동 복제도 일어나지 않음

### 가용 영역

![alt text](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/images/region-with-azs.png)

- 각 리전은 **_가용 영역_**이라는 격리된 여러 개의 위치로 나뉨
- 가용 영역 코드는 리전 코드와 문자 식별자를 조합 (`us-east-1a`)
- 여러 가용 영역에서 EC2 인스턴스를 시작하여 단일 위치에 의한 장애 발생으로부터 애플리케이션을 보호
- 서비넷이 없는 가용 영역에서는 인스턴스를 시작할 수 없음 (**EC2 → Subnet → Availability Zone**)
- 초기 인스턴스를 시작 → 시스템 상태 및 가용 용량에 따라 가용 영역이 선택 자동으로 선택 되는 것이 좋음
- 추가 인스턴스 시작 → 기존 인스턴스와 가까이 있거나 분리되어 있어야 하는 경우에 가용 영역 지정

### 로컬 영역

![alt text](https://docs.aws.amazon.com/ko_kr/AWSEC2/latest/UserGuide/images/region-with-lzs.png)

- 리전과 가까운 위치에 있는 작은 데이터 센터, 리전과는 고속 네트워크로 연결
- **Run low-latency applications at the edge**
  - 애플리케이션을 최종 사용자와 가까운 위치에 배포
  - 실시간 게이밍이나 라이브 스트리밍, VR/AR 등을 제공
- **Simplify hybrid cloud migrations**
  - 하이브리드 클라우드를 구축할 때 기존 온프레미스 또는 기업 내부 데이터 센터와 가까운 곳에 구성
  - 지연 시간이 짧은 하이브리드 배포 모델 설계 가능
- **Meet stringent data residency requirements**
  - 데이터가 특정 지역에 저장되어야 하는 등의 법적, 정책적 요구사항을 충족

### Wavelength Zone

- 5G 네트워크를 통해 초저지연 컴퓨팅 설계
- 5G 통신 제공 기지국 인프라 내에 컴퓨팅 자원을 직접 배치 (통신사 데이터센터 등)

### AWS Outposts

- 고객사의 데이터센터(온프렘 환경)에 AWS 인터페이스를 설치하여 운용

## AWS 서비스 구성

- 네트워크 (VPC)
  ![alt text](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/images/how-it-works.png)

  - VPC (Virtual Private Cloud)
    - AWS 내의 격리된 네트워크 공간
    - 논리적으로 격리된 가상 네트워크를 통해 AWS 리소스 사용
    - AWS의 확장 가능한 인프라를 사용하면서 기업 내부 데이터 센터에서 운영하는 기존 네트워크와 유사하게 구성
    - VPC 구성
      - VPC 리전의 **각 가용 영역**에는 **하나의 서브넷** 존재
      - 각 서브넷에 **EC2 인터페이스** 존재
      - VPC의 리소스와 인터넷 간 통신을 허용하는 **인터넷 게이트웨이** 존재
    - VPC를 생성 후 서브넷을 추가
  - 서브넷 (Subnet)
    - VPC의 IP 주소 범위
    - 한 개의 가용 영역에 할당
    - 서브넷을 추가한 후 VPC에 AWS 리소스 배포 가능
  - 인터넷 게이트웨이

    - VPC와 인터넷 간 통신할 수 있게 연결하는 역할
    - IPv4 트래픽 및 IPv6 트래픽 지원
    - 리소스에 퍼블릭 IP 주소가 존재할 때, 인터넷 게이트웨이를 사용하여 퍼블릭 서브넷(EC2 인스턴스)의 리소스가 인터넷에 연결
    - 인터넷의 리소스는 퍼블릭 IP를 사용하여 서브넷의 리소스에 연결
    - IPv4 통신에서 Network Address Translation(NAT) 수행
      - 인스턴스는 VPC 및 서브넷 내부 정의된 프라이빗 IP 주소 공간만 인식
      - 인터넷 게이트웨이를 통해 트래픽이 인터넷으로 나갈 경우, 회신 주소를 인스턴스의 퍼블릭 IPv4 주소 또는 Elastic IP로 설정
      - 퍼블릭 IP로 트래픽이 들어올 경우 인스턴스의 프라이빗 IP 주소로 변환

  - 컴퓨팅

    - EC2 (Elastic Compute Cloud)

      - AWS VM 실행 서비스
      - AWS 클라우드의 온디멘드, 확장 가능한 컴퓨팅을 제공
      - 보안과 네트워크를 구성하고, 스토리지를 관리
      - 인스턴스 타입 선택을 통해 하드웨어 사양을 결정
        ![alt text](https://docs.aws.amazon.com/images/AWSEC2/latest/UserGuide/images/instance-types.png)

      - 구성 요소
        - Amazone Machine Image(AMIs)
          - 사전에 정의된 인스턴스 템플릿
          - 운영체제 및 추가 소프트웨어 등의 컴포넌트를 패키징
        - Instance Type
          - CPU, 메모리, 스토리지, 네트워크, 그래픽 등의 인스턴스 하드웨어 타입
        - Key pairs
          - 인스턴스에 보안 로그인을 위한 키
          - AWS는 공개 키를 보관하고, 사용자는 비밀키를 저장
        - Security groups
          - 가상 방화벽
          - 프로토콜, 포트, 인바운드 IP 대역, 아웃바운드 IP 대역

  - 스토리지
    - S3 (Simple Storage Service)
      - 객체 스토리지 서비스
      - 확장성, 가용성, 보안성, 성능 등의 특성을 가진 스토리지에 다양한 형태의 데이터를 저장
      - 구조
        - `S3-Bucket (최상위 컨테이너)-Object (파일 + 메타데이터 객체)-Key (경로와 비슷한 고유 이름)`
    - EBS (Elastic Block Store)
      - EC2에 연결되는 디스크
      - 블록 형태로 제공
      - 인스턴스를 중지하더라도 EBS는 인스턴스를 삭제할 때까지 요금을 지불
