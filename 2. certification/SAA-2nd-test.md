# SAA-2nd-test 오답 정리

## 1. Amazon SQS
- 메시지 보존 기간을 1분~14일 사이의 값으로 구성할 수 있으며 기본 값은 4일이다.
- 메시지 보존 제한에 도달하면 메시지는 자동으로 삭제된다.
- 단일 Amazon SQS 메시지 대기열에는 무제한의 메시지가 포함될 수있다.
- 표준 큐의 이동 중 메시지 수는 120,000개, FIFO 큐의 경우 20,000개의 제한이 있다.

1). 표준 대기열
 - 무제한 처리량 : 표준 대기열은 API 작업당 거의 무제한의 초당 트랜잭션(TPS)을 지원
 - 최소한 한 번 전달 : 메시지가 최소한 한 번 전달되고, 가끔 2개 이상의 메시지 복사본이 전달될 수 있다.
 - 최선 노력 순서 : 가끔 메시지가 전송된 순서와 다르게 전달될 수 있다.
 - 한 번 이상 잘못된 순서로 도착하는 메시지를 애플리케이션에서 다음과 같이 처리할 수 있다면 여러 시나리오에서 표준 메시지 대기열을 사용할 수 있다.
 - 실시간 사용자 요청을 집중적 백그라운드 작업과 분리: 사용자들이 미디어 크기를 조정하거나 인코딩하면서 미디어를 업로드할 수 있게 합니다.
 - 작업을 여러 작업자 노드에 할당: 많은 수의 신용 카드 인증 요청을 처리합니다.
 - 장래 처리를 위한 메시지 배치: 데이터베이스에 여러 항목이 추가되도록 예약합니다.

2). FIFO 대기열
 - 높은 처리량 : 기본적으로 FIFO 대기열은 초당 최대 300개의 메시지(초당 300개의 전송, 수신 또는 삭제 작업)을 지원한다. 작업당 최대 10개 메시지를 일괄 처리할 경우, FIFO 대기열은 초당 3000개의 메시지까지 지원할 수 있다. 한도 증가 신청가능.
 - 정확히 한 번 처리 : 메시지가 한 번 전달되고 소비자가 이를 처리 및 삭제할 때까지 유지된다. 중복 메시지는 대기열에 올라가지 않는다.
 - 선입선출 전달 : 메시지가 전송되고 수신되는 순서가 엄격하게 지켜진다.(예: 선입선출)
 - FIFO 대기열은 다음 예와 같이 작업 및 이벤트 순서가 중요하거나 중복 항목이 허용되지 않는 경우에 애플리케이션 간 메시징을 강화해 줍니다.
 - 사용자가 입력한 명령이 올바른 순서로 실행되도록 해야 하는 경우.
 - 가격 수정을 올바른 순서로 전송하여 제품 가격을 정확히 표시해야 하는 경우.
 - 계정에 등록하기 전에 학생이 과정에 등록되는 것을 방지해야 하는 경우.

</br></br>

## 2. S3

- S3 정적 웹 호스팅 주소 : bucket-name.s3-website-ap-northeast-2.amazonaws.com
- S3 버킷의 유효한 엔드포인트
    - bucket-name.s3.amazonaws.com/object
    - s3.region.amazonaws.com/object

### 1. Amazon S3 데이터 일관성 모델

Amazon S3는 모든 AWS 리전의 Amazon S3 버킷에 있는 객체의 PUT 및 DELETE 요청에 대해 강력한 쓰기 후 읽기(read-after-write) 일관성을 제공한다. 이것은 새 객체에 대한 쓰기와, 기존 객체 및 DELETE 요청을 덮어쓰는 PUT 요청 모두에 적용된다. 또한 Amazon S3 Select, Amazon S3 액세스 제어 목록(ACL), Amazon S3 객체 태그, 객체 메타데이터(예: HEAD 객체)에 대한 읽기 작업은 매우 일관적이다.

