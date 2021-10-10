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
- ```bin/zookeeper-server.sh -daemon config/zookeeper.properties```
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
bin/kafka-console-consumer.sh --boostrap-server localhost:9092 \
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
