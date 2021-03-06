IPSec
======

IPsec은 Network Layer이며 Internet Protocol통신을 보호하기 위해 설계된  프로토콜 모음이다. 네트워크 IP계층에서의 보안에 중점을 두며 가상 사설망과 사설망을 사용하는 tcp/ip통신을 안전하게 유지할 수 있도록 인증, 데이터 무결성, 접근제어, 데이터 기밀성 등등의 서비스를 IP 계층에 제공한다. 

IPSec은 세션이 시작할 때 상호 인증을 위한 프로토콜과 세션 중에 사용할 암호키의 교섭이 포함되어 있다. host-to-host, network-to-host, network-to-network의 데이터 흐름을  보호하기 위해 사용되어지고 있다. 
주요 장점으로는 Network Layer에서 동작되기 때문에 application이랑은 무관하게 동작한다. HTTP, FTP등 TCP/IP프로그램에 대한 연결에 대하여 보안이 보장된다. 마지막으로 tunneling mode를 사용하면 다른 프로토콜과도 동작할 수 있다.

IPSec에는 데이터 송신자의 인증을 허용하는 인증 헤더 (AH), 송신자의 인증 및 데이터 암호화를 지원하는 ESP (Encapsulating Security Payload), IKE 프로토콜로 구성 되어있다.

AH(Authentication Header protocol)
-----------------------------------
AH는 패킷에 대한 인증기능을 제공하는 역할을 한다. AH 인증은 IP 헤터 패킷이 전송중일 때 그에 대한 조작을 불가능하게 만든다. 따라서 NAT 종단간 환경에서의 사용을 부적절하게 만들기도 한다. AH의 가장 큰 특징은 출발지 인증, 데이터 무결성은 보장하지만 기밀성은 보장되지 않는 것 이다. 공유 비밀 키를 사용하여 데이터 원본 인증이 보장되며 HMAC-MD5 이나 HMAC-SHA와 같은 알고리즘으로 데이터 무결성을 보장한다. 또한 TTL(Time To Live) 필드와 함께 전송 중에 합법적으로 변경 될 수 있는 헤더 필드를 제외하고 IP 헤더와 해당 페이로드를 인증하게 된다.
AH에는 전송모드와 터널모드가 있다.

