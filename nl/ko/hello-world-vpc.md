---

copyright:
  years: 2017, 2018
lastupdated: "2018-09-25"

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:new_window: target="_blank"}
{:pre: .pre}
{:tip: .tip}
{:table: .aria-labeledby="caption"}
{:download: .download}

# CLI를 사용하는 Hello World VPC

이 단계별 안내서는 IBM Cloud CLI를 사용하여 {{site.data.keyword.cloud}} 가상 프라이빗 클라우드(VPC)에서 가상 서버 인스턴스(VSI)를 프로비저닝하는 방법을 설명합니다.

## 전제조건:

1. IBM Cloud 계정을 VPC에 대해 사용할 수 있는지 확인하십시오. 자세한 정보는 [참여 방법](how-to-participate.html)을 참조하십시오.

2. 사용할 수 있는 공개 SSH 키가 있는지 확인하십시오.

   이미 공개 SSH 키를 가지고 있을 수 있습니다. 홈 디렉토리의 ``.ssh`` 디렉토리에 있는 ``id_rsa.pub``이라는 파일을 찾으십시오. 예를 들어, ``/Users/<USERNAME>/.ssh/id_rsa.pub``입니다. 파일은 ``ssh-rsa``로 시작하고 이메일 주소로 끝납니다.

   공개 SSH 키가 없거나 기존 키의 비밀번호가 생각나지 않으면 `ssh-keygen` 명령을 실행하고 프롬프트에 따라 새 키를 생성하십시오.

