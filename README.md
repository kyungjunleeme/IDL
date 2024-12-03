# IDL



### 1. Backward Compatibility란?

Backward Compatibility는 이전 버전의 클라이언트(또는 서비스)가 새로운 버전의 메시지를 이해할 수 있도록 설계하는 것을 의미합니다.
예를 들어, Protocol Buffers 메시지에서 새로운 필드가 추가되거나, 기존 필드가 변경되었을 때도 이전 버전의 클라이언트가 여전히 메시지를 처리할 수 있어야 합니다.


### 2. 예제: IDL 서버 없이 발생할 수 있는 문제
초기 상태: 메시지 정의
처음에는 다음과 같은 User 메시지가 있다고 가정합니다.

Proto 파일 (v1):

```proto
syntax = "proto3";

message User {
    int32 id = 1;
    string name = 2;
}
```
python에서 이 메시지를 처리하는 코드:

```python
from user_pb2 import User

# v1 메시지 생성
user = User(id=1, name="John Doe")

# 직렬화
serialized_data = user.SerializeToString()

# 역직렬화
received_user = User()
received_user.ParseFromString(serialized_data)
print(received_user)
```
출력:
```sh
id: 1
name: "John Doe"

```
버전 변경: 필드 추가
새로운 요구 사항으로 인해 메시지에 이메일 필드가 추가됩니다.

Proto 파일 (v2):
```proto
syntax = "proto3";

message User {
    int32 id = 1;
    string name = 2;
    string email = 3;  // 추가된 필드
}
```
만약 이 새로운 버전의 메시지를 사용하는 서버가 있고, IDL 서버 없이 이를 구버전 클라이언트가 처리하려고 하면 문제가 발생할 수 있습니다. 구버전 클라이언트는 email 필드를 인식하지 못해 에러가 발생하거나, 메시지를 무시할 수 있습니다.

v1 클라이언트 처리 (구버전 클라이언트):

```proto
syntax = "proto3";

message User {
    int32 id = 1;
    string name = 2;
    string email = 3;  // 추가된 필드
}
```
만약 이 새로운 버전의 메시지를 사용하는 서버가 있고, IDL 서버 없이 이를 구버전 클라이언트가 처리하려고 하면 문제가 발생할 수 있습니다. 구버전 클라이언트는 email 필드를 인식하지 못해 에러가 발생하거나, 메시지를 무시할 수 있습니다.


### 3. IDL 서버가 Backward Compatibility를 보장하는 방식

IDL 서버를 사용하면 구버전과 신버전 메시지 간의 호환성을 보장합니다. 이는 다음과 같은 규칙으로 이루어집니다:

1. 필드 추가는 허용:
- 새 필드를 추가하면 구버전 클라이언트는 이를 무시하되, 기존 필드만 처리할 수 있도록 보장합니다.
- 예: 구버전 클라이언트는 email 필드를 몰라도 메시지를 처리합니다.

2. 필드 제거는 금지:
- 기존 필드를 제거하면 구버전 클라이언트가 메시지를 해석할 수 없게 됩니다.

3. IDL 서버가 여러 버전을 관리:
- IDL 서버는 여러 버전의 .proto 파일을 관리하여, 클라이언트 요청에 맞는 버전의 메시지를 제공할 수 있습니다.

cf) 더 다양한 원칙이 있음. https://docs.confluent.io/platform/current/schema-registry/fundamentals/schema-evolution.html#summary

### 4. Python 코드로 Backward Compatibility 예제

v2 메시지 생성 (신버전 서버):
```python
from user_pb2 import User

# v2 메시지 생성
user = User(id=1, name="John Doe", email="john.doe@example.com")

# 직렬화
serialized_data = user.SerializeToString()
```

v1 클라이언트 처리 (구버전 클라이언트):
```python
from user_pb2 import User

# v1 클라이언트는 v2 메시지를 수신
received_user = User()
received_user.ParseFromString(serialized_data)

# 구버전 클라이언트는 email 필드를 무시하고 처리
print(received_user)

```
출력:
```bash
id: 1
name: "John Doe"

```

### 5. IDL 서버의 주요 역할
IDL 서버는 아래와 같은 방식을 통해 Backward Compatibility를 보장합니다:

1. 다중 버전 지원:
- IDL 서버는 여러 버전의 .proto 파일을 관리합니다.
- 신버전 클라이언트는 v2 스키마를, 구버전 클라이언트는 v1 스키마를 사용하도록 제공합니다.

2. 자동 호환성 체크:
- 새로운 .proto 파일이 추가되거나 수정될 때, 기존 메시지와 호환되는지 검사합니다.

3. 버전 별 직렬화 지원:
- IDL 서버는 요청에 따라 구버전 또는 신버전 메시지로 데이터를 직렬화하여 제공합니다.

### 6. 결론
IDL 서버가 없으면 새로운 필드가 추가되거나 메시지가 변경될 때, 구버전 클라이언트에서 문제가 발생할 수 있습니다. 하지만, IDL 서버를 사용하면 여러 버전의 메시지를 동시에 관리하고, 자동으로 호환성을 검사하며, 클라이언트가 필요로 하는 메시지 버전을 안전하게 제공할 수 있습니다.

Python에서는 Protobuf의 동적 확장성 덕분에 Backward Compatibility를 코드 수준에서도 쉽게 처리할 수 있습니다. IDL 서버는 이를 더 체계적으로 관리할 수 있게 해줍니다.




