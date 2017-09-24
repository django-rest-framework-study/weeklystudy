# Exceptions


### DRF views에서의 예외 처리

DRF의 views에서 다양한 error에 알맞는 response를 반환해줌으로써 exception을 처리합니다.

다루는 exception처리는:

* DRF 내 `APIException`의 subclass
* Django의 `Http404`
* Django의 `PermissionDenied`

DRF는 각 경우에 따라 적절한 status code와 내용이 포함된 response를 반환해줍니다. response의 body는 error에 대한 추가적인 상세내용을 포함시킵니다.

대부분의 error response는 `detail` 값을 가지고 있습니다.

```
DELETE http://api.example.com/foo/bar HTTP/1.1
Accept: application/json
```
위의 요청 같은 경우, `DELETE` method는 해당 resource에 대해 허용되지 않았다는 response를 받을 것입니다.

```
HTTP/1.1 405 Method Not Allowed
Content-Type: application/json
Content-Length: 42

{"detail": "Method 'DELETE' not allowed."}
```

Validation error는 조금 다르게 처리를 하며, 필드명을 response 내에 key 형태로 포함시킵니다. 만약 validation error가 특정 field에 특정되어 있지 않으면, "non\_field\_errors" 키를 사용하거나, `NON_FIELD_ERRORS_KEY`에 저장된 값으로 지정될 것입니다.

예를 들어:

```
HTTP/1.1 400 Bad Request
Content-Type: application/json
Content-Length: 94

{"amount": ["A valid integer is required."], "description": ["This field may not be blank."]}
```



### Custom exception handling

자신의 API views에서 발생한 Exception부분을 response 객체로 변환하는 function과 같은 custom exception handler를 만들 수 있다.
이를 통해 자신의 API에 쓰이는 error response를 관리할 수 있다.

생성한 function은 예외처리 되어야 할 부분과 현재 다루어지는 view와 같은 여러 문맥을 담은 dictionary, 2개의 매개변수를 받아야합니다.

이런 exception handler function은 `Response` 객체를, 또는 예외처리하지 못할 경우 `None`을 반환해줘야합니다.

만약 `None`이 반환될 경우, 해당 exception이 다시 발생하고 Django는 standard HTTP 500에 상응하는 '서버에러' 응답을 반환할 것입니다.

예를 들어, 모든 error response가 HTTP 상태코드를 포함되어 있는걸 확인하고 싶은 경우 아래와 같이 시도해 볼 수 있다.

```
HTTP/1.1 405 Method Not Allowed
Content-Type: application/json
Content_Length: 62

{"status_code": 405, "detail": "Method 'DELETE' not allowed."}
```

response의 방식을 고치려면, 다음과 같이 custom exception handler를 쓸 수 있다.

```python
from rest_framework.views import exception_handler

def custom_exception_handler(exc, context):
	# 표준 error response를 받기 위해
	# DRF의 default exception handler를 불러온다.
	response = exception_handler(exc, context)
	
	# 이제 response에 HTTP status code를 붙여준다.
	if response is not None:
		response.data['status_code'] = response.status_code
		
	return response
```

context argument는 default handler에서 사용하지 않는 대신, `context['view']`로 접근가능한 view와 같은 추가적인 정보가 필요한 exception handler에서 유용하게 쓰일 수 있다.

exception handler는 `settings.py`에서 `EXCEPTION_HANDLER`를 사용하여 아래와 같이 설정해줘야 한다.

```
REST_FRAMEWORK = {
	'EXCEPTION_HANDLER': 'my_project.my_app.utils.custom_exception_handler'
}
```

만약 명확하게 지정 되어있지 않을 경우, `EXCEPTION_HANDLER`는 DRF에서 제공되는 default standard exception으로 설정된다.

```
REST_FRAMEWORK = {
	'EXCEPTION_HANDLER': 'rest_framework.views.exception_handler'
}
```

Exception handler는 해당 exception에서 발생한 response로 호출된다는 것을 명심하길 바랍니다.
이는 `HTTP_400_BAD_REQUEST`와 같이 serializer validation이 실패했을 때 generic views에서 반환되는 것과 같이, views에 의해 즉시 반환되는 response에서 쓰이지 않습니다. ***

<br>
--
# API Reference
### APIException

**Signiture:** `APIException()`

`APIView`클래스 또는 `@api_view` 안에서 발생한 모든 exception에 쓰이는 **base class**

custom exception을 제공하기 위해, `APIException`을 상속받고, `.status_code`, `default_detail`, 그리고 `default_code`를 설정합니다.

예를 들어, API가 종종 접근 불가능한 third party service에 의존하고 있다면, HTTP response code에 해당하는 "503 Service Unavailable"에 대한 exception을 아래와 같이 넣을 수 있습니다.

```python
from rest_framework.exceptions import APIException

class ServiceUnavailable(APIException):
    status_code = 503
    default_detail = 'Service temporarily unavailable, try again later.'
    default_code = 'service_unavailable'
```

> Inspecting API exceptions

API exception을 점검할 수 있는 다양한 property가 있습니다. 이를 사용하여 각 프로젝트에 exception handling을 만들 수 있습니다.

