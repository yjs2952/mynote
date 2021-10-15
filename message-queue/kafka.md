# 카프카

## 카프카 실행
### 카프카 브로커 실행 옵션 설정
- ```broker.id``` 
    + 브로커 식별 번호. 클러스터 구축 시 브로커들을 구분하기 위한 번호 
- ```listeners``` 
    + 카프카 브로커가 통신을 위해 열어둘 인터페이스 IP, PORT, 프로토콜 지정 (default : 모든 ip, port 접속 가능)
    + ex) PLAINTEXT://127.0.0.1:9092
- advertised.listeners 
    + 카프카 클라이언트 또는 커맨드 라인 툴에서 접속하기 위한 IP, PORT
    + ex) AWS 인스턴스의 public ip
- SASL_PLAINTEXT,SASL_SSL:SASL_SSL
    + SASL_SSL,SASL_PLAIN 보안 설정 시 프로토콜 매핑을 위한 설정
- num.network.threads
    + 네트워크를 통한 처리를 사용할 때 네트워크 스레드 개수 설정
- num.io.threads
    + 카프카 브로커 내부에서 사용할 스레드 개수 설정
- ```log.dirs=/tmp/kafka-logs```
    + 통신을 통해 가져온 데이터를 파일로 저장할 디렉토리
    + 경로에 디렉토리가 없는 경우 오류 발생
- ```num.partitions```
    + 파티션 개수를 명시하지 않고 토픽을 생성할 때 기본 설정되는 파티션 수
    + 파티션 양이 많아지면 병렬처리 데이터양이 늘어남
- log.retention.hours
    + 카프카 브로커가 저장한 파일이 삭제되기까지 걸리는 시간
    + ```log.retention.ms``` 옵션 사용을 권장
    + ```log.retention.ms=-1``` 로 설정하면 파일은 영구적으로 삭제되지 않음
- log.segment.byte
    + 카프카 브로커가 저장할 파일의 최대 크기
    + 데이터양이 이 크기를 넘을 경우 새로운 파일이 생성
- log.retention.check.interval.ms
    + 카프카 브로커가 저장한 파일을 삭제하기 위해 체크하는 시간 간격
- ```zookeeper.connect```
    + 카프카 브로커와 연동할 주키퍼의 IP, PORT 지정
- zookeeper.connection.timeout.ms
    + 주키퍼 세션 타임아웃 시간을 지정

#### 주키퍼 실행
- 주키퍼는 카프카의 클러스터 설정 리더 정보, 컨트롤러 정보를 담고 있어 카프카 실행에 필수적인 애플리케이션이다
- ```bin/zookeeper-server-start.sh -daemon config/zookeeper.properties```
- ```jps -vm``` 명령어
    + jps : JVM 프로세스 상태를 확인하는 도구 (JVM 위에서 돌아가는 주키퍼의 프로세스 확인용)
        * -v : JVM 에 전달된 인자 (힙메모리, log4j 설정 등)를 확인할 수 있음
        * -m : main 메서드에 전달된 인자를 확인할 수 있음

#### 카프카 브로커 실행
- ```bin/kafka-server-start.sh -daemon config/server.properties```
- ```tail -f logs/server.log```
    + 로그 확인

## 카프카 커맨드 라인 툴
### kafka-topics.sh
- 카프카 토픽과 관련됨 명령을 사용할 수 있음
- 토픽은 카프카에서 데이터를 구분하는 가장 기본적인 개념 
- RDBMS 의 테이블과 유사

#### 토픽 생성
```
bin/kafka-topics.sh \
--create \
--bootstrap-server localhost:9092 \
--partitions 3 \
--replication-factor 1 \
--config retention.ms=172800000 \
--topic hello.kafka 
```
- ```--create```
    + 토픽을 생성하는 명령어라고 명시
- ```--bootstrap-server``` (필수)
    + 토픽을 생성할 카프카 클러스터를 구성하는 브로커들의 IP, PORT (일반적으로는 3개 이상)
- ```--topic``` (필수)
    + 토픽 이름 작성
    + 작성시 내부 데이터가 무엇인지 유추할 수 있는 이름으로 지정하는 것을 권장
- ```--partitions```
    + 파티션 개수 지정 (최소 1개 이상)
    + 설정하지 않으면 브로커 설정파일 **config/server.properties** 파일의 ```num.partitions``` 값에 따라 생성됨