단일 키에 대한 업데이트는 원자성입니다. 예를 들어, 한 스레드에서 기존 키에 PUT 요청을 수행하고 두 번째 스레드에서 동일한 키에 GET 요청을 동시에 수행하면, 이전 데이터 또는 새 데이터는 얻지만 부분적 데이터나 손상된 데이터는 얻지 못합니다.

Amazon S3에서는 AWS 데이터 센터 내의 여러 서버로 데이터를 복제함으로써 고가용성을 구현합니다. PUT 요청이 성공하면 데이터가 안전하게 저장됩니다. 성공적인 PUT 응답을 받은 후 시작된 모든 읽기(GET 또는 LIST 요청)는 PUT 요청에 의해 쓰여진 데이터를 반환합니다. 다음은 이 동작의 예입니다.

- 프로세스가 Amazon S3로 새 객체를 쓰고 해당 버킷 내에 바로 키를 나열합니다. 새 객체가 목록에 나타납니다.
- 프로세스가 기존 객체를 대체하고 바로 읽기를 시도합니다. Amazon S3가 새 데이터를 반환합니다.
- 프로세스가 기존 객체를 삭제하고 바로 읽기를 시도합니다. 객체가 삭제되었으므로 Amazon S3는 데이터를 반환하지 않습니다.
- 프로세스가 기존 객체를 삭제하고 해당 버킷 내에 바로 키를 나열합니다. 객체가 목록에 나타나지 않습니다.
- Amazon S3는 동시 작성자에 대한 객체 잠금을 지원하지 않습니다. 두 PUT 요청을 동시에 같은 키에 대해 실행할 경우 타임스탬프가 최신인 요청이 우선 적용됩니다. 이것이 문제가 되는 경우 객체 잠금 메커니즘을 애플리케이션에 구축해야 합니다.
- 업데이트는 키를 기반으로 하므로 여러 키에서 원자성 업데이트를 수행할 방법은 없습니다. 예를 들어, 한 키의 업데이트에 의존하는 다른 키의 업데이트를 수행할 수 없습니다. 단, 이 기능을 애플리케이션에 설계해 넣는 경우는 예외입니다.

### 2. S3 Glacier Select

S3 Glacier Select로 간단한 구조화 쿼리 언어(SQL)문을 S3 Glacier의 데이터에서 직접 사용하여 필터링 작업을 수행할 수 있다. S3 Glacier 아카이브 객체에 대해 SQL 쿼리를 제공하면 S3 Glacier Select가 쿼리를 실행하고 출력 결과를 Amazon S3에 작성합니다. S3 Glacier Select를 사용하면 Amazon S3 같이 더 자주 사용하는 계층으로 데이터를 복원할 필요 없이 S3 Glacier에 저장된 데이터에서 쿼리 및 사용자 지정 분석을 실행할 수 있습니다.
-> Amazon Glacier Select 옵션은 아카이브 검색 옵션이 아니며 주로 Glacier의 데이터 아카이브에서 직접 SQL문을 사용하여 필터링 작업을 수행하는데 사용된다.

작업을 시작할 때 다음 중 한 가지를 지정하여 액세스 시간과 비용 요건을 기준으로 아카이브를 가져올 수 있습니다.

- 신속 — 아카이브의 하위 집합에 대한 긴급 요청이 간헐적으로 필요한 경우 신속 가져오기를 통해 빠르게 데이터에 액세스할 수 있습니다. 매우 큰 아카이브(250MB+)를 제외한 모든 경우, 신속 가져오기를 사용하여 액세스된 데이터는 일반적으로 1~5분 안에 사용할 수 있게 됩니다. 프로비저닝된 용량을 통해 필요할 때 신속 검색에 대한 검색 용량이 보장됩니다.
- 표준 — 표준 가져오기를 사용하면 몇 시간 내에 아카이브에 액세스할 수 있습니다. 표준 검색은 보통 3~5시간 안에 완료됩니다. 검색 요청 시 검색 옵션을 지정하지 않을 경우 기본 옵션이 됩니다.
- 벌크 — 벌크 가져오기는 S3 Glacier에서 가장 저렴한 가져오기 옵션으로서 심지어 페타바이트 규모의 대용량 데이터까지도 1일 동안 가져올 때 사용할 수 있습니다. 벌크 검색은 보통 5~12시간 안에 완료됩니다.

