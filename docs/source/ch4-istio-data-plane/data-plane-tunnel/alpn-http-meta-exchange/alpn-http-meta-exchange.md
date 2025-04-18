---
typora-root-url: ../../..
---

# HTTP Meta-data Exchange on ALPN/TLS handshake


Let's say you are an HR person in a large company and you have to contact and communicate with different colleagues every day. So what do you do before you start the formal communication content? Introduce yourself and let the other person introduce themselves. Each identifies himself or herself. This information can come from the greeting before the formal conversation starts, or you can look at the other person's personal information on Outlook before starting the conversation.

Similarly, in order for Istio to implement a service mesh, the Istio Proxies needs to exchange meta-information during the interconnection handshake. Therefore, Istio customized the transport layer protocols on top of the native Envoy to enable meta-information exchange.

Let's start by an example:
```
[serviceA app --h2c--> serviceA istio-proxy] ----(http over mTLS)---> [serviceB istio-proxy --h2c--> serviceB app]
```

One basic design principle of `Istio proxy` is know the downstream/upstream meta-data of a new connection as early as possible. Because this allows you to implement some decisions as early as possible. 



**For the client side(Outbound) `Istio proxy` :**

If it knew about the meta-data of upstream in advance :
- The upstream is another `Istio proxy` on the same Istio mesh ?
- Which type of TLS is supported by the upstream ? Auto mutual TLS of Istio mesh ? or manually traditional TLS.
- Which application level protocol are supported by the upstream ? http1.1 or http2 or http3 ?

Client side(Outbound) `Istio proxy`  can also make upstream HTTP version decision based on downstream(app on the same pod)  connection too, see  [use_downstream_protocol_config](https://www.envoyproxy.io/docs/envoy/latest/api-v3/extensions/upstreams/http/v3/http_protocol_options.proto#envoy-v3-api-field-extensions-upstreams-http-v3-httpprotocoloptions-use-downstream-protocol-config).



Client side(Outbound) `Istio proxy` can overwrite the provided ALPN protocol list according to the supported HTTP  version by upstream server.



**For the server side(Inbound) `Istio proxy` :**

For example, when a new connection from downstream is accepted by a `Istio proxy`, it want to know some meta-data of the downstream:
- The downstream is another `Istio proxy` on the same Istio mesh ?
- Which type of TLS is supported by the downstream ? Auto mutual TLS of Istio mesh ? or manually traditional TLS.
- Which application level protocol are supported by the downstream ? http1.1 or http2 or http3 ?

The `Istio proxy` which acts as a server make decisions of `Network Filter Chains` and `Http Filter Chains` selections base on above downstream meta-data.





## Glossary

- h2c - HTTP/2 over TCP or HTTP/2 Cleartext
- h2 - HTTP/2 over TLS (protocol negotiation via ALPN)
- ALPN - [`ALPN(Application-Layer Protocol Negotiation)` on TLS](https://en.wikipedia.org/wiki/Application-Layer_Protocol_Negotiation)



## Cooperation of outbound and inbound istio-proxy



Above section show that outbound and inbound istio-proxy should cooperate to select HTTP version. Below figure show an example of how it work.






:::{figure-md} Figure: HTTP protocol meta-data exchange at high level

<img src="/ch4-istio-data-plane/data-plane-tunnel/alpn-http-meta-exchange/alpn-http-meta-exchange-high-level.drawio.svg" alt="Figure - HTTP protocol meta-data exchange at high level">

*Figure: HTTP protocol meta-data exchange at high level*
:::
*[Open with Draw.io](https://app.diagrams.net/?ui=sketch#Uhttps%3A%2F%2Fistio-insider.mygraphql.com%2Fzh_CN%2Flatest%2F_images%2Falpn-http-meta-exchange-high-level.drawio.svg)*


- Outbound istio-proxy

  Base on the  `istio.alpn HTTP filter` and `Upstream Cluster` meta-data.



- Inbound istio-proxy

  Base on the `Listener->filter_chains->filter_chain_match->application_protocols`  configuration and the ALPN provided by Outbound istio-proxy



A troubleshooting example of ALPN HTTP Meta Exchange: {doc}`/troubleshooting/istio-troubleshooting/http_protocol_options-accidentally-disable-http2/http_protocol_options-accidentally-disable-http2`


Below figure deep dive into the related source code of Envoy Proxy and Istio Proxy. It show you why under the hood.



:::{figure-md}

<img src="/troubleshooting/istio-troubleshooting/http_protocol_options-accidentally-disable-http2/upstream-http-protocol-selection-src.drawio.svg" alt="Figure - upstream http protocol selection example">

*Figure: upstream http protocol selection example*
:::
*[Open with Draw.io](https://app.diagrams.net/?ui=sketch#Uhttps%3A%2F%2Fistio-insider.mygraphql.com%2Fzh_CN%2Flatest%2F_images%2Fupstream-http-protocol-selection-src.drawio.svg)*



## Read more
- [Better Default Networking – Protocol sniffing](https://docs.google.com/document/d/1l0oVAneaLLp9KjVOQSb3bwnJJpjyxU_xthpMKFM_l7o/edit#heading=h.edsodfixs1x7)
- [Istio MTLS Smartness Explained](https://devops-insider.mygraphql.com/zh-cn/latest/service-mesh/istio/istio-mtls/istio-mtls-smartness-explained.html#alpn)