- ```--replication-factor```
    + 토픽의 파티션을 복제할 복제 개수
    + 1로 설정 시 복제하지 않음. 2로 설정 시 1개의 복제본 사용
    + 파티션의 데이터는 브로커마다 저장
    + 한 브로커에서 장애가 나도 다른 브로커에서 저장된 데이터를 사용하여 안전하게 지속적으로 처리할 수 있음
    + 최소 설정은 1, **최대 설정은 카프카 브로커의 개수** (브로커가 3개인 경우 3으로 설정)
    + 설정하지 않으면 브로커 설정파일의 ```default.replication.factor``` 값을 따름
- ```--config```
    + kafka-topics.sh 명령에 포함되지 않은 추가 설정 가능
    + 예제의 ```--config retention.ms=172800000``` 옵션은 토픽의 데이터 유지 기간을 추가

#### 토픽 목록 조회
```
bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```
- ```--list``` 옵션을 사용하여 카프카 클러스터에 생성된 토픽들의 이름을 확인할 수 있음

#### 토픽 상세 조회
```
bin/kafka-topics.sh --bootstrap-server localhost:9092 --describe --topic hello.kafka

Topic: hello.kafka PartitionCount: 3 ReplicationFactor: 1 Configs: segment.bytes=~~~~,retention.ms=172800000

Topic: hello.kafka   Partition: 0   Leader: 0   Replicas: 0   Isr: 0
Topic: hello.kafka   Partition: 1   Leader: 0   Replicas: 0   Isr: 0
Topic: hello.kafka   Partition: 2   Leader: 0   Replicas: 0   Isr: 0
```
- ```--describe``` 옵션으로 토픽의 상태를 확인
- 파티션 개수(PartitionCount), 복제된 파티션이 위치한 브로커 번호(Leader), 기타 설정등을 출력
- 여러 브로커로 카프카 클러스터를 운영하는 경우 리더 파티션이 일부 브로커에 몰리는 현상이 있을 수 있음 이를 확인하기 위한 옵션
    + 리더 파티션이 몰리는 경우 카프카 클러스터 부하가 특정 브로커로 몰릴 수 있어 네트워크 대역 문제가 생길 수 있음

#### 토픽 옵션 수정
- 토픽 옵션을 변경하기 위해서 ```kafka-topics.sh``` 또는 ```kafka-configs.sh``` 를 사용해야 함
- ```--describe``` 옵션으로 변경된 결과 확인 가능
##### kafka-topics.sh
```
bin/kafka-topics.sh --bootstrap-server localhost:9092 \
--tipics hello.kafka \
--alter \
--partitions 4
```
- 파티션 개수를 변경하기 위해 ```kafka-topics.sh``` 사용
- ```alter``` 옵션과 ```partitions``` 옵션으로 파티션 개수 변경 가능
- **주의)** **파티션 개수**는 늘릴 수는 있지만 **줄일 수 없음**
##### kafka-configs.sh
*dynamic topic config 로 정의되는 설정은 kafka-configs.sh 로 수정 가능 (log.segment.bytes, log.retention.ms 등)*
```
bin/kafka-configs.sh --bootstrap-server localhost:9092 \
--entity-type topics \
--entity-name hello.kafka \
--alter --add-config retention.ms=86400000
```
- 토픽 삭제 정책인 리텐션 기간을 변경
- retention.ms 를 수정하기 위해 --alter, --add-config 옵션 사용
    + --add-config 사용 시 이미 존재하는 설정값은 변경하고 존재하지 않는 설정값은 새로 추가함

### kafka-console-producer.sh
```
bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic hello.kafka
```
- 토픽에 데이터를 넣을 수 있음
- 데이터는 ```레코드``` 라는 단위로 구분되고 key, value 로 이루어져 있음
- key 를 입력하지 않고 메시지만 보낼 시 key 는 null 로 기본 설정되어 브로커로 전송됨
- ```kafka-console-producer.sh``` 로 전송되는 레코드 값은 UTF-8 을 기반으로 Byte로 변환됨
    + 변환된 Byte는 ```ByteArraySerializer``` 로만 직렬화하여 전송할 수 있음

