Envoy 내부 들여다보기

Istio를 깊이 있게 이해하려면, 트래픽의 중심에 있는 Envoy Proxy를 이해하는 것이 중요하며, 이를 위한 이해 수준은 다음 세 가지로 나눌 수 있습니다:
  1. 네이티브로 프로그래밍 가능한 프록시인 Envoy Proxy 아키텍처를 이해하기
  2. Istio의 커스터마이즈된 Envoy Proxy (github.com/istio/proxy)가 Envoy를 어떻게 확장하는지를 이해하기
  3. Istio의 커스터마이즈된 Envoy Proxy를 istiod가 어떻게 프로그래밍 방식으로 제어하여 서비스 메시 기능을 구현하는지를 이해하기

```{toctree}
:caption: Catalog
:maxdepth: 5
:includehidden: 

envoy-overview.md
envoy-istio-conf-eg.md
envoy-high-level-flow/envoy-high-level-flow.md
arch/arch.md
req-resp-flow-timeline/req-resp-flow-timeline.md
connection-life/connection-life.md
envoy-istio-metrics/index.md
upstream/upstream.md
socket/socket.md
```
