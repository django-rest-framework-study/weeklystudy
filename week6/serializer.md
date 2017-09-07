# Serializers

Serializer 는 django 의 Model 인스턴스와 같은 복잡한 데이터를 JSON, XML 또는 다른 형식으로 쉽게 렌더링 할 수 있는 native python 데이터 타입으로 변환할 수 있게 해줍니다.
또한 Serializer 는 **역직렬화(deserialization)** 을 제공해서 들어오는 데이터의 유효성을 검사한 뒤에 데이터를 여러 형식으로 변환하게 해줍니다.
Serializer 는 django 의 Form 이나 ModelForm 과 매우 비슷하게 작동합니다.


### Serializer 정의하기

아래 같은 객체가 있다고 가정합니다.
```
from datet1ime import datetime

class Comment(object):
    def __init__(self, email, content, created=None):
        self.email = email
        self.content = content
        self.created = created or datetime.now()

comment = Comment(email='leila@example.com', content='foo bar')
```

Serializer 는 다음과 같이 정의할 수 있습니다.
```
from rest_framework import serializers

class CommentSerializer(serializers.Serializer):
    email = serializers.EmailField()
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
```


### 직렬화(Serialization) 하기
### 역직렬화(Deserialization) 하기
### 인스턴스 저장하기
### 유효성 검사하기
### 초기 데이터와 인스턴스에 접근하기
### 부분 업데이트
### 중첩된 객체 다루기
### 쓰기 가능한 중첩 표현
### 여러 객체 다루기
### 추가 Context 포함


## ModelSerializer

### ModelSerializer의 검사
### 포함할 필드 지정
### 중첩된 직렬화 지정
### 필드를 명시적으로 지정하기
### 읽기 전용 필드 지정하기
### 추가 키워드 인수
### 관계형 필드
### 필드 매핑 사용자 정의

## HyperlinkedModelSerializer
### 절대 및 상대 URL
### 하이퍼 링크로 연결된 뷰가 결정되는 방법
### URL 필드 이름 변경

## ListSerializer
### allow_empty
### ListSerializer 행동 사용자 정의

## BaseSerializer
### 읽기 전용 BaseSerializer 클래스
### 읽기 - 쓰기 BaseSerializer 클래스
### 새 기본 클래스 만들기

## 고급 Serializer 사용법
### 직렬화 및 직렬화 복원 동작의 오버라이드
### Serializer 상속
### 동적으로 필드 수정하기
### 기본 필드 사용자 정의하기

## 타사 패키지
### Django REST marshmallow
### Serpy
### MongoengineModelSerializer
### GeoFeatureModelSerializer
### HStoreSerializer
### Dynamic REST
### Dynamic Fields Mixin
### DRF FlexFields
### Serializer Extensions
### HTML JSON Forms
### DRF-Base64
### QueryFields
### DRF Writable Nested
