# Istio 아키텍처

어렸을 때 나는 라디오, CD 플레이어, 컴퓨터 같은 전자기기를 분해했다가 다시 조립하는 걸 참 좋아했다. 분해한 걸 다시 원래대로 조립할 수 있다는 건 일종의 마법 같은 능력이었고, 그러면서 구조를 이해하게 됐다. 그런데 요즘 내 아이들 세대를 보면, 그런 것에 대한 관심이나 기회 자체가 별로 없는 것 같다. 한번 생각해보자. 어떤 아이가 아이패드를 분해하겠는가… 설령 분해한다 해도 부품이 너무 작고 정밀해서 메커니즘을 알아보기도 어렵다. 이제는 스스로 배워가는 식의 학습자가 점점 사라지고 있다.

라디오의 작동 원리를 배우는 것처럼, 기술적인 학습도 두 가지 방향이 있다:
  - 큰 것에서 작은 것으로 (또는 상향식, top-down)
전체를 먼저 보고, 기능, 아키텍처 구성 요소들, 구성 요소 간의 관계, 외부 인터페이스, 데이터 흐름 등을 살펴본다. 예를 들면 HTTP 요청이 Istio 아키텍처를 통과하는 과정을 보는 것이다.
  - 작은 것에서 큰 것으로 (또는 하향식, bottom-up, 또는 bottom-to-top)
예를 들어 보면:
  - Istio 사이드카 트래픽 인터셉션을 위한 iptables / netfilter / conntrack
  - Istio Destination Rule과 Istio Virtual Service를 위한 Envoy HTTP Filter / Route

하지만 대부분의 경우 이 두 방법을 조합해서 사용하게 된다.



```{toctree}
:hidden:
istio-arch-overview
service-mesh-base-concept
istio-ports-components
istio-data-panel-arch
```
