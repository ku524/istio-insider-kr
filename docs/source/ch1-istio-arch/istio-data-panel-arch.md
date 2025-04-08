# Istio 데이터 플레인 아키텍처

어떤 시스템의 핵심 동작 원리를 이해하고자 한다면, 우선 그 시스템의 주요 데이터 흐름을 살펴봐야 한다. Istio도 예외는 아니다. 아래는 Istio 데이터 플레인의 배포 아키텍처를 나타낸 것이다.

```{note}
이 섹션에서 사용되는 실습 환경에 대한 설명은 다음에서 확인할 수 있다: {ref}`appendix-lab-env/appendix-lab-env-base:Simple layered lab environment`
```

:::{figure-md} 그림: Istio 데이터 플레인 아키텍처

<img src="istio-data-panel-arch.assets/istio-data-panel-arch.drawio.svg" alt="Inbound and Outbound concepts">

*그림: Istio 데이터 플레인 아키텍처*
:::
*[Draw.io로 열기](https://app.diagrams.net/?ui=sketch#Uhttps%3A%2F%2Fistio-insider.mygraphql.com%2Fzh_CN%2Flatest%2F_images%2Fistio-data-panel-arch.drawio.svg)*

{ref}`Figure: Istio Data Plane Architecture`는 다음 호출 체인에 대한 데이터 플레인 관계도이다:  
`client ➔ fortio-server:8080 ➔ fortio-server-l2:8080`.  
다이어그램에 표시된 숫자는 포트 번호이다.

## netfilter/iptables

{ref}`Figure: Istio Data Plane Architecture`에 있는 `kernel netfilter`는 TCP 연결에 대한 일부 인터셉션 및 포워딩 규칙을 의미하며, 다음과 같은 방식으로 확인할 수 있다:

```bash
export WORKNODE=xzy # 관심 있는 POD가 실행 중인 워커 노드
ssh $WORKNODE
export POD=fortio-server # 확인하려는 POD 이름
ENVOY_PIDS=$(pgrep envoy)
while IFS= read -r ENVOY_PID; do
    if [ $(sudo nsenter -u -t $ENVOY_PID hostname)=="$POD" ]; then
        export TARGET_ENVOY_PID=$ENVOY_PID
    export TARGET_ENVOY_PID=$ENVOY_PID
done <<< "$ENVOY_PIDS"

sudo nsenter -n -t $TARGET_ENVOY_PID iptables-save
```

출력 결과:

```
*nat
:PREROUTING ACCEPT [1112:66720]
:INPUT ACCEPT [1112:66720]
:OUTPUT ACCEPT [152:13538]
:postprocessing accpt [152:13538] :istio_inbounds
:istio_inbound - [0:0] :istio_in_routing - [0:0
:ISTIO_IN_REDIRECT - [0:0] :ISTIO_OUTPUT - [0:0
:ISTIO_OUTPUT - [0:0] :ISTIO_REDIRECT - [0:0] :ISTIO_REDIRECT - [0:0
:ISTIO_REDIRECT - [0:0] :ISTIO_REDIRECT - [0:0]
-A PREROUTING -p tcp -j ISTIO_INBOUND
-A OUTPUT -p tcp -j ISTIO_OUTPUT
-A ISTIO_INBOUND -p tcp -m tcp --dport 15008 -j RETURN
-A ISTIO_INBOUND -p tcp -m tcp --dport 22 -j RETURN
-A ISTIO_INBOUND -p tcp -m tcp --dport 15090 -j RETURN
-A ISTIO_INBOUND -p tcp -m tcp --dport 15021 -j RETURN
-A ISTIO_INBOUND -p tcp -m tcp --dport 15020 -j RETURN
-A ISTIO_INBOUND -p tcp -j ISTIO_IN_REDIRECT
-A ISTIO_IN_REDIRECT -p tcp -j REDIRECT --to-ports 15006
-A ISTIO_OUTPUT -s 127.0.0.6/32 -o lo -j RETURN
-A ISTIO_OUTPUT ! -d 127.0.0.1/32 -o lo -m owner --uid-owner 1337 -j ISTIO_IN_REDIRECT
-A ISTIO_OUTPUT -o lo -m owner ! --uid-owner 1337 -j RETURN
-A ISTIO_OUTPUT -m owner --uid-owner 1337 -j RETURN
-A ISTIO_OUTPUT ! -d 127.0.0.1/32 -o lo -m owner --gid-owner 1337 -j ISTIO_IN_REDIRECT
-A ISTIO_OUTPUT -o lo -m owner ! --gid-owner 1337 -j RETURN
-A ISTIO_OUTPUT -m owner --gid-owner 1337 -j RETURN
-A ISTIO_OUTPUT -d 127.0.0.1/32 -j RETURN
-A ISTIO_OUTPUT -j ISTIO_REDIRECT
-A ISTIO_REDIRECT -p tcp -j REDIRECT --to-ports 15001
COMMIT

```
