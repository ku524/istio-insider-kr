# Istio 전체 아키텍처

Istio의 전체 아키텍처는 이 책의 핵심 주제가 아니다. 이 책을 읽고자 하는 사람이라면, 아마 이미 어느 정도 학습을 했을 것이다.  
이 섹션의 주요 목적은 전체 아키텍처를 간단히 복습하는 데 있다. 독자 여러분은 아마도 Istio 사용자이거나, 이미 어느 정도 경험이 있는 사용자일 것이다. 하지만 어떤 주제에 너무 깊게 몰입하다 보면, 오히려 전체 그림을 잊기 쉽다.  

또한 이 지점은 이 책에서 어떤 부분에 집중할 것인지 설명하기에도 좋은 시점이다. 어차피 내 에너지와 관심은 한정적이기 때문에, Istio 전체가 아니라 특정 부분에만 집중할 예정이다.

:::{figure-md} Istio 전체 아키텍처

<img src="index.assets/istio-arch.svg" alt="Istio overall architecture">

그림: Istio 아키텍처  
출처: https://istio.io/latest/docs/ops/deployment/architecture/  
:::

- Proxy  
  따로 설명이 필요 없을 정도로 잘 알려진 구성 요소다. 데이터 플레인에서 가장 중요한 컴포넌트이며, 이 책의 핵심 주제이기도 하다. 나는 컨트롤 플레인보다는 데이터 플레인에 더 관심이 많기 때문이다.  
  여기서 말하는 Proxy는 `istio-proxy` 컨테이너를 의미하며, 이 안에는 최소 두 가지 주요 구성 요소가 포함되어 있다:
  - 컨트롤 플레인에 속하는 `pilot-agent`  
  - 데이터 플레인에 속하는 `Envoy Proxy` — 이 책의 첫 번째 주요 초점이다.

- istiod  
  별명: 컨트롤 플레인의 두뇌, 전략적 지휘 센터, 권위 있는 인증 기관.

좋다, 여기까지가 고수준 차트에 대한 설명이다. 이제부터는 이 구성 요소들을 하나씩 분해하고, 이들 간의 상호작용을 분석해보자. 출발!