![transport](https://user-images.githubusercontent.com/63446087/82060283-21bd3c00-9702-11ea-84ec-1a0294965a9a.png)

* 전송 모드인 경우 IP 패킷을 인증하는데 AH에서 쓰이는 인증은 ESP와 달리 패킷 전체에 대한 인증을 가지고 있다.
  IPSec 통신을 시작할때 쓰이며 상위 계층 프로토콜을 보호하게 된다.


![tunnel](https://user-images.githubusercontent.com/63446087/82044375-35f43f80-96e8-11ea-839f-5bd6f4aa9b2c.png)


* 터널 모드의 경우 새로운 IP 헤더와 AH가 IP 패킷 앞부분에 추가되어 전체 패킷에 대한 인증을 하게 된다. 전송모드와 비슷한 형태를 가지고 있는데  NEW IP Hdr가 있다는 게 차이점이다.
    

![AH outbound](https://user-images.githubusercontent.com/63446087/82045005-4d7ff800-96e9-11ea-9859-efad61df1047.jpg)

    
* AH에서 Outbound는 먼저 SA가 존재하는지 찾아야한다. SA가 개설 되었다면 SN값은 0으로 설정을 해둔다. 셋팅이 되면 패킷 전송이 되고 1씩 증가 할 것이다. 만약 SN이 Overfloew가 되면 SA를 삭제하고 다시 만들어야 한다. 패킷의 ICV를 계산하고 전송을 하면 된다.
* 수신하는 Inbound 패킷 처리는 SA를 찾아야한다. 찾지 못하면 패킷은 버려지게 될 것이다. SA를 찾았다면 SN값을 0으로 설정 한 뒤 들어오는 패킷의 SN을 지켜본다. 중복 된 SN은 버린다.  ICV값과 AH내의 값이 동일하면 넘기고 같지 않으면 버린다.
    
    
    

ESP
---
AH는 기밀성 보장이 지원되지 않았지만 ESP는 출발지 인증, 데이터 무결성 보장, 데이터 기밀성까지 제공한다. AH의 모든 기능에는 암호화된 ESP가 포함되며, 인증되지 않은 데이터에 대한 공격을 방지하기 위한 것이다. 이 서비스들은 별도로 제공이 가능하기 때문에 인증 없이 기밀성만 제공하는 것도 가능하다. 인증 기능을 제공할 때 AH와 같은 알고리즘을 쓰지만 적용 범위가 다르다. AH인증은 외부 IP헤더를 포함한 전체 IP 패킷을 인증하지만 ESP 인증은 IP 패킷의 IP 데이터 그램 부분만을 인증한다.
ESP에는 전송 모드와 터널 모드가 있다.


* 전송 모드의 경우 IP 헤더 뒤에 추가되며 Encryption scope이 암호화 된 부분이다.  

* 터널 모드인 경우 새로운 IP 패킷이 만들어지면서 원래 IP 패킷은 새로운 IP 패킷의 데이터가 된다. 터널 모드에서는 
  IP 헤더도 암호화 되기 때문에 전송 모드 보다 안전하다는 장점을 가지고 있다.

![esp in outbound](https://user-images.githubusercontent.com/63446087/82045421-07776400-96ea-11ea-9088-a9a43e84d74c.jpg)

* 송신하는 Outbound 패킷 처리는 먼저 패킷의 헤더 정보, IP주소, 포트번호를 기준으로 적용할 보안 정책을 찾아야한다. 그 다음 SA가 설정되어 있는지 SA가 없다면 새로운 SA를 설정해야한다. SA가 있다는 가정하에 설명을 이어가자면 전송하고자 하는 패킷을 암호화 하는데 전송 모드일 때는 상위 Layer에서만 암호화를 진행하게 된다. 재전송 공격 방지를 위해 SN(Sequence Number)을 생성해야 하는데 SN값은 0으로 설정해둔다. 패킷이 하나씩 나갈 때마다 1만큼 증가하며 수신측에서 예측 가능한 필드 값을 사용하여 ESP 패킷의 ICV를 계산하고 전송할 패킷 단위로 분할 한 뒤 전송하면 된다.

* 수신하는 Inbound 패킷 처리는 수신된 패킷을 정리한 뒤 AH 헤더의 목적지 주소를 이용하여 SA를 찾는다. 이때 SA가 없는 경우 해당 패킷을 버리고 SA가 있을 경우 SN값을 0으로 설정하고 들어오는 패킷의 SN을 지켜봐야한다. 만약 중복 SN을 가진 패킷이 들어오면 그것도 버린다. 그 다음 복호화를 하기 전에 SN과 ICV에 대한 검증을 한 뒤 패킷의 복호화 수행을 하면 된다. 검증단계에서 수신 패킷의 ICV값과 AH내의 값이 동일하면 넘기고 같지 않으면 버린다.

IPSec VPN
----------
VPN 가상 사설망이다. VPN 네트워크에 접속하게 되면 외부 컴퓨터라도 내부 네트워크에 접속해 있는 것처럼 사용할 수 있게 된다. VPN을 사용하는 이유는 인터넷에 연결된 장치에는 장치를 식별하고 연결하는데 이용되는 공용IP주소가 있는데 해당 데이터는 인터넷을 통해 전송되어지는데 IP주소 정보를 포함하고 있다. 안전하지 않은 네트워크를 통해 전송되면 중간에 차단하는 역할을 할 것이다. 인터넷을 통해 전송되는 개인정보, 데이터를 보호하기 위해 VPN을 사용하게 된다.

**라우터를 이용한 IPSec VPN**

![VPN](https://user-images.githubusercontent.com/63446087/82045762-997f6c80-96ea-11ea-9519-2c046adfbe31.jpg)

* Cisco Packet Tracer 프로그램으로 구현 환경을 만들어 주었다.
  IPSec은 VPN 연결을 통하여 데이터를 전송할 때 데이터를 암호화 하는 기능을 수행하기 때문에 내부의 데이터를 보호 해줄 수 있다.
  
 <img width="270" alt="경기" src="https://user-images.githubusercontent.com/63446087/82045897-d9465400-96ea-11ea-974a-5f0136187f12.png">
 
![경기 isakmp](https://user-images.githubusercontent.com/63446087/82045899-da778100-96ea-11ea-8f7f-6e34eddfba6e.jpg)

* 암호화는 AES방식을 이용하였고 lifetime은 1800초를 두었다.

 
 <img width="230" alt="IPSec" src="https://user-images.githubusercontent.com/63446087/82045986-fc710380-96ea-11ea-8248-2f2574d15eed.png">
 
*  IPSec이 잘 구현되었는지 시뮬레이션 해보았다.
 




## 참고

* 참고문헌

   Issues and Security on IPSec: Survey(홍성혁;2014)
   
   TCP Performance Analysis on IPSec Environments(양광식)


 
* 이미지 출처
https://www.itfind.or.kr/WZIN/jugidong/952/95201.html