프로비저닝된 용량을 통해 필요할 때 신속 검색에 대한 검색 용량이 보장됩니다. 각 용량 단위로 5분마다 신속 검색 3회를 수행할 수 있고, 최대 150MB/s의 검색 처리량이 제공됩니다.

워크로드에 몇 분 내로 데이터의 하위 집합에 대한 매우 안전하고 예측 가능한 액세스가 필요한 경우 프로비저닝된 검색 용량을 구매해야 합니다. 프로비저닝된 검색 용량이 없더라도 비정상적으로 수요가 높지 않은 경우를 제외하면 Expedited 검색은 허용됩니다. 하지만 모든 상황에서 신속 검색에 액세스해야 하는 경우 프로비저닝된 검색 용량을 구매해야 합니다.



</br></br>

## 3. AWS Site-to-Site VPN

기본적으로 Amazon VPC로 시작하는 인스턴스는 자체(원격) 네트워크와 통신할 수 없습니다. AWS Site-to-Site VPN(Site-to-Site VPN) 연결을 생성하고 연결을 통해 트래픽을 전달하도록 라우팅을 구성하여 VPC에서 원격 네트워크에 대한 액세스를 활성화할 수 있습니다.

VPN 연결이라는 용어는 일반적인 용어지만 이 설명서에서 VPN 연결은 VPC와 자체 네트워크 사이의 연결을 의미합니다. Site-to-Site VPN은 인터넷 프로토콜 보안(IPsec) VPN 연결을 지원합니다.

다음은 Site-to-Site VPN의 핵심 개념입니다.

• VPN 연결: 온프레미스 장비와 VPC 간의 보안 연결입니다.
• VPN 터널: 데이터가 고객 네트워크에서 AWS와 주고받을 수 있는 암호화된 링크입니다.
• 각 VPN 연결에는 고가용성을 위해 동시에 사용할 수 있는 두 개의 VPN 터널이 포함되어 있습니다.
• 고객 게이트웨이: 고객 게이트웨이 디바이스에 대한 정보를 AWS에 제공하는 AWS 리소스입니다.
• 고객 게이트웨이 디바이스는 Site-to-Site VPN 연결을 위해 고객 측에 설치된 물리적 디바이스 또는 소프트웨어 애플리케이션입니다.

VPN 연결을 생성하려면 AWS에서 고객 게이트웨이 장치에 대한 정보를 제공하는 고객 게이트웨이 리소스를 생성해야 합니다. 다음으로 고객 게이트웨이 외부 인터페이스의 인터넷 라우팅 가능한 고정 IP 주소를 설정해야 합니다.

VPC에는 연결된 가상 프라이빗 게이트웨이가 있으며 원격 네트워크에는 고객 게이트웨이가 포함되어 있으며 VPN을 사용하도록 구성해야 합니다. 네트워크에 바인딩 된 VPC의 모든 트래픽이 가상 프라이빗 게이트웨이로 라우팅 되도록 라우팅을 설정합니다.

</br></br>

## 4. Amazon EMR

Amazon EMR은 Apache Spark, Apache Hive, Apache HBase, Apache Flink, Apache Hudi 및 Presto와 같은 오픈 소스 도구를 사용하여 방대한 양의 데이터를 처리하기 위한 업계 최고의 클라우드 빅 데이터 플랫폼입니다. EMR을 사용하면 기존 온프레미스 솔루션의 50%도 안 되는 비용으로 표준 Apache Spark보다 3배 이상 빠르게 페타바이트 규모의 분석을 실행할 수 있습니다. 단기 실행 작업의 경우 클러스터를 가동 및 중단하고 사용된 인스턴스에 따라 초 단위로 지불하면 됩니다. 장기 실행 워크로드의 경우 수요에 맞게 자동으로 규모를 조정하는 고가용성의 클러스터를 만들 수 있습니다. Apache Spark 및 Apache Hive와 같은 오픈 소스 도구의 기존 온프레미스 배포가 있는 경우 AWS Outposts에서 EMR 클러스터를 실행할 수도 있습니다.

