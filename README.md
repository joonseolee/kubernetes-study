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

## step 8

인그레스를 설명하면서 `vegeta` 같은 라이브러리로 스트레스 테스트를 하는걸 설명하고있는데  
그외에는 책이 오래되서 그런지 제대로 실행되는게 없다..ㅠ 쿠버네티스 공식 예제를 참고하는게 좋을듯  

## step 9

레이블을 기준으로 소팅하거나 할때 편하다고 함.

```bash
1223@vv ~/D/g/kubernetes-tutorial (main)> kubectl get pods -l app=nginx
NAME                             READY   STATUS    RESTARTS   AGE
nginx-label01-5688c56779-ztjfh   1/1     Running   0          2m16s
nginx-label02-68bb4556f9-pvppq   1/1     Running   0          2m16s
nginx-label03-76cddc5444-cxbpq   1/1     Running   0          2m16s
nginx-label04-798fbc5748-hhqml   1/1     Running   0          2m16s
1223@vv ~/D/g/kubernetes-tutorial (main)> kubectl get pods -l environment=develop,release=stable
NAME                             READY   STATUS    RESTARTS   AGE
nginx-label03-76cddc5444-cxbpq   1/1     Running   0          2m38s
1004883@vv ~/D/g/kubernetes-tutorial (main)> kubectl get pods -l "app=nginx,environment notin (develop)"
NAME                             READY   STATUS    RESTARTS   AGE
nginx-label02-68bb4556f9-pvppq   1/1     Running   0          3m18s
nginx-label04-798fbc5748-hhqml   1/1     Running   0          3m18s
1223@vv ~/D/g/kubernetes-tutorial (main)> kubectl get pods -l release!=stable
NAME                             READY   STATUS    RESTARTS   AGE
nginx-label01-5688c56779-ztjfh   1/1     Running   0          3m44s
nginx-label02-68bb4556f9-pvppq   1/1     Running   0          3m44s
```

아노테이션은 이유를 기입할때 매번 사용하기 좋을듯싶다.  
가령 `.metadata.annotations` 에 `manager`, `contact`, `release-version` 이런것들을 넣어두면 좋을듯  


deployment-canary-v1, deployment-canary-v2 를 올리고 service-canary 로 연동시켜주면 같은 deployment 로 사용이 되게 된다.

컨테이너로 컨피그맵을 사용할수있는데 볼륨형태로도 사용가능하다.  
`.template.spec.volumes` 을 참고하면됨  

```yaml
template:
  spec:
    containers:
    - name: testapp
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
    volumes:
    - name: config-volume
      configMap:
        name: config-dev
```

브라우저에상에서 검색할때는 쿼리 파라미터를 사용해야한다.    
`localhost:30800/volume-config?path=/etc/config/DB_USER`

## step 11

### 명령어로 시크릿 생성하는 방법  

```
echo -n "username" > ./username.txt
echo -n "password" > ./password.txt
kubectl create secret generic user-pass-secret --from-file=./username.txt --from-file=./password.txt

# kubectl 로 확인도 가능하다
kubectl get secret user-pass-secret -o yaml

echo cGFz... | base64 --decode
```

### 템플릿으로 시크릿 생성

이미 폴더에 추가해두었는데 다만 직접 base64 로 인코딩해서 넣어줘야하기때문에 귀찮음.  
```
echo "username" | base64
```

### 볼륨형식으로 팟에 시크릿 제공 

이것도 가능..

### 프라이빗 컨테이너 이밎를 가져올때 시크릿을 사용

내부 메이븐을 쓰는것처럼 주소를 지정하여 사용가능.  
--docker-server 키워드 잊지말기.  

### 시크릿으로 TLS 인증서를 저장해 사용하기

왜 쓰는건가싶었는데 이게 가장 큰 요인인듯..  
이 시크릿을 인그레스와 연결해서 사용 

시크릿들은 etcd 에 저장되는데 접근제한을 걸어둬야하고 각 시크릿당 최대 용량은 1MB 로 정해져있다.  

## step 12

레이블 리스트를 확인  
kubectl get nodes --show-labels  

특정 키값 추가  
kubectl label nodes minikube disktype=ssd  

이후에 `.spec.nodeSelector.disktype=hdd` 를 넣어주게 되면 실행이 되지않는다.  
왜냐하면 레이블설정이 다르기때문에 실행할 노드가 없다.  

matchExpressions.operator 의 연산자로는 아래와 같다.  
```
In, NotIn, Exists, DoesNotExist, Gt, Lt
```

테인트를 설정하여 테인트를 설정한 노드에는 팟들을 스케줄링하지 않는다.  
삭제하고싶을땐 위에서 마이너스만 추가해주면된다.  
```
kubectl taint nodes minikube key01=value01:NoSchedule
```

이후 평상시처럼 실행시킬때 펜딩만 되고 러닝이 되지않는데 그럴땐 tolerations 값을 넣어줘야한다.  
```
tolerations:
- key: "key01"
  operator: "Equal"
  value: "value01"
  effect: "NoSchedule"
```

클러스터를 관리하는걸로 cordon 과 drain 이 있는데  
cordon 을 설정하면 스케줄링이 정지되어 펜딩상태로 변하게 되고 스케줄링을 할려면 다시 Uncordon 을 하면된다.  
특정 노드에 있는 팟들을 다른 노드로 이동시키는게 drain 이다.  
스케줄링을 실행하지않도록 설정후 기존에 있던 노드들에 있는 팟들을 삭제한다.  

노드에 데몬셋이 있을경우 삭제가 되지않는데 그건 계속 살아나기때문!  
무시하기위해선 --ignore-daemonset=true 값을 지정해준다.  