가능한 attributes, methods :

* `.detail` - error에 대한 text 설명을 반환
* `.get_codes()` - error의 코드 식별자를 반환
* `.get_full_details()` - text 설명과 코드 식별자 둘 다 반환

대부분의 경우 error detail은 다음과 같은 간단한 형태를 반환합니다.

```python
>>> print(exc.detail)
You do not have permission to perform this action.

>>> print(exc.get_codes())
permission_denied

>>> print(exc.get_full_details())
{'message': 'You do not have permission to perform this action.', 'code': 'permission_denied'}
```

validation error의 경우 error detail은 list 또는 dictionary 형태로 반환할 것입니다.

```python
>>> print(exc.detail)
{"name":"This field is required.","age":"A valid integer is required."}

>>> print(exc.get_codes())
{"name":"required","age":"invalid"}

>>> print(exc.get_full_details())
{"name":{"message":"This field is required.","code":"required"},"age":{"message":"A valid integer is required.","code":"invalid"}}
```
---
<br><br>
### ParseError

**Signature:** `ParseError(detail=None, code=None)`

Reqeust가 잘못된 data를 가지고 `request.data`에 접근하려고 할 때 발생합니다.

default 값으로 HTTP status code인 "400 Bad Request'를 가진 response가 들어갑니다.

--

### AuthenticationFailed

**Signature:** `AuthenticationFailed(detail=None, code=None)`

incoming request가 잘못된 authentication을 포함할 때 발생합니다.

default 값으로 HTTP status code인 "401 Unauthenticated"를 가지고 있지만, authentication scheme에 따라 "403 Forbidden"를 가진 response가 들어갈 수 있습니다.
**authentication documentation**에서 자세한 내용을 참고.

--

### NoAuthenticated

**Signature:** `NoAuthenticated(detail=None, code=None)`

**AuthenticationFailed**와 동일.
***다시 확인하기

--

### PermissionDenied

**Signature:** `PermissionDenied(detail=None, code=None)`

인증된 request가 permission check에서 실패하면 발생합니다.

default 값으로 HTTP status code인 "403 Forbidden"를 가진 response가 들어갑니다.

--

### NotFound

**Signature:** `NotFound(detail=None, code=None)`

주어진 URL에 맞는 resource가 없을 때 발생합니다. 이는 Django exception의 `Http404`와 동등합니다.

default 값으로 HTTP status code인 "404 Not Found"를 가진 response가 들어갑니다.

--

### MethodNotAllowed

**Signature:** `MethodNotAllowed(method, detail=None, code=None)`

view에 존재하는 handler method에 전달되지 않는 incoming request가 들어왔을 때 발생합니다.

default 값으로 HTTP status code인 "405 Method Not Allowed"를 가진 response가 들어갑니다.

--

### NotAcceptable

**Signature:** `NotAcceptable(detail=None, code=None)`

프로젝트에서 가능한 renderer로 충족할 수 없는 `Accept` header를 가진 incoming request가 들어왔을 경우 발생합니다.

default 값으로 HTTP status code인 "406 Not Acceptable"를 가진 response가 들어갑니다.

--

### UnsupportedMediaType

**Signature:** `UnsupportedMediaType(media_type, detail=None, code=None)`

`request.data`에 접근할 때 reqeust의 content 종류를 다룰 수 있는 parser가 없을 경우 발생합니다.

default 값으로 HTTP status code인 "415 Unsupported Media Type"를 가진 response가 들어갑니다.

--

### Throttled

**Signature:** `Throttled(wait=None, detail=None, code=None)`

incoming request가 throttling 검사에서 실패했을 때 발생합니다.

default 값으로 HTTP status code인 "429 Too Many Requests"를 가진 response가 들어갑니다.

> throttling - 짧은 시간에 여러번의 같은 요청을 하나의 요청으로 서버에 전송시켜 서버 부하를 방지하는 방법

--

### ValidationError

**Signature:** `ValidationError(detail, code=None)`

`ValidationError` exception은 다른 `APIException` 클래스와는 조금 다릅니다.

* `detail` argument가 필수적으로 입력되어야 합니다.
* `detail` argument가 error detail이 list 또는 dictionary와 같은 집합적인(nested) data 구조로 되어 있을 수 있습니다.
* 관습대로(by convention) Django의 built-in validation error와 구별하기 위해 serializder module을 import 시켜야하고, 적합한 `ValidationError`를 사용해야 합니다.
<br>예시 :  `raise.serializers.ValidationError('This field must be an integer value.')`

`ValidationError` 클래스는 serializer와 field validation에 validator class로써 사용되어야 합니다.<br>
이는 `serializer.is_valid`를 `raise_exception`을 argument로 호출했을 때 발생시켜야 합니다.

```
serializer.is_valid(raise_exception=True)
```

generic view는 `raise_exception=True`를 사용하여 validation error response의 형식을 자신의 API에 전역적으로 override 하여 사용할 수 있습니다.<br>
이렇게 하기 위해, 위에 나타낸 것과 같이 custom exception handler를 사용하면 됩니다.

default 값으로 HTTP status code인 "400 Bad Request"를 가진 response가 들어갑니다.