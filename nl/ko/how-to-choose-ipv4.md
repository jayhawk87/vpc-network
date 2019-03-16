---

copyright:
  years: 2017, 2018
lastupdated: "2018-10-31"

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}


# VPC용 IP 범위 선택

다음과 같은 CIDR 표기법을 사용하십시오.

* `<IPv4 address>/number`(VPC 주소 예: 10.10.0.0/16)

동일한 VPC 내의 다양한 서브넷 IP 주소용으로 사용할 수 있도록 IPv4의 마지막 16비트(65,536개의 주소)를 0으로 예약하고자 할 것입니다(서브넷 IP 주소 예: 10.10.1.0/24).

**참고:** 이 규칙은 RFC 1918에 의해 권장됩니다.

CIDR 표기법을 처음 사용하는 경우, 슬래시 뒤의 숫자가 적을수록 **더 많은** IP 주소를 할당함에 유의하십시오. 슬래시 뒤의 숫자는 서브넷의 접두부 마스크 내의 선행 1의 수를 나타내기 때문입니다.

자세한 정보가 필요하면 온라인에서 찾을 수 있는 _CIDR(Classless Inter-Domain Routing)_ 관련 여러 우수 기사를 참조하십시오.
