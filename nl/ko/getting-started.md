---

copyright:
  years: 2017, 2018
lastupdated: "2018-11-29"

---

{:shortdesc: .shortdesc}
{:new_window: target="_blank"}
{:codeblock: .codeblock}
{:pre: .pre}
{:screen: .screen}
{:tip: .tip}
{:download: .download}
{:DomainName: data-hd-keyref="DomainName"}

# 가상 프라이빗 클라우드에 대한 네트워킹 시작하기

[IBM 콘솔 ![외부 링크 아이콘](../../icons/launch-glyph.svg "외부 링크 아이콘")]( https://console.bluemix.net/vpc), [IBM Cloud 명령행 인터페이스](https://{DomainName}/docs/infrastructure-service-cli-plugin/vpc-cli-reference.html) 또는 [VPC REST API](https://{DomainName}/apidocs/rias)를 통해 {{site.data.keyword.cloud}} 가상 프라이빗 클라우드(VPC)를 작성합니다.

## 전제조건

 * **사용자 권한**: 사용자가 VPC에서 리소스를 작성하고 관리하기에 충분한 권한을 보유하고 있어야 합니다. 필수 권한 목록을 보려면 [VPC 사용자에게 필요한 권한 부여](https://{DomainName}/docs/infrastructure/vpc/vpc-user-permissions.html)를 참조하십시오.

 * **SSH 키 준비**: SSH 키를 사용하여 가상 서버 인스턴스에 연결합니다.

   * 홈 디렉토리 아래의 `.ssh` 디렉토리 아래에 있는 `id_rsa.pub`이라는 파일을 찾으십시오. 예를 들어, `/Users/<USERNAME>/.ssh/id_rsa.pub`입니다. 파일은 `ssh-rsa`로 시작하고 이메일 주소로 끝납니다.

   * 또는 SSH 키를 생성하십시오. 공개 SSH 키가 없거나 기존 키의 비밀번호가 생각나지 않으면 `ssh-keygen` 명령을 실행하고 프롬프트에 따라 새 키를 생성하십시오. 예를 들어, `ssh-keygen -t rsa -C "user_ID"` 명령을 실행하여 Linux 서버에서 SSH 키를 생성할 수 있습니다. 해당 명령은 두 개의 파일을 생성합니다. 생성된 공개 키는 `<your key>.pub` 파일에 있습니다.

* **CLI를 사용할 수 있도록 IBM Cloud CLI를 다운로드하고 인프라 플러그인 설치**: `ibmcloud` CLI를 사용하려면 [CLI를 다운로드하고 인프라 플러그인을 설치](https://{DomainName}/docs/infrastructure-service-cli-plugin/vpc-cli-reference.html)해야 합니다.

* **API 엔드포인트**: REST API에 액세스하려면, 계정 소유자가 온보딩 중에 API 엔드포인트를 수신해야 합니다. 시작하려면 [예제 명령](https://{DomainName}/docs/infrastructure/vpc/example-code.html)을 체크아웃하십시오.

## 시작하기 위한 단계

다음은 선택한 인터페이스 방법에 상관없이 VPC용 네트워크를 시작하기 위한 최소 필수 단계입니다.

1. 가상 프라이빗 클라우드를 작성하십시오.
2. 하나 이상의 구역에서 가상 프라이빗 클라우드에 하나 이상의 서브넷을 작성하십시오.
3. 서브넷의 리소스가 인터넷에 액세스하도록 허용하거나 그 반대를 허용하려면 서브넷에 퍼블릭 게이트웨이(PGW)를 작성하십시오.

[VPC를 설정하고 서브넷을 작성하는 방법을 설명하는 간단한 튜토리얼이 여기에 있습니다.]()

## 다음 단계

가상 서버 인스턴스 및 스토리지를 포함하여 전체 VPC 및 리소스 작성을 시작하십시오! 

고급 접근법을 사용하려면 계속 진행하기 전에 [전체 VPC 네트워크 배치를 디자인](https://{DomainName}/docs/infrastructure/vpc/getting-started.html)하십시오.

또는 빠른 시작을 원하는 경우

* [Hello World VPC](hello-world-vpc.html) 문서를 참고하면 IBM Cloud CLI를 사용하여 VSI를 작성하는 방법을 배울 수 있습니다.
* 또는 [예제 코드](https://{DomainName}/docs/infrastructure/vpc/example-code.html) 문서를 참고하면 API를 사용하여 VPC 및 모든 리소스를 작성할 수 있습니다.