3. [IBM Cloud CLI ![외부 링크 아이콘](../../icons/launch-glyph.svg "외부 링크 아이콘")](https://console.bluemix.net/docs/cli/reference/bluemix_cli/get_started.html#getting-started){: new_window}를 설치하십시오.

4. IBM Cloud CLI용 `infrastructure-service` 플러그인을 설치하십시오.

```
ibmcloud plugin install infrastructure-service
```
{: codeblock}

## 1단계: IBM Cloud에 로그인하십시오.

연합 계정이 있는 경우, 싱글 사인온(SSO)을 사용하십시오.

```
ibmcloud login -sso
```
{: codeblock}

그렇지 않으면 다음 명령을 사용하십시오.

```
ibmcloud login
```
{: codeblock}

## 2단계: VPC를 작성하고 VPC ID를 저장하십시오.

_helloworld-vpc_라는 이름의 VPC를 작성하려면 다음 명령을 사용하십시오.

```
ibmcloud is vpc-create helloworld-vpc
```
{: codeblock}

다음과 같은 출력이 표시됩니다.

```
Creating VPC helloworld-vpc in resource group Default under account <Account Name> as user <User>...

ID        ba9e785a-3e10-418a-811c-56cfe5669676
Name      helloworld-vpc
Default   no
Created   1 second ago
Status    available
Tags      -
```
{:screen}

나중에 사용할 수 있도록 ID를 변수에 저장하십시오. 예를 들어, 다음과 같습니다.

```
vpc="ba9e785a-3e10-418a-811c-56cfe5669676"
```
{: codeblock}

## 3단계: 서브넷을 작성하고 서브넷 ID를 저장하십시오. 

서브넷의 위치로 `us-south-2` 구역을 선택하고 _helloworld-subnet_ 서브넷을 호출하십시오.

```
ibmcloud is subnet-create helloworld-subnet $vpc us-south-2 --ipv4_cidr_block "10.0.1.0/24"
```
{: codeblock}

다음과 같은 출력이 표시됩니다.

```
Creating Subnet helloworld-subnet in resource group Default under account <Account Name> as user <User>...

ID               50ba0da9-279a-4791-b7cb-cd3d7b2bc14d
Name             helloworld-subnet
IPv*             ipv4
IPv4 CIDR        10.0.1.0/24
IPv6 CIDR        -
Addr available   0
Addr Total       0
Gen              -
ACL              allow-all-network-acl-ba9e785a-3e10-418a-811c-56cfe5669676(e9c2790b-cee2-465a-8539-d8cd90d33621)
Gateway          -
Created          now
Status           pending
Zone             us-south-2
VPC              helloworld-vpc(ba9e785a-3e10-418a-811c-56cfe5669676)
Resource Group   -
Tags             -
```
{:screen}

나중에 사용할 수 있도록 ID를 변수에 저장하십시오. 예를 들어, 다음과 같습니다.

```
subnet="50ba0da9-279a-4791-b7cb-cd3d7b2bc14"
```
{: codeblock}

서브넷이 처음 작성될 때의 상태는 `pending`입니다. 계속 진행하기 전에 서브넷이 `available` 상태로 변경되어야 하며 이 작업에는 몇 초 정도 소요됩니다. 서브넷의 상태를 확인하려면 다음 명령을 실행하십시오.

```
ibmcloud is subnet $subnet
```
{: codeblock}

## 4단계: IBM 퍼블릭 클라우드에서 SSH 키를 작성하십시오.

이 키를 사용하여 가상 서버 인스턴스를 프로비저닝합니다. 하나의 키를 사용하여 여러 개의 가상 서버 인스턴스를 프로비저닝할 수 있습니다.

IBM Cloud 계정에서 사용 가능한 키를 보려면 다음 명령을 실행하십시오.

```
ibmcloud is keys
```
{: codeblock}

키를 작성하려면 다음 명령을 실행하십시오. 경로를 사용자의 `id_rsa.pub` 파일로 대체하십시오.

```
ibmcloud is key-create helloworld-key @$HOME/.ssh/id_rsa.pub
```
{: codeblock}

다음과 같은 출력이 표시됩니다.

```
Creating key helloworld-key in resource group Default under account <Account Name> as user <User>...

ID               859b4e97-7540-4337-9c64-384792b85653
Name             helloworld-key
Type             rsa
Length           2048
FingerPrint      SHA256:hkcAOGB5z7QXqZLHd0kGqhj735qrfMjZLH9PxS/42vA
Key              ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC9i2joQ8eiFVdJ7pOlC6h5+IoBL6wFFygkk9Na3gV8Bi52dv44YOAjSJ2oduguHEtLp5r4eh4+5jiEBkFYkHNUhE0MxlcVZABTYWePXx4QnlmGr99xyOfi6DAHhSRQiSBhmhjcGjbADavDuIgoyKpVXbU9O1If3P0miNpaouaZmr+d68OHt4yPvNnztlluV3JBISJgqJ7pzg6wFF0SrjqtEYKBd8oxwogHu+rmRgT7IF09oSiKpKZRF0VfeLFz+REh9RuKa4Jh63aa2PRIgDKq6HO7MEdfOtGzCzoqqlFKgpl+VgGyT0b5BjQEnWv13cwx02bv5QCwma/GeAOpW0CD user@email.com

Created          now
Resource Group   -
Tags             -
```
{:screen}

나중에 사용할 수 있도록 ID를 변수에 저장하십시오. 예를 들어, 다음과 같습니다.

```
key="859b4e97-7540-4337-9c64-384792b85653"
```

## 5단계: 가상 서버 인스턴스용 프로파일 및 이미지를 선택하십시오. 

인스턴스 프로파일 `b-4x8` 및 이미지 `ubuntu-16.04-amd64`를 선택하십시오. `ubuntu-16.04-amd64`의 이미지 ID를 가져오려면 다음 명령을 실행하십시오.

```
image=$(ibmcloud is images | grep "ubuntu-16.04-amd64" | cut -d" " -f1)
```
{: codeblock}

## 6단계: 가상 서버 인스턴스를 프로비저닝하십시오. 

```
ibmcloud is instance-create helloworld-vsi $vpc us-south-2 b-4x8 $subnet 1000 --image $image --keys $key
```
{: codeblock}

다음과 같은 출력이 표시됩니다.

```
Creating instance helloworld-vsi in resource group Default under account <Account Name> as user <User>...

ID                4562c5c0-9cf7-4406-bc90-ab4baea91057
Name              helloworld-vsi
Gen
Profile           b-4x8
CPU Arch          amd64
CPU Cores         4
CPU Frequency     2000
Memory            8
Primary Intf      primary(2e850924-b5d7-4386-a778-03556d5850c1)
Primary Address   10.0.1.5
Image             ubuntu-16.04-amd64(7eb4e35b-4257-56f8-d7da-326d85452591)
Status            pending
Created           5 seconds ago
VPC               helloworld-vpc(ba9e785a-3e10-418a-811c-56cfe5669676)
Zone              us-south-2
Resource Group    -
Tags              -
```
{:screen}

나중에 사용할 수 있도록 기본 네트워크 인터페이스 `primary intf`의 ID를 변수에 저장하십시오. 예를 들어, 다음과 같습니다.

```
nic="2e850924-b5d7-4386-a778-03556d5850c1"
```
{:codeblock}

인스턴스가 처음 작성될 때의 상태는 `pending`입니다. 계속 진행하기 전에 인스턴스가 `running` 상태로 변경되어야 하며 이 작업에는 몇 분 정도 소요됩니다. 모든 인스턴스의 상태를 확인하려면 다음 명령을 실행하십시오.

```
ibmcloud is instances
```
{: codeblock}


## 7단계: 유동 IP 주소를 작성하십시오.

인터넷에서 가상 서버 인스턴스(VSI)에 로그인하려면 유동 IP 주소가 필요합니다.

```
ibmcloud is floating-ip-reserve helloworld-fip --nic $nic
```
{: codeblock}

다음과 같은 출력이 표시됩니다.

```
Creating floating ip helloworld-fip in resource group Default under account <Account Name> as user <User>...

ID               b9d1cc1f-67db-40e3-81de-9228465170a5
Address          169.61.181.53
Name             helloworld-fip
Target           primary(2e850924-.)
Target Type      intf
Target IP        10.0.1.5
Created          now
Status           available
Zone             us-south-2
Resource Group   -
Tags             -
```
{:screen}

나중에 사용할 수 있도록 `Address`를 변수에 저장하십시오.

```
address=169.61.181.53
```
{: codeblock}

## 8단계: SSH용 기본 보안 그룹에 규칙을 추가하십시오.

VPC용 보안 그룹을 찾으십시오.

```
ibmcloud is vpc-sg $vpc
```
{: codeblock}

다음과 같은 출력이 표시됩니다.
```
Getting default security group of vpc ba9e785a-3e10-418a-811c-56cfe5669676 under account <Account Name> as user <User>...

ID               2d364f0a-a870-42c3-a554-000000981149
Name             errand-drastic-imperial-retail-unlocked-jester
Created          1 week ago
VPC              helloworld-vpc(ba9e785a-3e10-418a-811c-56cfe5669676)
Resource Group   -
Tags             -

Rules
ID                                     Direction   IPv*   Protocol                  Remote
b597cff2-38e8-4e6e-999d-000002031345   inbound     ipv4   all                       errand-drastic-imperial-retail-unlocked-jester(2d364f0a-.)
b597cff2-38e8-4e6e-999d-000002031527   outbound    ipv4   all                       -

```
{:screen}

나중에 사용할 수 있도록 ID를 변수에 저장하십시오.

```
sg=2d364f0a-a870-42c3-a554-000000981149
```
{: codeblock}

SSH를 허용하는 규칙을 작성하십시오.

```
ibmcloud is sg-rulec $sg ingress tcp --port-min=22 --port-max=22
```
{: codeblock}

다음과 같은 출력이 표시됩니다.
```
Creating rule for security group 2d364f0a-a870-42c3-a554-000000981149 under account <Account Name> as user <User>...

ID          b597cff2-38e8-4e6e-999d-000001949921
Direction   inbound
IPv*        ipv4
Protocol    tcp
Min Port    22
Max Port    22
Remote      -
```
{:screen}

## 9단계: 개인 SSH 키를 사용하여 가상 서버 인스턴스에 로그인하십시오.

예를 들어, 다음 양식의 명령을 사용할 수 있습니다.

```
$ ssh -i $HOME/.ssh/id_rsa root@$address
```
{:codeblock}

 다음의 예와 유사한 응답을 수신하게 됩니다. 연결을 계속할지 여부에 대해 프롬프트되면 `yes`를 입력하십시오.

```
The authenticity of host '169.61.181.53 (169.61.181.53)' can't be established.
ECDSA key fingerprint is SHA256:9MczXIwJq892DYwu0sZpQZOXORdvNXeP1aFioZpQDsM.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '169.61.181.53' (ECDSA) to the list of known hosts.
Welcome to Ubuntu 16.04.5 LTS (GNU/Linux 4.4.0-133-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  Get cloud support with Ubuntu Advantage Cloud Guest:
    http://www.ubuntu.com/business/services/cloud

0 packages can be updated.
0 updates are security updates.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

root@helloworld-vsi:~#
```
{:screen}

## 10단계: Hello, World!

터미널 창에서 다음 명령을 실행하십시오.

```
echo `hostname` says "Hello, World!"
```
{:codeblock}

다음 출력이 표시되어야 합니다.

```
helloworld-vsi says Hello, World!
```
{:screen}

## 다음 작업

또 다른 [단계별 안내서](step-by-step-pages.html)를 시도하십시오.