Amazon EMR을 통해 로그 분석을 자동으로 제공 할 수 있으며, 이는 맞춤형 로그 분석 애플리케이션을 구축하고 EC2에서 호스팅하는 것보다 경제적입니다. 따라서 ELB 로그 파일을 저장하기 위한 Amazon S3와 로그 파일을 분석하기 위한 Amazon EMR 옵션이 이 둘 사이에 가장 적합합니다.

</br></br>

## 5. Amazon EC2

사용자는 Amazon EC2 인스턴스를 모니터링하고 기본 하드웨어 장애나 복구에 AWS 개입이 필요한 문제로 인해 인스턴스가 손상된 경우 인스턴스를 자동으로 복구하는 Amazon CloudWatch 경보를 만들 수 있습니다. 종료한 인스턴스는 복구할 수 없습니다. 복구된 인스턴스는 인스턴스 ID, 프라이빗 IP 주소, 탄력적 IP 주소 및 모든 인스턴스 메타데이터를 포함하여 원본 인스턴스와 동일합니다. 손상된 인스턴스가 배치 그룹에 있다면, 복구된 인스턴스는 배치 그룹에서 실행됩니다.

### 1. Scheduled Reserved Instance

정기 예약 인스턴스를 사용하면 1년 동안 지정된 시작 시간 및 기간으로 매일, 매주 또는 매월 반복적으로 정기 용량을 예약할 수 있습니다. 구매가 완료되면 지정한 시간에 인스턴스를 시작할 수 있습니다.

</br></br>

## 6. VPC

VPC(Virtual Private Cloud)는 사용자의 AWS계정 전용 가상 네트워크다.
VPC는 AWS 클라우드에서 다른 가상 네트워크와 논리적으로 분리되어 있다. Amazon EC2 인스턴스와 같은 AWS리소스를 VPC에서 실행할 수 있다.

VPC를 만들 때, VPC의 IPv4주소의 범위를 CIDR(Classless Inter-Domain Routing)블록 형태로 지정해야한다. VPC의 기본 CIDR블록은 10.0.0.0/16 과 같다.

VPC는 리전의 모든 가용 영역에 적용된다. VPC를 만든 후, 각 가용 영역에 하나 이상의 서브넷을 추가할 수있다. 필요에 따라 컴퓨팅, 스토리지, DB 및 기타 선택 서비스를 최종 사용자에게 더 가깝게 배치하는 AWS 인프라 배포인 로컬 영역에 서브넷을 추가할 수 있다. 로컬 영역을 사용하면 최종 사용자가 한 자릿 수 밀리초 지연 시간이 필요한 애플리케이션을 실행할 수 있다.

서브넷을 만들 때 해당 서브넷에 대한 CIDR 블록을 지정한다. 이는 VPC CIDR 블록의 서브넷이다. 각 서브넷은 단일 가용 영역 내에서만 존재해야하며, 여러 영역으로 확장할 수 없다. 각 가용 영역은 다른 가용 영역에서 발생한 장애를 격리시킬 수 있도록 서로 분리된 공간이다. 별도의 가용 영역에서 인스턴스를 시작함으로써 단일 위치에서 장애가 발생할 경우 애플리케이션을 보호라 수 있다. AWS는 각 서브넷에 고유 ID를 할당한다.

- 각 서브넷은 단일 가용 영역에 매핑된다.
- 생성한 모든 서브넷은 VPC의 기본 라우팅 테이블과 자동으로 연결된다.
- 서브넷 트래픽이 인터넷 게이트웨이로 라우팅되는 경우 서브넷을 퍼블릭 서브넷이라고 한다.

</br></br>