#### 메시지 키를 가지는 레코드 전송
```
bin/kafka-console-producer.sh --bootstrap-server localhost:9092 \
--topic hello.kafka
--property "parse.key=true" \
--property "key.seperator=:"

>key1:no1
>key2:no2
>key3:no3
```
- ```parse.key``` 를 true 로 설정시 메시지키를 추가할 수 있음
- ```key.seperator```  로 메시지 키와 값을 구분하는 구분자를 선언
    + default 는 ```\t``` 탭이다
- 메시지 키가 null 인 경우 프로듀서가 파티션으로 전송할 때 레코드 배치 단위로 라운드 로빈으로 전송
- 메시지 키가 존재하는 경우 키의 해시값을 작성하여 존재하는 파티션 중 하나에 할당됨
    + 동일한 키인 경우 동일한 파티션으로 전송됨 (기본 파티셔너인 경우)
    + 단 **파티션 개수가 늘어나는 경우 같은 파티션으로 전송된다는 보장이 없음**
        * 키의 일관성을 보장해야 한다면 커스텀 파티셔너를 만들어야함

### kafka-console-consumer.sh
```
bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 \
--topic hello.kafka \
--property print.key=true \
--property key.seperator="-" \
--group hello-group \
--from-beginning
```
- 토픽으로 전송한 데이터를 확인할 수 있음
- 필수옵션
    + ```--bootstrap-server``` : 카프카 클러스터 정보
    + ```--topic```     : 토픽 이름
- ```print.key``` 를 true 로 설정 시 메시지키를 확인할 수 있음
- ```key.seperator``` 로 키 값을 구분 기본 설정은 ```\t```
- ```--group``` 옵션으로 컨슈머 그룹 생성
    + 한 개 이상의 컨슈머로 이루어져 있음
    + 컨슈머 그룹을 통해 가져간 토픽의 메시지는 가져간 메시지에 대해 커밋(commit)을 함
    + 커밋은 컨슈머가 특정 레코드까지 처리를 완료했다는 레코드의 오프셋 번호를 카프카 브로커에 저장하는 것
        * 커밋 정보는 _consumer_offsets 이름의 내부 토픽에 저장됨
- ```--from-beginning``` 옵션으로 토픽에 저장된 가장 처음 데이터 부터 출력 가능
- kafka-console-producer.sh 로 전송한 데이터의 순서가 출력되는 순서와 다름
    + 토픽의 모든 파티션으로 부터 동일한 중요도로 데이터를 가져오기 때문
    + 순서를 보장해야 한다면 파티션을 1개로 구성하면 됨

### kafka-delete-records.sh
이미 적재된 토픽의 데이터를 지우는 명령어로 가장 오래된(가장 낮은 오프셋) 데이터부터 특정 시점까지 삭제 가능

```
vi delete-topic.json

bin/kafka-delete-records.sh --bootstrap-server localhost:9092 \
--offset-json-file delete-topic.json
```
- 삭제 하고자 하는 데이터에 대한 정보를 파일로 저장해서 사용해야 함
    + delete-topic.json 파일로 새성
    + 파일에 삭제할 **토픽, 파티션, 오프셋** 정보가 있어야 함
- 삭제할 토픽이 있는 클러스터의 호스트, 포트를 --bootstrap-server 옵션으로 입력
- delete-topic.json 을 ```--offset-json-file``` 옵션으로 입력하면 파일을 읽고 데이터를 삭제
- **주의)** 토픽의 특정 레코드 하나만 삭제되는 것이 아닌 **파티션에 존재하는 가장 오래된 오프셋 부터 지정한 오프셋까지 삭제**
    + 파티션에 저장된 특정 데이터만 삭제할 수 없음

## 카프카 기본 개념

### 카프카 브로커, 클러스터, 주키퍼

#### 데이터 저장, 전송
- 카프카는 DB나 메모리에 저장하지 않으며 캐시 메모리를 구현하여 사용하지 않고 파일 시스템에 저장
- **페이지 캐싱**을 이용하여 디스크 입출력 속도를 높여서 파일 시스템 저장으로 인한 속도 문제를 해결
    + 페이지 캐시는 OS 에서 파일 입출력 성능 향상을 위해 만들어놓은 메모리 영역
    + 한번 읽은 파일의 내용을 페이지 캐시에 저장하여 동일한 요청시 메모리에서 읽음
    + 힙 메모리를 크게 지정할 필요가 없음
    + 브로커가 페이지 캐시를 사용하지 않는다면?
        * 카프카에서 캐시를 직접 구현해야 함
        * 지속적인 데이터 변경으로 JVM에서 돌아가는 브로커에서 가비지 컬렉션이 자주 일어나 속도저하가 발생

