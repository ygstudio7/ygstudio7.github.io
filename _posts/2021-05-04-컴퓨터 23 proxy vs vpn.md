# proxy vs vpn

ref:

https://blog.naver.com/PostView.nhn?blogId=reductionist101&logNo=221567693949&proxyReferer=https:%2F%2Fwww.google.com%2F

### proxy

IP 우회를 통해 해외 사이트에 접속하는 것과 같은 목적으로 사용된다.

유투브나 웹사이트에서 지역 제한이 있는 경우, IP 주소의 특정 대역을 특정 지역에 할당해 놓고, 이를 통해서 웹사이트 관리자나 서버 관리자가 특정 IP 대역을 차단하면, 그 IP를 사용하는 사람은 그 사이트를 사용할 수 없게 된다. 

이때 proxy 서버를 사용하면, 그 서버가 우리의 요청을 대신 사이트에 요청해주고, 응답 결과를 다시 우리에게 전달해준다, 하지만, proxy 서버는 IP 주소를 숨겨줄뿐, 데이터를 암호화하지 않는다.



### VPN (Virtual Private Network)

VPN은 전송 내용을 암호화하며, 스누핑을 피하는등 중대한 목적으로 사용된다.

vpn은 요청과 응답 결과를 암호화하여 전달하므로, 다른 사용자가 데이터를 탈취해도 무슨 내용인지 알 수 없다. 그러므로 금융 거래나 보안이 필요한 문서의 송수신시에는 프록시가 아닌 VPN을 사용해야 안전하다.



정리하자면, 

* proxy: 프록시 서버가 요청과 응답을 대신해줘서, IP를 숨길 수 있지만, 데이터 암호화는 없는 단점이 있다.
* VPN: proxy + 데이터 암호화로 금융거래, 보안 문서 송수신시 안전하다.
* 



# 210514: 크롬 프록시 설정방법

참고: https://ckghwn83.tistory.com/338#:~:text=%ED%81%AC%EB%A1%AC%20%EB%B8%8C%EB%9D%BC%EC%9A%B0%EC%A0%80%20%ED%94%84%EB%A1%9D%EC%8B%9C%20%EC%84%A4%EC%A0%95%20%EB%B0%A9%EB%B2%95&text=%EC%8A%A4%ED%81%AC%EB%A1%A4%EC%9D%84%20%EB%82%B4%EB%A0%A4%EC%84%9C%20%EA%B3%A0%EA%B8%89%EC%9D%84,%ED%8F%AC%ED%8A%B8%EB%A5%BC%20%EC%9E%85%EB%A0%A5%EC%9D%84%20%ED%95%A9%EB%8B%88%EB%8B%A4.



구글: 크롬 프록시 설정방법



# 210521: vpn추천

https://netxhack.com/vpn/freevpn-privacy/

: 아주 좋은 분석 글들

Nord vpn, express vpn 추천

https://netxhack.com/vpn/surfshark-vpn/

: Surfshark VPN 추천



https://surfshark.com/blog/anonymous-vpn

: 24 개월 plan구입, $2.49 x 24 = , with ollehbaker@gmail.com

![image-20210521233156774](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210521233156774.png)



![image-20210521233139286](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210521233139286.png)



m54에서 SurfShark 실행  > 러시아로 설정

m54를 재실행시 다음과 같이 위치 확인 가능

![image-20210521233846711](e:\_ygkim\_Doc\_future\\_work\github pages\ygstudio7\ygstudio7.github.io\images\image-20210521233846711.png)