## 7. ELB

- ELB의 SSL협상을 위한 정책 2가지
    1. predefined security policies
    2. custom security policies

### 1. Network Load Balancer

NLB는 개방형 시스템간 상호 연결(OSI) 모델의 4계층에서 작동한다. 초당 수백만 개의 요청을 처리할 수 있다. 로드밸런서가 연결 요청을 받으면 기본 규칙의 대상 그룹에서 대상을 선택한다. 리스너 구성에 지정된 포트에서 선택한 대상에 대한 TCP 연결을 열려고 시도한다.

로드 밸런서에서 가용 영역을 활성화하면 ELB가 해당 가용 영역에서 로드 밸런서 노드를 생성한다. 기본적으로 각 로드 밸런서 노드는 해당 가용 영역의 등록된 대상에만 트래픽을 분산한다. 교차 영역 로드 밸런싱을 활성화하면 각 로드 밸런서 노드가 활성화된 모든 가용 영역에 있는 등록된 대상 간에 트래픽을 분산한다.

로드 밸런서에 대해 여러 가용 영역을 활성화하고 각 대상 그룹에 각 활성화된 가용 영역에 하나 이상의 대상이 있는지 확인하면 응용 프로그램의 내결함성이 향상된다. 예를들어, 하나 이상의 대상 그룹이 가용 영역에서 정상 대상이 없는 경우 DNS에서 해당 서브넷의 IP주소를 제거하지만 다른 가용 영역의 로드 밸런서 노드는 여전히 트래픽을 라우팅할 수 있다. 클라이언트가 TTL(Time-to-live)을 인식하지 못하고 DNS에서 제거된 IP주소로 요청을 보내면 요청이 실패한다.

### 2. Application Load Balancer

ALB는 요청 수준(7계층)에서 작동하여 트래픽을 대상으로 라우팅한다. HTTP 및 HTTPS 트래픽의 고급 로드 밸런싱에 이상적인 ALB 장치는 마이크로 서비스 및 컨테이너 기반 애플리케이션을 비롯한 최신 애플리케이션 아키텍처를 제공하는데 적합한 고급 요청 라우팅을 제공한다.
ALB는 지정된 사용 사례에서 언급된 짧은 지연 시간 및 높은 처리량 시나리오에 적합하지 않다.

- ALB는 로드밸런서에서 생성된 쿠키만을 지원하며 쿠키의 이름은 AWSALB 이다.
- Cross-zon Load Balancing이 기본 활성화되어있으며, 사설 인스턴스로 LB하기 위해서는 공용서비스넷이 필요함.
- ALB는 Cognito와 통합되어 OIDC ID 공급자 인증을 지원함


### 3. Classic Load Balancer

CLB는 여러 Amazon EC2인스턴스 간에 기본 로드 밸런싱을 제공하며 요청 수준과 연결 수준 모두에서 작동한다. CLB는 EC2-Classic 네트워크 내에 구축된 애플리케이션을 위한 것이다. 기존 로드 밸런서는 지정된 사용 사례에서 언급된 짧은 지연 시간 및 처리량 시나리오에 적합하지 않다.

</br></br>

## 8. AWS Lambda

Lambda를 사용하면 서버를 프로비저닝하거나 관리할 필요 없이 코드를 실행할 수 있으며, 사용한 컴퓨팅 시간에 대해서만 비용을 지불하면 된다.

Lambda에서는 사실상 모든 유형의 애플리케이션이나 백엔드 서비스에 대한 코드를 별도의 관리 없이 실행할 수 있다. 코드를 업로드하기만 하면, Lambda에서 높은 가용성으로 코드를 실행 및 확장하는데 피룡한 모든 것을 처리한다. 다른 AWS 서비스에서 코드를 자동으로 트리거하도록 설정하거나 웹 또는 모바일 앱에서 직접 코드를 호출할 수 있다.