#### 데이터 복제, 싱크
- 데이터 복제는 카프카를 장애 허용 시스템으로 동작하게 함
- 클러스터로 묶인 브로커 중 일부에 장애가 발생해도 데이터를 유실하지 않음
- 데이터 복제는 파티션 단위 
    + 토픽 생성 시 파티션 **복제 개수**(replication factor)도 같이 설정
    + 복제 개수 설정 **최소값은 1(복제 없음)**, **최대값은 브로커 개수**
- 프로듀서, 컨슈머와 직접 통신하는 파티션을 **리더**, 나머지 복제 데이터를 가진 파티션을 **팔로워**라 부름
    + 리더 파티션의 오프셋과 차이가 나는 경우 팔로워 파티션에서 복제가 이루어짐
    + 리더 파티션이 포함된 브로커에서 장애가 발생 시 팔로워를 가진 파티션 중 하나가 리더 파티션이 됨

#### 컨트롤러
- 다른 브로커의 상태를 체크하고 브로커가 클러스터에서 빠지는 경우 해당 브로커의 리더 파티션을 재분배
- 클러스터의 여러 브로커 중 한 대가 컨트롤러 역할을 함
- 컨트롤러 역할의 브로커에 장애가 발생하는 경우 다른 브로커가 컨트롤러 역할을 하게 됨

#### 데이터 삭제
- 카프카는 컨슈머가 데이터를 가져가도 토픽의 데이터는 삭제되지 않음
    + 컨슈머 혹은 프로듀서가 데이터 삭제 요청을 할 수 없음
- 데이터 삭제는 로그 세그먼트(log segment) 라는 단위로 삭제
    + 세그먼트에는 다수의 데이터가 들어있기 때문에 DB 처럼 특정 데이터만 삭제할 수 없음
- 세그먼트 파일이 닫히는 기본 값은 1GB
- 닫힌 세그먼트 파일 삭제 주기는 `log.segment.bytes` 또는 `log.segment.ms` 로 설정할 수 있음
- 닫힌 세그먼트 파일 체크 주기는 `log.retention.check.interval.ms` 로 설정할 수 있음

#### 컨슈머 오프셋 저장
- 컨슈머 그룹은 토픽이 특정 파티션으로부터 어느 레코드까지 가져갔는지 확인하기 위해 오프셋을 커밋함
- 커밋한 오프셋은 _consumer_offsets 토픽에 저장
    + 이 토픽에 저장된 오프셋을 토대로 컨슈머 그룹은 다음 레코드를 가져가서 처리

#### 코디네이터(coordinator)
- 컨슈머 그룹의 상태를 체크하고 파티션을 컨슈머와 매칭되도록 분배하는 역할
- 클러스터의 다수 브로커 중 한대는 코디네이터 역할 수행
- 컨슈머가 그룹에서 빠질 경우 매칭되지 않은 파티션을 정상 동작하는 컨슈머에 할당
    + 파티션을 컨슈머로 재할당하는 과정을 **리밸런스**(rebalance) 라고 부름

### 토픽, 파티션

### 레코드
- 타임스탬프, 메시지 키, 메시지 값, 오프셋으로 구성
- 수정할 수는 없고 삭제만 가능
- 메시지 키는 메시지 값을 순서대로 처리하거나 메시지 값의 종류를 나타내기 위해 사용
- 메시지 키를 사용하면 프로듀서가 토픽에 레코드 전송 시 메시지 키의 해시값을 토대로 파티션 지정
    + 동일한 메시지 키라면 동일한 파티션에 전송됨
    + 어느 파티션으로 지정될지 알 수 없음
    + 파티션 개수 변경 시 메시지 키와 파티션 매칭이 달라짐으로 주의
- 메시지 키 미설정시 null 로 지정
- 직렬화, 역직렬화를 동일한 형태로 해야함

## 카프카 클라이언트

### 프로듀서 API

#### 프로듀서 중요 개념
> 프로듀서는 카프카 브로커로 데이터를 전송할 때 내부적으로 파티셔너, 배치 생성 단계를 거친다.   
> KafkaProducr 인스턴스가 send() 메서드를 호출하면 ProducerRecord 는 파티셔너에서 토픽의 어느 파티션으로 전송될지 정해진다.
- 파티셔너를 따로지정하지 않으면 기본적으로 DefaultPartitioner 로 설정
- 파티셔너에 의해 구분된 레코드는 데이터를 정송하기 전 Accumulator 에 데이터를 버퍼로 쌓아두고 발송
    + 버퍼로 쌓인 데이터는 배치로 묶어서 전송
