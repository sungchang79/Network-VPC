## Network > VPC > 개요

VPC(Virtual Private Cloud)는 논리적으로 격리된 가상의 네트워크에서 NHN Cloud의 리소스를 운영할 수 있는 기능을 제공합니다. 각각의 VPC는 완전히 독립된 서브넷, 라우팅테이블과 게이트웨이를 구성하고 제어할 수 있습니다.
사용자는 VPC 구성을 콘솔을 사용하여 간단히 정의할 수 있습니다. 인터넷 액세스를 할 수 있는 서비스를 연결하거나 폐쇄된 서브넷에서 데이터베이스나 어플리케이션을 구동할 수 있으며 보안 그룹을 사용하여 각 요소를 보호 할 수 있습니다.
또한 TCC1 호스팅과 연결 기능을 제공하여 신뢰있는 호스팅과 클라우드 서비스의 장점을 모두 가질 수 있습니다.




### 주요 기능

NHN Cloud VPC는 퍼블릭 네트워크나 데이터 센터 내 호스팅 연결 혹은 인터넷 게이트웨이를 통한 외부 연결을 지원합니다.

* VPC는 전체를 아우르는 프라이빗 IP 주소 범위를 지정합니다. VPC 내 각각의 서브넷은 VPC의 IP 주소 범위 내에서 자유롭게 생성이 가능합니다.

* VPC 각각의 서브넷을 서로 다른 연결 방법으로 사용할 수 있습니다. 특정 서브넷은 외부 액세스가 가능한 웹 서비스,
또 다른 서브넷은 폐쇄되는 데이터베이스, 또 다른 서브넷은 다른 인터넷 게이트웨이를 연결하여 분할된 네트워크 연결을 지원합니다.

* 라우팅 테이블을 이용하여 각 서비스에 맞는 네트워크 구성을 할 수 있습니다. 각 서비스 별로 서브넷이 나뉘어져 있는 경우나
특정한 네트워크 서비스의 경우에는 라우팅 테이블을 원하는 대로 수정하여 사용할 수 있습니다.

* 플로팅 IP를 이용하여 인스턴스를 인터넷에서 직접 액세스할 수 있습니다.

* 보안 그룹을 이용하여 인스턴스에 대한 접근 제어를 할 수 있습니다.

* 특정한 인스턴스에 복수개의 인터페이스를 적용할 수 있습니다. 목적에 따라 특정 인스턴스의 경우 여러 서브넷에 대해 서비스해야 하는 경우, 해당 인스턴스에는 여러 서브넷의 인터페이스를 적용할 수 있습니다.

* VPC는 NHN Cloud 다른 서비스와의 연결을 숨겨진 형태로 제공합니다.

<br>
### 용어 및 표기

용어  | 표기 | 설명
------------- | ------------- | -------------------
Subnet  | 서브넷 | Subnetwork를 의미하며 IP 네트워크의 중 더 작은 단위로 세분화 된 IP 주소 영역입니다.<br><https://en.wikipedia.org/wiki/Subnetwork>
Internet Gateway| 인터넷 게이트웨이 | 통상적으로 게이트웨이(gateway)로 알려져 있으며 Subnet에 의해 구성된 네트워크가 외부와 연결되는 통로를 의미합니다.<br><https://en.wikipedia.org/wiki/Gateway_(telecommunications)#Internet_gateway>
Routing Table | 라우팅 테이블 | CIDR notation에 의해 정의되는 전달 경로를 기술하는 테이블로 목적지 주소에 의해 전달되는 장비를 지정하거나 우회 시킬 수 있습니다.<br><https://en.wikipedia.org/wiki/Routing_table>
Private Network| 프라이빗 네트워크 | 인터넷 주소 구조에서 사설 네트워크를 정의하는 IP 영역으로 표기된 네트워크를 의미하며 퍼블릭 망에서 이들 트래픽은 전달되지 않습니다.<br><https://en.wikipedia.org/wiki/Private_network>
Interface | 인터페이스 | 인스턴스를 네트워크에 연결하기 위한 장치를 네트워크 인터페이스라고 합니다.