AWS Lambda는 함수를 실행하고 저장하는데 사용할 수 있는 컴퓨팅 및 스토리지 리소스의 양을 제한한다. 리전당 다음과 같은 한도가 적용되며 이 한도를 늘릴 수 있다. 지정된 시간 초과에 도달하면 AWS Lambda가 Lambda 함수의 실행을 종료한다. 예상 실행 시간에 따라 이 값을 설정하는 것이 좋으며, 기본 시간 초과는 3초이고 AWS Lambda의 됴청 당 최대 실행 기간은 900초(15분)이다.

</br></br>

## 9. Amazon Kinesis Data Streams

Amazon Kinesis Data Streams를 사용하여 대규모 데이터 레코드 스트림을 실시간으로 수집하고 처리할 수 있다. Kinesis Data Streams 애플리케이션이라고 알려진 데이터 처리 애플리케이션을 생성할 수 있다. 일반적으로 Kinesis Data Streams 애플리케이션은 데이터가 기록될 때 데이터 스트림에서 데이터를 읽는다. 이러한 애플리케이션은 Kinesis Client Library를 사용하며 Amazon EC2 인스턴스에서 실행될 수 있다. 처리된 레코드를 대시보드로 보내거나, 알림을 생성하는데 사용하거나, 요금 및 광고 전략을 동적으로 변경하거나, 다른 여러 AWS 제품에 데이터를 보낼 수 있다.

</br></br>

## 10. Amazon Athena

Amazon Athena는 Amazon S3에서 표준 SQL을 사용하여 데이터를 쉽게 바로 분석할 수 있는 대화형 쿼리 서비스이다. AWA Management Console에서 몇 가지 작업을 수행하면 Amazon S3에 저장된 데이터에서 Athena를 가리키고, 표준 SQL을 사용하여 임시 쿼리를 실행하고, 몇 초 안에 결과를 얻을 수 있다.

Athena은(는) 서버리스 서비스이므로 설정하거나 관리할 인프라가 없으며, 실행한 쿼리에 대해서만 비용을 지불합니다. Athena에서는 쿼리를 동시에 실행하여 규모를 자동으로 조절합니다. 따라서 많은 데이터 세트와 복잡한 쿼리가 있더라도 결과를 빠르게 도출합니다.

Athena은(는) Amazon S3에 저장된 비정형, 반정형 및 정형 데이터를 분석하는 데 도움을 줍니다. 예를 들면 CSV, JSON 또는 컬럼 방식 데이터 형식(예: Apache Parquet 및 Apache ORC)이 해당됩니다. Athena을(를) 사용하면 데이터를 집계하거나 Athena(으)로 로드할 필요 없이 ANSI SQL을 사용한 임의 쿼리를 실행할 수 있습니다.

Athena는 간편한 데이터 가상화를 위해 Amazon QuickSight와 통합됩니다. Athena을(를) 사용하면 보고서를 생성하거나 JDBC 또는 ODBC 드라이버를 통해 연결된 비즈니스 인텔리전스 도구 또는 SQL 클라이언트로 데이터를 탐색할 수 있습니다.

</br></br>

## 11. AWS STS(AWS Security Token Service)

AWS Security Token Service(AWS STS)를 사용하면 AWS 리소스에 대한 액세스를 제어할 수 있는 임시 보안 자격 증명을 생성하여 신뢰받는 사용자에게 제공할 수 있습니다. 임시 보안 자격 증명은 다음과 같은 차이점을 제외하고는 IAM 사용자가 사용할 수 있는 장기 액세스 키 자격 증명과 거의 동일한 효력을 지닙니다.

- 임시 보안 자격 증명은 그 이름이 암시하듯 단기적입니다. 이 자격 증명은 몇 분에서 몇 시간까지 지속되도록 구성할 수 있습니다. 자격 증명이 만료된 후 AWS는 더는 그 자격 증명을 인식하지 못하거나 그 자격 증명을 사용한 API 요청으로부터 이루어지는 어떤 종류의 액세스도 허용하지 않습니다.

