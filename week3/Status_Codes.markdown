# Status Codes
--
#### DRF에서 Response에 알맞는 status code 사용시, bare 형태보다 DRF 내에 포함된 코드를 사용하는 걸 선호합니다.

각 view의 Response에 필요한 status code를 돌려줍니다.

```python
# SomeApp/views.py

from rest_framework import status
from rest_framework.response import Response

def empty_view(self):
	content = {'please move along': 'nothing to see here'}
	return Response(content, status=status.HTTP_404_NOT_FOUND)
```

**APITestCase**로 status code가 해당 범위에 존재하는지 테스트를 진행해 볼 수 있습니다.

```python
from rest_framework import status
from rest_framework.test import APITestCase

class ExampleTestCase(APITestCase):
    def test_url_root(self):
        url = reverse('index')
        response = self.client.get(url)
        self.assertTrue(status.is_success(response.status_code))
```

HTTP status code에 대해서는 **RFC2616**과 **RFC6585**에서 확인할 수 있습니다.

* **Informational - 1xx**

> 임시적인 response를 나타냅니다. DRF에서는 1xx를 default로 사용하지 않습니다.

```
HTTP_100_CONTINUE
HTTP_101_SWITCHING_PROTOCOLS
```

* **Successful - 2xx**

> 클라이언트의 요청를 제대로 받고, 이해하여 처리한지 나타냅니다.

```
HTTP_200_OK
HTTP_201_CREATED
HTTP_202_ACCEPTED
HTTP_203_NON_AUTHORITATIVE_INFORMATION
HTTP_204_NO_CONTENT
HTTP_205_RESET_CONTENT
HTTP_206_PARTIAL_CONTENT
HTTP_207_MULTI_STATUS
```

* **Redirection - 3xx**

> 유저가 request를 보내기 위해 필요한 추가적인 행동을 나타냅니다.

```
HTTP_300_MULTIPLE_CHOICES
HTTP_301_MOVED_PERMANENTLY
HTTP_302_FOUND
HTTP_303_SEE_OTHER
HTTP_304_NOT_MODIFIED
HTTP_305_USE_PROXY
HTTP_306_RESERVED
HTTP_307_TEMPORARY_REDIRECT
```

* **Client Error - 4xx**

> 클라이언트가 잘못된 요청을 보냈을 경우에 보내줍니다. HEAD request에 반응하는 경우를 제외하고, 서버는 에러에 대해 설명과 함께 해당  일시적/영구적 상태를 entity를 포함하고 있어야한다.

```
HTTP_400_BAD_REQUEST
HTTP_401_UNAUTHORIZED
HTTP_402_PAYMENT_REQUIRED
HTTP_403_FORBIDDEN
HTTP_404_NOT_FOUND
HTTP_405_METHOD_NOT_ALLOWED
HTTP_406_NOT_ACCEPTABLE
HTTP_407_PROXY_AUTHENTICATION_REQUIRED
HTTP_408_REQUEST_TIMEOUT
HTTP_409_CONFLICT
HTTP_410_GONE
HTTP_411_LENGTH_REQUIRED
HTTP_412_PRECONDITION_FAILED
HTTP_413_REQUEST_ENTITY_TOO_LARGE
HTTP_414_REQUEST_URI_TOO_LONG
HTTP_415_UNSUPPORTED_MEDIA_TYPE
HTTP_416_REQUESTED_RANGE_NOT_SATISFIABLE
HTTP_417_EXPECTATION_FAILED
HTTP_422_UNPROCESSABLE_ENTITY
HTTP_423_LOCKED
HTTP_424_FAILED_DEPENDENCY
HTTP_428_PRECONDITION_REQUIRED
HTTP_429_TOO_MANY_REQUESTS
HTTP_431_REQUEST_HEADER_FIELDS_TOO_LARGE
HTTP_451_UNAVAILABLE_FOR_LEGAL_REASONS
```

* **Server Error - 5xx**

> 서버에서 문제가 발생하거나 해당 request를 처리하지 못하는 상태일 경우 일어난다. HEAD request의 응답을 제외하고, 서버는 해당 오류에 대한 설명과 일시적/영구적 상태를 포함하고 있어야한다.

```
HTTP_500_INTERNAL_SERVER_ERROR
HTTP_501_NOT_IMPLEMENTED
HTTP_502_BAD_GATEWAY
HTTP_503_SERVICE_UNAVAILABLE
HTTP_504_GATEWAY_TIMEOUT
HTTP_505_HTTP_VERSION_NOT_SUPPORTED
HTTP_507_INSUFFICIENT_STORAGE
HTTP_511_NETWORK_AUTHENTICATION_REQUIRED
```

* **Helper functions**

> 위처럼 분류된 response의 종류를 알려주는게 가능하다.

```
is_informational()	# 1xx
is_success()		# 2xx
is_redirect()		# 3xx
is_client_error()	# 4xx
is_server_error()	# 5xx
```
