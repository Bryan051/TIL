# 배포
- 용어 정리
  
|용어|뜻|
|:---|:---|
|컨테이너|앱이 구동되는 환경까지 감싸서 실행할 수 있도록 하는 격리 기술|
|컨테이너 런타임|컨테이너를 다루는 도구|
|도커|컨테이너를 다루는 도구 중 가장 유명한 것|
|쿠버네티스|컨테이너 런타임을 통해 컨테이너를 오케스트레이션 하는 도구|
|오케스트레이션|여러 서버에 걸친 컨테이너 및 사용하는 환경 설정을 관리하는 행위|

 ![image](https://github.com/Bryan051/TIL/assets/68111122/583b79fe-ad10-4544-89c8-90870883a711)

- 기존 배포 환경은 특정 프로그램이 성능을 독점할 경우 또 다른 프로그램의 성능이 떨어진다는 단점. 
-  Virtualized Deployment(가상화 배포) / Hypervisor
    -  하이퍼바이저는 하나의 시스템 상에서 가상 컴퓨터를 여러 개 구동할 수 있도록 해 주는 중간 계층을 의미
    -  가상머신은 완전한 컴퓨터이고 가상머신에 일일이 운영체제를 설치해야 하기 때문에 컨테이너 중심의 배포(Container Deployment)보다는 무거운 편.
-  Container Development / Container Runtime
    -  Hypervisor -> Container Runtime / VM -> Container 으로 대체.
    -  프로그램 구동 시 OS 설치 불필요.
    -  하나의 OS 상에서 구동되지만 각각의 어플리케이션 간 CPU, 메모리 등의 자원을 독립적으로 사용할 수 있도록 할당하고 관리.
    -  OS에 문제를 일으킬 경우에는 OS에서 구동 중인 전체 컨테이너의 문제가 될 가능성이 있다.

## Docker란?
- Docker는 개발자가 컨테이너를 빌드, 배포 및 실행하도록 지원하는 상용 컨테이너화 플랫폼 및 런타임이다.</br>
  단일 API를 통한 간단한 명령과 자동화를 갖춘 클라이언트-서버 아키텍처를 사용한다.
- Docker는 컨테이너화된 애플리케이션을 패키징하고 배포하는 효율적인 방법을 제공하지만, Docker만으로는 대규모로 컨테이너를 실행하고 관리하기 어렵다.</br>
  여러 서버/클러스터에서 컨테이너를 조정 및 예약, 가동 중지 시간 없이 애플리케이션을 업그레이드 또는 배포, 컨테이너의 상태를 모니터링하는 등의 고려사항이 있다.</br>
  이와 같은 여러 문제를 해결하기 위해 컨테이너를 오케스트레이션하는 솔루션이 Kubernetes, Docker Swarm 등이 있다. 

## Kubernetes란?
- Kubernetes(K8s라고도 함)는 네트워크로 연결된 리소스의 클러스터 전체에서 컨테이너 런타임 시스템의 오케스트레이션에 널리 사용되는 오픈 소스 플랫폼이다.</br>
  Kubernetes는 Docker와 함께 또는 따로 사용할 수 있다.
- Kubernetes는 컨테이너 집합을 동일한 시스템에서 관리하는 그룹으로 번들링하여 네트워크 오버헤드를 줄이고 리소스 사용의 효율을 높인다.</br>
  컨테이너 집합의 예로는 앱 서버, Redis 캐시 및 SQL 데이터베이스가 있습니다. </br>
  Docker 컨테이너는 컨테이너당 하나의 프로세스다.

    - [Kubernetes vs Docker](https://www.atlassian.com/ko/microservices/microservices-architecture/kubernetes-vs-docker)


##

- 2024 쿠버네티스 표준 구성
  ![image](https://github.com/sysnet4admin/_Book_k8sInfra/blob/main/docs/k8s-stnd-arch/2024/img/2024-k8s-stnd-arch-thumbnail.png)



## 

https://www.samsungsds.com/kr/insights/220222_kubernetes1.html
https://www.samsungsds.com/kr/insights/220128-kubernetes.html?referrer=https://www.samsungsds.com/kr/insights/220222_kubernetes1.html