- 임시 보안 자격 증명은 사용자와 함께 저장되지 않지만 동적으로 생성되어 요청시 사용자에게 제공됩니다. 임시 보안 자격 증명이 만료되었을 때(심지어는 만료 전이라도) 사용자는 새 자격 증명을 요청할 수 있습니다. 단, 자격 증명을 요청하는 해당 사용자에게 그렇게 할 수 있는 권한이 있어야 합니다.


이러한 차이점은 다음과 같은 임시 자격 증명 사용의 이점을 발생시킬 수 있습니다.

- 애플리케이션으로 장기 AWS 보안 자격 증명을 배포 또는 포함할 필요가 없습니다.
- 사용자에 대한 AWS 자격 증명을 정의하지 않고도 AWS 리소스에 대한 액세스 권한을 사용자에게 제공할 수 있습니다. 임시 자격 증명은 역할 및 자격 증명 연동을 위한 기초입니다.
- 임시 보안 자격 증명은 수명이 제한되어 있어서, 더 이상 필요하지 않을 때 교체하거나 명시적으로 취소할 필요가 없습니다. 임시 보안 자격 증명이 만료된 후에는 다시 사용할 수 없습니다. 그 자격 증명에 대해 유효 기간을 최대 한계까지 지정할 수 있습니다.

## 12. CloudFront

- 지리적 차단, 지리적 제한을 사용하면 특정 지리적 위치에 있는 사용자가 CloudFront 배포를 통해 배포된 콘텐츠에 액세스하는 것을 차단할 수 있다. 
- CloudFront 접근제어는 사용자가 서명된 URL을 사용하여 파일에 액세스하고 OAI를 작성한 후 S3 버킷의 파일에 대한 액세스를 OAI에 제한하도록 구성해야 함.

## 13. CloudWatch

1). CloudWatch 로그 파일에 저장하기 위한 유효한 옵션
- CloudWatch Log
- splunk
- S3 사용자 정의 스크립트


- Kinesis data analytics는 표준 SQL을 사용하여 처리하고 Firehose는 SQL 쿼리를 실행할 수 없음

## 14. AutoScaling

1). AutoScaling 의 조정 정책 (아래 2개 모두 예측 불가능한 상황에 적합)
    - 대상 추적 조정 정책 : 조정 지표를 선택하고, 대상 값을 설정 (CPU활용도, 네트워크 인터페이스에서 받은/보낸 평균 바이트 수, 대상그룹 내 대상별 요청 수)
    - 단순 및 단계 조정 정책 : 조정 프로세스를 트리거하는 CloudWatch 경보에 대한 조정 지표와 임계값을 선택하고 위반시 조정 방법을 결정

- AutoScaling은 각 AZ의 인스턴스 수가 균형을 이루지 않을 경우 재조정을 시도함.
    - 먼저 인스턴스가 없는 AZ에서 인스턴스를 생성한 후, 나머지 AZ에서 같은 수의 인스턴스를 종료함
- AutoScaling은 손상된 인스턴스가 확인될 경우, 이를 종료한 후 !!!!!! 새로운 인스턴스로 교체함

</br></br>

## 15. EBS

- EBS 볼륨의 성능을 향상시키기 위해서는 프로비져닝된 볼륨을 사용하고, RAID 0 배열에 여러 볼륨을 추가하는 것
- EBS는 요구사항 증가시 자동으로 증가하지 않으므로, 볼륨크기를 늘린다음 파일 시스템을 확장해야함. -> EFS 의 장점

## 16. Direct Connect

- Direct Connect는 한 지역 내 모든 AZ에 연결할 수 있음

</br></br>

## 17. DynamoDB

1). DynamoDB의 모범 사례
    - 항목 크기를 작게 유지
    - 데이터 / 시간에 기반한 작업이 필요한 경우 일별/주별/월별 테이블 생성
    - 액세스가 빈번한 테이블과 적은 테이블 구분
    - S3에 400KB를 초과하는 객체를 저장하고 DynamoDB에서 포인터 사용

</br></br>

## 18. S3 vpc Endpoint
