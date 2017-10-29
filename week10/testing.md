# Testing

> "Code without tests is broken as designed(테스트 없는 코드는 의도한 대로 작동하지 않는다.)" - [Jacob Kaplan-Moss](https://www.pycon.kr/2016apac/program/speaker/jacob) (Django co-founder)

## APIRequestFactory
> Django 의 `RequestFactory` 클래스를 오버라이딩한 클래스

### 테스트 요청(Request) 만들기

`APIRequestFactory` 클래스는 Django 의 `RequestFactory` 와 거의 비슷한 수준의 API 를 제공합니다. 따라서 `get()`, `post()` 는 물론 `put()`, `patch()`, `delete()`, `head()` 메서드를 모두 사용할 수 있습니다.

```
from rest_framework.test import APIRequestFactory

# Using the standard RequestFactory API to create a form POST request
factory = APIRequestFactory()
request = factory.post('/notes/', {'title': 'new idea'})
```

#### `format` 인자 사용하기

`POST`, `PUT`, `PATCH` 와 같은 요청 내용을 만드는 메소드에는 `multipart form data` 가 아닌 다른 `content_type` 을 사용하여 요청을 쉽게 생성 할 수 있도록 format 인자가 포함되어 있습니다.

```
# Create a JSON POST request
factory = APIRequestFactory()
request = factory.post('/notes/', {'title': 'new idea'}, format='json')
```

기본적으로 사용할 수 있는 형식은 `multipart` 와 `json` 입니다. Django 의 기본 `RequestFactory` 와의 호환을 위해서 기본 형식은 `multipart` 입니다.

#### 요청(Request) 에 인코딩 방식 지정하기

요청 내용을 명시적으로 인코딩 해야하는 경우 `content_type` 인자를 설정해서 요청 내용을 인코딩 할 수 있습니다.

```
request = factory.post('/notes/', json.dumps({'title': 'new idea'}), content_type='application/json')
```

#### 폼 데이터(form data) 를 활용한 PUT, PATCH 메서드

Django 의 `RequestFactory` 와 REST 프레임 워크의 `APIRequestFactory` 사이에 주목할 가치가있는 차이점 중 하나는 `multipart form data` 가 `post()` 이외의 메소드 용으로 인코딩 된다는 것입니다.
예를 들어 `APIRequestFactory` 를 사용하면 다음과 같이 양식 PUT 요청을 할 수 있습니다.

```
factory = APIRequestFactory()
request = factory.put('/notes/547/', {'title': 'remember to email dave'})
```

Django의 `RequestFactory` 를 사용하면 데이터를 직접 명시적으로 인코딩해야 합니다.

```
from django.test.client import encode_multipart, RequestFactory

factory = RequestFactory()
data = {'title': 'remember to email dave'}
content = encode_multipart('BoUnDaRyStRiNg', data)
content_type = 'multipart/form-data; boundary=BoUnDaRyStRiNg'
request = factory.put('/notes/547/', content, content_type=content_type)
```

### 강제 인증(Authentication)
### CSRF 검증 강제

## APIClient

### 요청
### 인증
### CSRF 유효성 검사

## RequestsClient

### RequestsClient 와 데이터베이스 작업
### 헤더 및 인증
### CSRF
### Live Tests

## CoreAPIClient

### 헤더 및 인증

## Test cases

### 예시

## Testing responses

### 응답(Response) 데이터 확인
### 렌더링 응답(Response)

## Configuration

### 기본 형식 설정
### 사용 가능한 형식 설정
