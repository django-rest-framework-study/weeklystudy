# Responses
### Response instance 의 사용
- view 에서 Response class 의 instance 를 사용하라면 `APIView` class 를 사용하거나 `@api_view` function 을 사용해야 합니다.

- Response intance 를 위해서는 data, status, template_name, headers, content_type를 각각 `Arguments`로 갖습니다.
> `Response(data, status=None, template_name=None, headers=None, content_type=None)`
> data 만 positional args

#### 1. Data
```python
class CreateModelMixin(object):
    """
    Create a model instance.
    """
    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        self.perform_create(serializer)
        return Response(serializer.data)
```
- 일반적으로 rest-api 에서는 serializer 를 통해 `serilaizer.data`를 사용 합니다.
- serializer.data 가 아니더라도 `python native dictionary` 또한 사용 가능 합니다.
- 주의할 점은, 해당 data 에는 json.dumps(serializer.data) 로 만든 json 타입이 아니라, python native dictionary object 가 와야 합니다.(render 설정에 따라 해당 dictionary 가 렌더링 됩니다.)

#### 2. status_code
```python
from rest_framework import status


class CreateModelMixin(object):
    """
    Create a model instance.
    """
    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        self.perform_create(serializer)
        return Response(serializer.data, status=status.HTTP_201_CREATED)
```
-` kwagrs`로 `status` 를 받습니다.
-  직접 int 숫자를 입력하지 않고 status.HTTP_201_CREATED 와 같이 drf 의 status 를 이용합니다.
- 하지만 drf 의 status 도 가서 보면 별거 없이 각각의 status code 에 따라  int 로 설정되어있는데, 보다 명시적으로 status code 를 나타내기 위해 이러한 방식을 사용했으리라 생각합니다.
```python
"""
Descriptive HTTP status codes, for code readability.

See RFC 2616 - http://www.w3.org/Protocols/rfc2616/rfc2616-sec10.html
And RFC 6585 - http://tools.ietf.org/html/rfc6585
And RFC 4918 - https://tools.ietf.org/html/rfc4918
"""
# restframework status.py

from __future__ import unicode_literals


def is_informational(code):
    return 100 <= code <= 199


def is_success(code):
    return 200 <= code <= 299


def is_redirect(code):
    return 300 <= code <= 399


def is_client_error(code):
    return 400 <= code <= 499


def is_server_error(code):
    return 500 <= code <= 599


HTTP_100_CONTINUE = 100
HTTP_101_SWITCHING_PROTOCOLS = 101
HTTP_200_OK = 200
HTTP_201_CREATED = 201
HTTP_202_ACCEPTED = 202
HTTP_203_NON_AUTHORITATIVE_INFORMATION = 203
HTTP_204_NO_CONTENT = 204
HTTP_205_RESET_CONTENT = 205
HTTP_206_PARTIAL_CONTENT = 206
HTTP_207_MULTI_STATUS = 207
HTTP_300_MULTIPLE_CHOICES = 300
HTTP_301_MOVED_PERMANENTLY = 301
HTTP_302_FOUND = 302
HTTP_303_SEE_OTHER = 303
HTTP_304_NOT_MODIFIED = 304
HTTP_305_USE_PROXY = 305
HTTP_306_RESERVED = 306
HTTP_307_TEMPORARY_REDIRECT = 307
HTTP_400_BAD_REQUEST = 400
HTTP_401_UNAUTHORIZED = 401
HTTP_402_PAYMENT_REQUIRED = 402
HTTP_403_FORBIDDEN = 403
HTTP_404_NOT_FOUND = 404
HTTP_405_METHOD_NOT_ALLOWED = 405
HTTP_406_NOT_ACCEPTABLE = 406
HTTP_407_PROXY_AUTHENTICATION_REQUIRED = 407
HTTP_408_REQUEST_TIMEOUT = 408
HTTP_409_CONFLICT = 409
HTTP_410_GONE = 410
HTTP_411_LENGTH_REQUIRED = 411
HTTP_412_PRECONDITION_FAILED = 412
HTTP_413_REQUEST_ENTITY_TOO_LARGE = 413
HTTP_414_REQUEST_URI_TOO_LONG = 414
HTTP_415_UNSUPPORTED_MEDIA_TYPE = 415
HTTP_416_REQUESTED_RANGE_NOT_SATISFIABLE = 416
HTTP_417_EXPECTATION_FAILED = 417
HTTP_422_UNPROCESSABLE_ENTITY = 422
HTTP_423_LOCKED = 423
HTTP_424_FAILED_DEPENDENCY = 424
HTTP_428_PRECONDITION_REQUIRED = 428
HTTP_429_TOO_MANY_REQUESTS = 429
HTTP_431_REQUEST_HEADER_FIELDS_TOO_LARGE = 431
HTTP_451_UNAVAILABLE_FOR_LEGAL_REASONS = 451
HTTP_500_INTERNAL_SERVER_ERROR = 500
HTTP_501_NOT_IMPLEMENTED = 501
HTTP_502_BAD_GATEWAY = 502
HTTP_503_SERVICE_UNAVAILABLE = 503
HTTP_504_GATEWAY_TIMEOUT = 504
HTTP_505_HTTP_VERSION_NOT_SUPPORTED = 505
HTTP_507_INSUFFICIENT_STORAGE = 507
HTTP_511_NETWORK_AUTHENTICATION_REQUIRED = 511

```
#### 3. template_name
- drf를 사용하지만 경우에 따라 `HTMLRenderer`로 설정된 경우 `template_name`을 지정해 django templte cbv에서와 같이 rendering 할 template_name 지정이 가능합니다.

#### 4. headers

```python
class CreateModelMixin(object):
    """
    Create a model instance.
    """
    def create(self, request, *args, **kwargs):
        serializer = self.get_serializer(data=request.data)
        serializer.is_valid(raise_exception=True)
        self.perform_create(serializer)
        headers = self.get_success_headers(serializer.data)
        return Response(serializer.data, status=status.HTTP_201_CREATED, headers=headers)

    def perform_create(self, serializer):
        serializer.save()

    def get_success_headers(self, data):
        try:
            return {'Location': data[api_settings.URL_FIELD_NAME]}
        except (TypeError, KeyError):
            return {}

```
- headers 파라미터를 통해 각 상황에 맞는 custom header 정보를 render 에 설정해줄 수 있습니다.

#### 5. content_type
- 일반적으로 request 요청에 따라 `content negotiation` 로  content_type 이 설정되지만,  `content_type='application/json'` 와 같이 명시적으로 적어주는 경우도 있다.
