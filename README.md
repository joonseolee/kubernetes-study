# kubernetes-tutorial

문서 및 도서를 활용하여 쓸만한 쿡북같은것들을 업로드하는 레파지토리

## step 6

### update 

컨테이너 이미지 설정 정보를 업데이트할때 할수있는 방법은 3가지
```bash
// Kubectl set 명령어로 직접 이미지 설정
kubectl set image deployment/nginx-deployment nginx-deployment=nginx:1.9.1

// Kubectl edit 로 이미지명 변경
kubectl edit deploy nginx-deployment

// 템플릿의 이미지정보를 수정후 다시 kubectl apply 하기


```

### rollback 

```bash
// 히스토리 확인 
kubectl rollout history deploy nginx-deployment

// 각 리비전별 확인
kubectl rollout history deploy nginx-deployment -revision=3

// 이전 리비전으로 이동하고자 할때
kubectl rollout undo deploy nginx-deployment

// 특정 리비전으로 가고자 할때
kubectl rollout undo deploy nginx-deployment --to-revision=3
```

히스토리를 남기기위해서는 `.metadata.annotations` 에 값을 넣어줘야한다.  
```bash
kubernetes.io/change-cause: hello-world~!
```

만약에 팟의 갯수를 현서비스상태에서 늘리거나 줄이고싶을때
```bash
kubectl scale deploy nginx-deployment --replica=5
```

이미지 버전등을 수정했을때 바로 반영되지않도록 막을수도 있는데  
사실 많이 쓰일지 좀 의문임...그 이후에 바로 재개하는방법도 있다.  
```bash
// 일시정지 서비스는 계속 가동중!
kubectl rollout pause deployment/nginx-deployment

// 재개 
kubectl rollout resume deploy/nginx-deployment
```

### daemonSet

RollingUpdate 는 기본값인데 1.5 버전이하에서는 onDelete 가 기본값.  
.spec.updateStrategy.rollingUpdate.maxUnavailable: 몇개의 팟을 삭제할것인지 설정 (기본값 1)
.spec.minReadySeconds: 한번에 교체하는 팟 갯수 조정
.spec.minReadySeconds: 새로 실행되는 팟의 준비 최소 시간

terminationGracePeriodSeconds 는 하던작업을 마무리하고 종료하는데 시간

* 기본적으로 순서대로 생성되고 그 반대순으로 삭제되지만 순서를 삭제할수도있다.
    * .spec.podManagementPolicy: Parallel


### cronJob

* .spec.concurrencyPolicy 잡의 동시성을 관리 
    * 기본값은 Allow, Forbid 로 하면 잡을 동시에 실행시키지않는다.
    * Replace 이전에 실행중이던 잡을 새로운 잡으로 대체 