- 압축 옵션을 제공하여 브로커 전송시 압축 방식 지정 가능
    + gzip, snappy, lz4, zstd 지원
    + 네트워크 처리량에 이득을 볼 수 있지만 CPU, 메모리 리소스 사용으로 인한 부하가 생길 수 있음
    + 프로듀서에서 압축하여 컨슈머에서 압축해제(양쪽 모두 서버자원을 사용하므로 주의)

##### 파티셔너
- 레코드와 파티션을 매핑시켜주는 장치
- **UniformStickyPartitioner**
    + 카프카 클라이언트 라이브러리 2.5.0 버전의 기본 파티셔너
    + 메시지 키가 있는 경우 키의 해시값과 파티션을 매칭하여 데이터를 전송
    + 키가 없는 경우 파티션에 최대한 동일하게 분배하는 로직 수행
    + 프로듀서 동작에 특화되어 **높은 처리량**과 **낮은 리소스 사용률**을 가짐
    + `Accumulator` 에 레코드가 **배치로 묶이면 동일한 파티션에 전송**
- RoundRobinPartitioner
    + 카프카 클라이언트 2.4.0 이전 버전의 기본 파티셔너
    + 메시지 키가 있는 경우 위와 동일
    + 키가 없는 경우 따로 로직 없음
    + 레코드가 들어오는 순서대로 파티션을 순회하면서 전송하기 때문에 배치로 묶이는 빈도가 적음
- 커스텀 파티셔너
    + 사용자 지정 파티셔너 생성을 위한 Partitioner 인터페이스 제공
    + 메시지키 또는 메시지 값에 따른 파티션 지정 로직 구현
    + 파티션 개수가 변경으로 파티션 분배가 동일하게 적용되지 않는 경우 사용
    + 파티셔너를 통해 파티션이 지정된 데이터는 Accumulator 에 버퍼로 쌓임
        + sender 쓰레드는 Accumulator 에 쌓인 데이터를 카프카 브로커에 전송

#### 프로듀서 주요 옵션
- 필수 옵션
    + bootstrap.servers
        * 프로듀서가 데이터를 전송할 브로커 IP, PORT 정보 (복수 지정 가능)
    + key.serializer
        * 메시지 키 직렬화
    + value.serialzer
        * 메시지 값 직렬화
- 선택 옵션
    + acks
    + buffer.memory 
    + retries
    + batch.size
    + linger.ms
    + partitioner.class
    + enable.idempotence
    + transactional.id

### 컨슈머 API
#### 컨슈머 주요 개념
> 토픽의 파티션으로부터 데이터를 가져가기 위해 컨슈머를 운영하는 방법은 크게 두 가지이다.
> 첫 번째는 1개 이상의 컨슈머 그룹을 운영하는 것, 두 번째는 토픽의 특정 파티션만 구독하는 컨슈머를 운영하는 것.

- 컨슈머 그룹 운영
    + 각 컨슈머 그룹으로 부터 격리된 환경에서 안전하게 운영가능
    + 토픽의 한 개 이상 파티션에 할당
    + 1개의 파티션은 최대 1개의 컨슈머에 할당 가능
    + 1개의 컨슈머는 여러 파티션에 할당 가능
    + 컨슈머는 파티션 개수보다 같거나 적어야함
        * 컨슈머가 파티션 보다 많을 경우 남는 컨슈머는 파티션 할당 못받음
        * 쓰레드만 차지하고 데이터 처리 안함
- 컨슈머 그룹내에서 컨슈머에 장애 발생시 과정
    + **리밸런싱** : 장애가 발생한 컨슈머에 할당된 파티션이 다른 컨슈머에 할당됨
    + 리밸런싱은 컨슈머가 추가되거나 제외되는 경우 발생
        * 컨슈머에 장애가 생길 경우 컨슈머 그룹에서 해당 컨슈머를 제외
    + 그룹 조정자(group coordinator) 에 의해 리밸런싱이 일어남
        * 컨슈머가 추가되고 삭제되는 것을 감
        * 그룹조정자는 브로커 중 한대가 담당

