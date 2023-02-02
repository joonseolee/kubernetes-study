# k8s-pattern

## chapter 3

배포 전략에는 롤링배포, 고정배포, 블루-그린, 카나리아 릴리스가 있다.  

* 롤링배포
  * n 개씩 올려가면서 n 개씩 내리고 무중단배포를 지원한다.  
* 고정배포
  * 우선적으로 현재 버전의 모든 컨테이너를 죽이고 신규 컨테이너를 재실행하기때문에 다운타임이 존재할수있다.  
* 블루-그린
  * 신버전인 컨테이너들을 모두 올리고나서 트래픽을 기존에서 신규로 전환다. 보통 Service Mesh, Knative 같은 확장 서비스를 사용하는데 그렇지않다면 직접 수동을 작업해야한다.  
* 카나리아
  * 일부 유저들만 업데이트된 버전을 사용하게하여 운영에 새 버전 도입할때의 위험을 줄여준다. 이후 인스턴스를 모두 새로운 버전으로 교체한다.  

라이브니스랑 레디니스 점검은 동일한 체크를 수행하지만 레디니스는 컨테이너에게 시작할수있는 시간을 줄수있으며 점검을 통과해야만 디플로이먼트가 성공한것으로 간주한다.  

## chapter 5

쿠버네티스가 컨테이너를 멈추기로 결정할때마다 컨테이너는 Sigterm 신호를 수신한다.  
Sigkill 신호를 보내기전에 컨테이너가 깔끔히 없앨수있도록 미리 보내보는것.  
만약 종료되지않으면 Sigkill 을 보내고 되고 그 텀을 조정할수도있다. `.spec.terminationGracePeriodSeconds` 필드 참조  

## chapter 6

affinity 에 대한 설명이 많이 나온다.  

* node-affinity
  * 아래 내용을 참고하면 `confidential: high` 레이블을 가진 파드가 실행중인 노드는 `security-zone` 레이블이 있어야한다.
```yaml
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        - topologyKey: security-zone
          labelSelector:
            matchLabels:
              confidential: high 
```

  * 여기선 팟이 배치가 되서는 안되는 규칙인데 `confidential: none` 레이블을 가진 팟이 실행중인 노드에 배치되어서는 아니된다!!! 라는 뜻 

```yaml
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
        - podAffinityTerm:
            topologyKey: kubernetes.io/hostname
            labelSelector:
              matchLabels:
                confidential: none
          weight: 100
```

테인트와 톨러레이션은 팟을 스케줄링할지 안할지 제어가 가능한데 노드에 스케줄링을 허용하는 opt-in 이라고 말하며 affinity 실행할 노드를 명시적으로 선택하여 선택되지 않은 노드를 모두 제외하는 opt-out 이라고 말함.  
테인트와 톨러레이션이라는 개념이 어렵기때문에 좀더 공부필요.  