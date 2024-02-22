![오늘의집](https://github.com/wwkang94/food-delivery/assets/25562517/e9f611b2-e52a-44e5-9efe-e2049cf1e980)

# Final Project - 오늘의집 MSA 구현

# Table of contents

- [서비스 시나리오](#서비스-시나리오)
  - 분석/설계
  - 이벤트 스토밍
  - MSA 아키텍처 구성도
- [구현](#구현)
  - 분산트랜잭션 & 보상처리 - Saga & Compensation
  - 단일 진입점 - Gateway
  - 분산 데이터 프로젝션 - CQRS 
- [운영](#운영)
  - 클라우드 배포
  - 컨테이너 자동확장 - HPA
  - 환경 분리 - ConfigMap
  - 셀프 힐링 - LivenessProbe
  - 서비스 메쉬 응용 - Mesh

---

# 서비스 시나리오

## 분석/설계
+ 요구사항
1. 고객이 가구를 선택하여 주문한다.
2. 고객이 결제한다.
3. 주문이 완료되면 주문 내역이 가구 브랜드에게 전달된다.
4. 가구 브랜드가 주문을 확인하여 배송 준비를 한다.
5. 고객이 주문을 취소할 수 있다.
6. 주문이 취소되면 배달이 취소된다.
7. 고객이 주문상태를 중간중간 조회한다.
8. 배송이 시작되면 고객에게 카카오톡 알림을 보낸다.

## 이벤트 스토밍
![eventstroming](https://github.com/wwkang94/food-delivery/assets/25562517/1a934cc5-1261-4dc8-acde-ca3f57862f2d)

## MSA 아키텍처 구성도
![architect](https://github.com/wwkang94/food-delivery/assets/25562517/b4dd5f8a-41b6-411a-8593-417bb8e96b38)


# 구현

## 분산트랜잭션 & 보상처리 - Saga & Compensation
```
http POST :8082/products productName=TV stock=100
http POST :8081/orders productId=[상품 Id] productName=TV qty=10 userId=1001
```
> 상품 등록 후 주문을 생성한다.

```
http GET :8081/orders/[주문번호]
```
> 주문 상태를 확인한다.

```
http GET :8083/deliveries
```
> 배송 조회 결과를 확인한다.

## 단일 진입점 - Gateway
Microservice들의 endpoint를 단일화한다.  
application.yaml 파일의 spring.cloud.gateway.routes 설정을 추가하여 라우팅 추가
![image](https://github.com/wwkang94/furniture-delivery/assets/25562517/0bafa7ac-6cf3-4902-a797-3d956407c38a)
![image](https://github.com/wwkang94/furniture-delivery/assets/25562517/79a4ffb5-2d4f-4a46-a264-4c78362df4f8)


## 분산 데이터 프로젝션 - CQRS 
데이터 저장소에 대한 읽기 및 업데이트 작업을 구분하는 패턴인 명령과 쿼리의 역할 분리를 의미  
Query 모델 설계  
![image](https://github.com/wwkang94/furniture-delivery/assets/25562517/44068872-75ff-4e20-964c-2cfd9713d7ab)


# 운영

## 클라우드 배포
주문/배송/상품 서비스 배포
![image](https://github.com/wwkang94/furniture-delivery/assets/25562517/a90c5ec7-5cca-4a01-a6ea-b19c811529c3)

## 컨테이너 자동확장 - HPA
![image](https://github.com/wwkang94/furniture-delivery/assets/25562517/016f2c3c-008b-4700-89b9-084d180bc5b5)
> [Auto Scale-out] CPU 사용율이 50% 이상이 될 경우 replica 를 10개까지 늘려준다.

## 환경 분리 - ConfigMap
![image](https://github.com/wwkang94/furniture-delivery/assets/25562517/b6d64ead-5e46-4926-b5e5-416710b18afd)
![image](https://github.com/wwkang94/furniture-delivery/assets/25562517/3bbd7b48-d0ff-40f0-a339-f47df45a5ffc)
> ConfigMap으로부터 ORDER_LOG_LEVEL 정보 설정

## 셀프 힐링 - LivenessProbe
문제가발생한컨테이너를종료하고, RestartPolicy (default: Always)에따라다시만들어지거나, 종료된상태로남는다.  
Pod의상태를체크하다가, Pod의상태가비정상인경우kubelet을통해서재시작한다.

```
apiVersion: v1
kind: Pod
metadata:
labels:
test: liveness
name: liveness-exec
spec:
containers:
-name: liveness
image: k8s.gcr.io/busybox
args:
-/bin/sh
--c
-touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600
livenessProbe:
exec:
command:
-cat
-/tmp/healthy
initialDelaySeconds: 3
periodSeconds: 5
```

![image](https://github.com/wwkang94/furniture-delivery/assets/25562517/13c09382-be4f-4ef6-8d4d-1b66437fab8e)

![image](https://github.com/wwkang94/furniture-delivery/assets/25562517/ef095d7b-a7e5-46e0-8a5d-188446b16c5e)

## 서비스 메쉬 응용 - Mesh
![image](https://github.com/wwkang94/furniture-delivery/assets/25562517/34c2ee8f-2208-4663-ac54-91846a50cc2d)
> istio 설치

![image](https://github.com/wwkang94/furniture-delivery/assets/25562517/b3412968-ab7e-46bc-8003-e0d3e4eee237)
> Kiali를 통한 서비스 메쉬 모니터링