##### 파티션 오프셋 커밋
- 컨슈머는 브로커로부터 데이터를 어디까지 가져갔는지 커밋을 통해 기록
    + `_consumer_offsets` 이라는 내부 토픽에 저장
- 컨슈머 오류로 어디까지 읽었는지 오프셋 커밋을 기록하지 못했다면 데이터 처리 중복이 발생할 수 있으므로 오프셋 커밋 검증 필요
- **비명시적 커밋**
    + `enable.auto.commit=true` 설정으로 poll() 메서드 실행될 때 일정 간격마다 오프셋을 커밋 (default) 
    + poll() 메서드가 `auto.commit.interval.ms` 에 설정된 값이 지났을 때 그 시점까지 읽을 레코드의 오프셋을 커밋
    + poll() 메서드 호출 이후 리밸런싱 또는 컨슈머 강제종료 등으로 데이터 중복 또는 유실이 발생할 수 있음
- **명시적 커밋**
    + poll() 메서드 호출 후 반환받은 데이터 처리 왆료되고 commitSync() 메소드 호출
    + commitSync() 메서드는 poll() 메서드로 반환된 레코드의 가장 마지막 오프셋을 기준으로 커밋
        * 브로커에 커밋 요청을 하고 처리될때까지 기다림
        * 커밋 처리시간이 길어지면 컨슈머 데이터 처리량에 영향이 생길 수 있음
    + commitAsync() 메서드를 이용하여 커밋 요청 전송 후 응답이 올때까지 데이터 처리를 수행할 수 있음
        * 비동기 커밋을 커밋 실패할 경우 현재 처리 중인 데이터의 순서를 보장하지 않음

##### 컨슈머 내부 구조
- poll() 메소드 호출 시 레코드를 반환하지만 실제 메서드 호출 시점에 가져오는 건 아님
- 컨슈머 애플리케이션 실행 시 내부에서 Fetcher 인스턴스가 생성되어 poll() 메서드 호출 전 미리 레코드를 내부 큐로 가져옴
- 이후 poll() 메서드 호출 시 내부 큐의 레코드들을 반환받아 처리를 수행

#### 컨슈머 주요 옵션
- 필수 옵션
    + bootstrap.servers
        * 프로듀서가 데이터를 전송할 대상 카프카 클러스터에 속한 브로커 정보
        * 브로커 호스트:포트 (여러개 작성 가능)
    + key.deserializer
        * 레코드 메시지 키를 역직렬화할 클래스 지정
    + value.deserializer
        * 레코드 메시지 값을 역직렬화할 클래스 지정
- 선택 옵션
    + group.id
        * 컨슈머 그룹 아이디 지정
        * subscribe() 메서드로 토픽 구독 시 반드시 필요
    + auto.offset.reset
        * 컨슈머 그룹이 특정 파티션을 읽을 때 저장된 오프셋이 없는 경우 어디부터 읽을지 결정
        * latest(가장 최근, default), earliest(가장 오래전), none(커밋 기록 탐색 없을 시 오류)
    + enable.auto.commit
        * 자동 커밋으로 할지 수동 커밋으로 할지 선택. default true
    + auto.commit.interval.ms
        * 자동 커밋일 경우 (enable.auto.commit=true) 오프셋 커밋 간격 지정
        * default 5000ms
    + max.poll.records
        * poll() 메서드를 통해 반환되는 레코드 수 지정
        * default 500
    + session.timeout.ms
        * 컨슈머가 브로커와 연결이 끊기는 최대 시간
        * 시간 내에 하트비트를 전송하지 않으면 브로커에서 리밸런싱을 실헹
        * 일반적으로 하트비트 시간의 3배 설정, default 10000ms
    + heartbeat.interval.ms
        * 하트비트를 전송하는 시간 간격 default 3000ms
    + max.poll.interval.ms
        * poll() 메서드 호출하는 간격의 최대 시간
        * poll() 메서드 호출 이후 데이터 처리에 시간이 너무 오래 걸리는 경우 리밸런싱 시작
        * default 300000ms (5분)
    + isolation.level
        * 트랜잭션 프로듀서가 레코드를 트랜잭션 단위로 보낼 경우 사용
        * read_committed
            - 커밋 완료된 레코드만 읽음
        * read_uncommitted
            - 커밋 여부 관계없이 파티션의 모든 레코드를 읽음