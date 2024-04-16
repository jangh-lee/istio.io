---
title: Architecture
description: Describes Istio's high-level architecture and design goals.
weight: 10
aliases:
  - /docs/concepts/architecture
  - /docs/ops/architecture
owner: istio/wg-environments-maintainers
test: n/a
---

An Istio service mesh is logically split into a **data plane** and a **control
plane**.

* The **data plane** is composed of a set of intelligent proxies
  ([Envoy](https://www.envoyproxy.io/)) deployed as sidecars. These proxies
  mediate and control all network communication between microservices. They
  also collect and report telemetry on all mesh traffic.

* The **control plane** manages and configures the proxies to route traffic.

The following diagram shows the different components that make up each plane:

{{< image width="80%"
    link="./arch.svg"
    alt="The overall architecture of an Istio-based application."
    caption="Istio Architecture"
    >}}

## 구성요소

The following sections provide a brief overview of each of Istio's core components.

### Envoy

Istio는 확장 버전의 
[Envoy](https://www.envoyproxy.io/) 프록시를 사용합니다. Envoy는 고성능의 프록시로 C++로 개발되었습니다. 서비스 매시를 통해 모든 서비스의 인바운드와 아웃바운드 트래픽을 중재할 수 있습니다.
Envoy 프록시는 Istio 구성요소 중 데이터 플레인과 통신하는 유일한 구성요소입니다.
Envoy 프록시는 서비스마다 사이드카 형태로 배포되고, 논리적으로 다양한 내장기능을 통해 서비스에 큰 역할을 합니다.

Envoy의 역할
* 동적 서비스 디스커버리
* 로드밸런싱
* TLS 제거(termination)
* HTTP/2와 gRPC 프록시
* 서킷 브레이커(Circuit breakers)
* 헬스 체크
* % 기반의 트래픽 단계적 분배 (Staged rollouts with %-based traffic split)
* 실패 주입(Fault injection)
* 다양한 매트릭

이 사이드카 형식의 배포를 통해 Istio는 정책 결정을 적용하고 모니터링 시스템으로 전송하는 다양한 원격 측정 항목을 추출해 전체 메시 동작의 정보를 제공합니다.

또한 사이드카 프록시 모델을 통해 코드를 다시 설계하거나 작성할 필요 없이 기존의 배포된 Istio에 기능을 확장할 수 있습니다.

Envoy 프록시를 통해 지원되는 Istio의 기능은 다음과 같다:
* 트래픽 제어 기능 : HTTP, gRPC, 웹소켓, TCP 통신과 같은 다양한 라우팅 규칙을 통한 정교한 트래픽 제어

* 네트워크 탄력 기능 : 설정 재시도(setup retries), 재해복구(failovers), 서킷 브레이커와 실패 주입 

* Security and authentication features: enforce security policies and enforce
  access control and rate limiting defined through the configuration API.

* Pluggable extensions model based on WebAssembly that allows for custom policy
  enforcement and telemetry generation for mesh traffic.

### Istiod

Istiod provides service discovery, configuration and certificate management.

Istiod converts high level routing rules that control traffic behavior into
Envoy-specific configurations, and propagates them to the sidecars at runtime.
Pilot abstracts platform-specific service discovery mechanisms and synthesizes
them into a standard format that any sidecar conforming with the
[Envoy API](https://www.envoyproxy.io/docs/envoy/latest/api/api) can consume.

Istio can support discovery for multiple environments such as Kubernetes or VMs.

You can use Istio's
[Traffic Management API](/docs/concepts/traffic-management/#introducing-istio-traffic-management)
to instruct Istiod to refine the Envoy configuration to exercise more granular control
over the traffic in your service mesh.

Istiod [security](/docs/concepts/security/) enables strong service-to-service and
end-user authentication with built-in identity and credential management. You
can use Istio to upgrade unencrypted traffic in the service mesh. Using
Istio, operators can enforce policies based on service identity rather than
on relatively unstable layer 3 or layer 4 network identifiers.
Additionally, you can use [Istio's authorization feature](/docs/concepts/security/#authorization)
to control who can access your services.

Istiod acts as a Certificate Authority (CA) and generates certificates to allow
secure mTLS communication in the data plane.
