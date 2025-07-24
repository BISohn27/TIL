# AWS Structure

## Virtual Private Cloud란?

VPC는 AWS 사용자에게 제공되는 리전 내 격리된 네트워크 공간으로, 물리적인 서버인 EC2 인스턴스는 VPC 네트워크 공간에 할당된다. VPC는 해당 사용자에게만 할당된 네트워크 공간이기 때문에 **RFC 1918에서 정의된 사설 IP 대역**에 걸쳐 인스턴스를 할당 할 수 있고, 서브넷이라는 작은 네트워크 범위로 해당 대역을 나눠 인스턴스 IP를 할당한다.

**RFC 1918에서 정의된 사설 IP 대역**

- 10.0.0.0/8
- 172.16.0.0/12
- 192.168.0.0/16

## VPC는 public subnet과 private subnet으로 이뤄져 있다

![](https://docs.aws.amazon.com/images/vpc/latest/userguide/images/subnet-diagram.png)

- VPC는 **퍼블릭 서브넷**과 **프라이빗 서브넷**로 나뉨
- 서브넷은 **가용 영역** 당 배정
- 둘의 가장 큰 차이는 네트워크 영역이 퍼블릭(인터넷)과 연결 되는지 여부
- 퍼블릭 서브넷의 경우, 인터넷과 연결 가능하기 때문에 공인 IP가 할당
- 퍼블릭 서브넷에 위치한 인스턴스의 경우, 인스턴스를 시작할 때마다 변하는 가변 IP를 공인 IP로 할당 받기 때문에 고정 IP를 할당 받고 싶을 때에는 **Elastic IP**를 받아야 함
- 프라이빗 서브넷의 경우 외부 인터넷과 연결되어 있지 않기 때문에 기본적으로 외부(인터넷)에서 들어오는 트래픽을 직접 받을 수는 없음

## subnet은 CIDR로 대역을 나눈다

- CIDR(Classless Inter-Domain Routing)은 IP 대역을 나누기 위한 방법
- CIDR은 `x.x.x.x/숫자`로 표현

  - 뒤의 숫자는 IPv4에서 앞쪽 고정된 비트 개수를 표현
  - `10.0.0.0/8`의 경우 앞의 8개 비트는 고정되어 있기 때문에 해당 서브넷에 포함되는 IP 대역은 `2^(32-8) = 2^24`개 (정확하게는 2^24 - 2(식별용 첫번째 IP와 브로드캐스팅용 마지막 IP 제외))

    > - `10.0.0.0 => 00001010.00000000.00000000.00000000`
    > - 앞쪽 `0001010` 고정
    > - 범위 `00001010.00000000.00000000.00000000` ~ `00001010.11111111.11111111.11111111`
    > - `00001010.00000000.00000000.00000000`: 서브넷 식별용으로 제외
    > - `00001010.11111111.11111111.11111111`: 서브넷 전체 IP에 브로드캐스팅 용도로 제외

## public subnet에서 외부와 트래픽을 교환하기 위해서 IGW를 사용한다

![](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/images/internet-gateway-basics.png)

- VPC에서 외부(인터넷)과의 모든 트래픽 교환은 IGW를 통해 이뤄짐
- IGW로 들어온 외부 트래픽은 **라우팅 테이블**이라는 공인IP-사설IP 매핑 테이블을 통해 트래픽을 실제 목적지로 전달 됨
- 프라이빗 서브넷은 **라우팅 테이블**에 등록되어 있지 않기 때문에 외부와의 통신이 불가능 함

## private subnet은 NAT gateway를 통해 트래픽을 외부에 전달 가능하다

![](https://docs.aws.amazon.com/ko_kr/vpc/latest/userguide/images/public-nat-gateway-diagram.png)

- 프라이빗 서브넷은 외부로부터 트래픽을 직접 받는 것은 불가능
- 프라이빗 서브넷은 외부로 트래픽을 보내기 위해 NAT(Network Address Translation) gateway를 거쳐야 하며, NAT gateway를 거치면서 프라이빗 서브넷 인스턴스의 사설 IP가 NAT gateway의 공인 IP로 변경
- 프라이빗 서브넷은 NAT gateway를 통해서만 외부에 요청을 보내고 응답을 받음

## private subnet으로 외부 트래픽을 받기 위해서는 퍼블릭 서브넷의 ELB(Elastic Load Balancer)를 사용해야 한다

![](https://docs.aws.amazon.com/ko_kr/whitepapers/latest/web-application-hosting-best-practices/images/image4.png)

- 기본적으로 프라이빗 서브넷은 외부의 트래픽을 받지 못함
- 프라이빗 서브넷이 외부의 트래픽을 받기 위해서는 로드밸런서(ELB)를 퍼블릭 서브넷에 두고 외부 트래픽을 받을 수 있음
- 이 구조는 인스턴스 자체는 프라이빗 영역에 둬서 직접적인 트래픽을 않도록 하여 보안을 유지하고, 외부 트래픽은 로드밸런서를 통해 받음
