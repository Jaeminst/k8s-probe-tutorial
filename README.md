# k8s-probe-tutorial
kubernetes Probe Tutorials

# livenessProbe
컨테이너 시작시 {initialDelaySeconds}초 대기 후, 컨테이너에 연결을 시도하고 {periodSeconds}초마다 수행, 탐침이 원하는 값이 아닌 값을 반환시 컨테이너를 재시작 한다.

> gRPC를 제외한 http, tcp 프로토콜에서는 ports를 선언하여 전달할 수 있습니다.

## 탐침 수행 방법
1. exec: 성공시 0을 반환
2. httpGet: (200 <= code < 400) ? true : false
3. tcpSocket: http검사와 유사하며 tcp 연결을 합니다.
4. grpc: etcd pod을 생성하여 gRPC 상태 확인 프로토콜을 구현합니다.

## gRPC
GRPC 서비스로서 기존의 모든 청구, 할당량 인프라 등을 재사용할 수 있으므로 서버가 상태 확인 서비스에 대한 액세스를 완전히 제어합니다.
https://github.com/grpc/grpc/blob/master/doc/health-checking.md

### gRPC 장점
높은 수준의 서비스를 만들어주는 몇 가지 이점을 제공합니다. 

 1. GRPC 서비스 자체이기 때문에 상태 확인을 수행하는 것은 일반 rpc와 동일한 형식입니다. 
 
 2. 서비스별 건강 상태와 같은 풍부한 의미를 가지고 있습니다. 
 
 3. GRPC 서비스로서 기존의 모든 청구, 할당량 인프라 등을 재사용할 수 있으므로 서버가 상태 확인 서비스에 대한 액세스를 완전히 제어합니다.

# 프로브 구성
프로브 에는 활성 및 준비 상태 확인 동작을 보다 정확하게 제어하는 데 사용할 수 있는 여러 필드가 있습니다.

- initialDelaySeconds
활성 상태 또는 준비 상태 프로브가 시작되기 전에 컨테이너가 시작된 후 시간(초)입니다. 기본값은 0초입니다. 최소값은 0입니다.

- periodSeconds
프로브를 수행하는 빈도(초)입니다. 기본값은 10초입니다. 최소값은 1입니다.

- timeoutSeconds
프로브가 시간 초과되는 시간(초)입니다. 기본값은 1초입니다. 최소값은 1입니다.

- successThreshold
프로브가 실패한 후 성공적인 것으로 간주되기 위한 최소 연속 성공입니다. 기본값은 1입니다. 활성 및 시작 프로브의 경우 1이어야 합니다. 최소값은 1입니다.

- failureThreshold
프로브가 실패하면 Kubernetes는 failureThreshold포기하기 전에 여러 번 시도합니다. 활성 프로브의 경우 포기는 컨테이너를 다시 시작하는 것을 의미합니다. 준비 상태 프로브의 경우 Pod는 Unready로 표시됩니다. 기본값은 3입니다. 최소값은 1입니다.

> deployment 구성 참고
- minReadySeconds
`.spec.minReadySeconds`는 새롭게 생성된 파드의 컨테이너가 어떤 것과도 충돌하지 않고 사 용할 수 있도록 준비되어야 하는 최소 시간(초)을 지정하는 선택적 필드이다. 
대기중에도 프로브가 탐침을 수행할 수 있습니다.

## HTTP 프로브
HTTP 프로브의 경우 kubelet은 검사를 수행하기 위해 지정된 경로와 포트로 HTTP 요청을 보냅니다. 

- host
연결할 호스트 이름, 기본값은 포드 IP입니다. 대신 httpHeaders에서 "호스트"를 설정하고 싶을 것입니다.

- scheme
호스트(HTTP 또는 HTTPS)에 연결하는 데 사용할 스키마입니다. 기본값은 HTTP입니다.

- path
HTTP 서버에서 액세스할 수 있는 경로입니다. 기본값은 /입니다.

- httpHeaders
요청에 설정할 사용자 정의 헤더입니다. HTTP는 반복되는 헤더를 허용합니다.

- port
컨테이너에서 액세스할 포트의 이름 또는 번호입니다. 숫자는 1에서 65535 사이여야 합니다.

## TCP 프로브
TCP 프로브의 경우 kubelet은 Pod가 아닌 노드에서 프로브 연결을 만듭니다. 이는 kubelet이 확인할 수 없기 때문에 매개변수에 서비스 이름을 사용할 수 없음을 의미합니다.

릴리스 1.21 이전에는 활성 또는 시작 프로브에 실패한 컨테이너를 종료하는 데 포드 수준 이 사용되었습니다.
1.21 이상에서 feature-gate `ProbeTerminationGracePeriod`가 활성화되면 사용자는 프로브 사양의 일부로 프로브 수준의 `terminationGracePeriodSeconds`를 지정할 수 있습니다.
- featureGate.ProbeTerminationGracePeriod: true
  - terminationGracePeriodSeconds: {종료되기 위한 대기시간}