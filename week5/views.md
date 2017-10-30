# Views

## 클래스 기반 뷰 (Class Based Views)
이 문서는 [DRF 공식 문서](http://www.django-rest-framework.org/api-guide/views/)를 번역한 것입니다.

> Django 의 클래스 기반 뷰는 기존의 views 로부터 시작하기에 좋습니다.
> — [Reinout van Rees](http://reinout.vanrees.org/weblog/2011/08/24/class-based-views-usage.html)


DRF 는 Django 의 `View` 하위 클래스로 `APIView` 클래스를 제공합니다.

DRF 의 `APIView` 와 `View` 의 차이점은 아래와 같습니다:

- 핸들러 메소드에 전달된 요청은 Django 의 `HttpRequest` 인스턴스가 아닌 DRF 의 `Request` 인스턴스를 통해 이루어집니다.
- 핸들러 메소드는 Django 의 `HttpResponse` 가 아닌 DRF 의 `Response` 를 리턴할 것입니다. view 는 [컨텐츠 협상(Content Negotiation)](https://developer.mozilla.org/ko/docs/Web/HTTP/Content_negotiation)을 하며 응답에 필요한 올바른 렌더러를 세팅합니다.
- 모든 `APIException` 예외는 캐치되어 적절한 응답으로 조정됩니다.
- 들어오는 요청은 인증되고 핸들러 메소드로 요청이 들어가기 전에 권한 또는 Throttle check 가 실행됩니다.

`APIView` 클래스를 사용하면 대개 `View` 클래스와 거의 동일하게 사용할 수 있습니다. 들어오는 요청을 `.get()` 또는 `.post()` 와 같은 적절한 핸들러 메소드로 보낼 수 있습니다. 추가적으로, 각 속성들이 클래스에 설정되어 다양한 API 정책에 따라 컨트롤할 수 있습니다.

예시는 아래와 같습니다.
```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import authentication, permissions
from django.contrib.auth.models import User

class ListUsers(APIView):
    """
    시스템의 모든 사용자를 리스트로 보여줍니다.

    * 토큰 인증을 필요로 합니다.
    * 관리자만 view 에 접근할 수 있습니다.
    """
    authentication_classes = (authentication.TokenAuthentication,)  # 토큰 인증
    permission_classes = (permissions.IsAdminUser,)  # 접근 권한

    def get(self, request, format=None):
        """
        모든 사용자를 리턴합니다.
        """
        usernames = [user.username for user in User.objects.all()]
        return Response(usernames)
```

### API 정책 속성
아래의 속성들을 부여하여 `APIVIew` 의 정책을 컨트롤 할 수 있습니다.
> 보완 예정

#### `.renderer_classes`
#### `.parser_classes`
#### `.authentication_classes`
#### `.throttle_classes`
#### `.permission_classes`
#### `.content_negotiation_class`


### API 정책 인스턴스화 메소드
아래의 메소드들은 DRF 에서 인스턴스화한 API 정책입니다. 이 메소드들은 대체로 재정의 할 필요가 없습니다.
> 보완 예정

#### `.get_renderers(self)`
#### `.get_parsers(self)`
#### `.get_authenticators(self)`
#### `.get_throttles(self)`
#### `.get_permissions(self)`
#### `.get_content_negotiator(self)`
#### `.get_exception_handler(self)`


### API 정책 구현 메소드
이 메소드들은 핸들러 메소드로 요청되기 전에 호출됩니다.
> 보완 예정

#### `.check_permissions(self, request)`
#### `.check_throttles(self, request)`
#### `.perform_content_negotiation(self, request, force=False)`


### 요청 전송 메소드
아래의 메소드는 뷰의 `.dispatch()`메소드에서 직접 호출합니다. 이 메소드들은 `.get()`, `.post()`, `put()`, `patch()` 및 `.delete()`와 같은 핸들러 메소드들을 호출하기 전후에 동작해야 하는 모든 작업들을 수행합니다.


#### `.initial(self, request, *args, **kwargs)`
핸들러 메소드가 호출되기 전에 발생해야하는 모든 작업을 수행합니다. 이 메소드는 사용 권한 및 제한을 적용하고 콘텐츠 협상을 수행하는데 사용됩니다.
일반적으로 이 메소드는 재정의를 할 필요가 없습니다.

#### `.handle_exception(self, exc)`
핸들러 메소드에 의해 버려진 예외는 `Resopnse`인스턴스를 반환하거나 예외를 다시 발생시키는 이 메소드로 전달됩니다. 기본 구현에서는 Django 의 `Http404` 와 `PermissionDenied` 예외 뿐만 아니라 `rest_framework.exceptions.APIException` 의 하위 클래스를 처리하고 해당 오류에 대한 response 값을 반환합니다. API 에서 반환하는 response 값을 커스터마이징 해야 하는 경우엔 이 메소드를 서브 클래스화해야 합니다.

#### `.initialize_request(self, request, *args, **kwargs)`
핸들러 메소드에 전달 된 request 객체가 일반적인 Django `HttpRequest`가 아닌 `Request`의 인스턴스인지 확인합니다. 일반적으로 이 메소드를 재정의 할 필요는 없습니다.

#### `.finalize_response(self, request, response, *args, **kwargs)`
핸들러 메소드에서 반환된 모든 `Response`객체가 내용 협상에 의해 결정된 대로 올바른 내용 유형으로 렌더링되도록 합니다. 일반적으로 이 메소드는 재정의 할 필요는 없습니다.

---

## 함수 기반 뷰 (Function Based Views)
> **클래스 기반의 뷰**가 항상 우월한 해결책이라고 말하는 것은 실수입니다.
> — [Nick Coghlan](http://www.curiousefficiency.org/posts/2012/05/djangos-cbvs-are-not-mistake-but.html)

DRF 를 사용하면 일반적인 함수형 기반 뷰 작업을 할 수 있습니다. 이것은 Django 의 간단한 데코레이터들로 **함수 기반의 뷰**를 감싸서 (DRF 의) `Request` 인스턴스를 전달받고 (DRF 의) `Response` 를 리턴하도록 하며, 받은 요청이 어떻게 진행되는지 설정합니다.


### @api_view()
**Signature**: `@api_view(http_method_names=['GET'], exclude_from_schema=False)`
이 기능의 핵심은 뷰가 응답해야하는 HTTP 메소드 리스트를 사용하는 `api_view` 데코레이터입니다. 예를 들어, 아래는 몇 가지 데이터를 수동으로 반환하는 아주 간단한 뷰를 작성하는 방법입니다.

```python
from rest_framework.decorators import api_view

@api_view()
def hello_world(request):
    return Response({"message": "Hello, world!"})
```

이 뷰는 [설정](http://www.django-rest-framework.org/api-guide/settings/)에 지정된 기본 렌더러, 파서, 인증 클래스 등을 사용합니다. 기본적으로 `GET` 메소드만 허용됩니다. 다른 메소드들은 "405 Method Not Allowed" 로 응답합니다. 이 동작을 변경하려면 view에서 허용하는 방법을 지정하세요.

```python
@api_view(['GET', 'POST'])
def hello_world(request):
    if request.method == 'POST':
        return Response({"message": "Got some data!", "data": request.data})
    return Response({"message": "Hello, world!"})
```
`exclude_from_schema`인수를 사용하여 API 뷰를 [자동 생성 스키마(auto-generated schema)](http://www.django-rest-framework.org/api-guide/schemas/)에서 생략된 것으로 표시 할 수도 있습니다.

```python
@api_view(['GET'], exclude_from_schema=True)
def api_docs(request):
    ...
```

### API policy decorators
기본 설정을 재정의하기 위해 REST 프레임워크는 뷰에 추가 할 수 있는 일련의 추가 데코레이터를 제공합니다. 이들은 `@api_view`데코레이터 다음에 와야합니다. 예를 들어, [`throttle`](http://www.django-rest-framework.org/api-guide/throttling/)을 사용하여 특정 사용자가 하루에 한번만 호출 할 수 있도록 뷰를 만들려면 `@thottle_classes` 데코레이터를 사용하여 `throttle` 클래스 목록을 전달하세요.

```python
from rest_framework.decorators import api_view, throttle_classes
from rest_framework.throttling import UserRateThrottle

class OncePerDayUserThrottle(UserRateThrottle):
        rate = '1/day'

@api_view(['GET'])
@throttle_classes([OncePerDayUserThrottle])
def view(request):
    return Response({"message": "Hello for today! See you tomorrow!"})
```

이러한 데코레이터들은 위에서 설명한 `APIView` 하위 클래스에 설정된 특성에 해당합니다. 사용 가능한 데코레이터는 다음과 같습니다.

- `@renderer_classes(...)`
- `@parser_classes(...)`
- `@authentication_classes(...)`
- `@throttle_classes(...)`
- `@permission_classes(...)`

이러한 데코레이터들은 클래스의 `list` 나 `tuple` 인 단일 인수를 취합니다.

### 뷰 스키마 데코레이터(View Schima Decorator)
함수 기반 뷰의 기본 스키마를 재정의 하기 위해서는 `@schema` 데코레이터를 사용해야 합니다. 이것은 `@api_view` 데코레이터 이후에 위치합니다.

```python
from rest_framework.decorators import api_view, schema
from rest_framework.schemas import AutoSchema

class CustomAutoSchema(AutoSchema):
    def get_link(self, path, method, base_url):
        # override view introspection here...

@api_view(['GET'])
@schema(CustomAutoSchema())
def view(request):
    return Response({"message": "Hello for today! See you tomorrow!"})
```

이 데코레이터는 단일 `AutoSchema` 인스턴스를 가지며, `AutoSchema` 서브클래스 인스턴스와 `ManualSchema` 인스턴스에 대한 정보는 [Schemas 문서](http://www.django-rest-framework.org/api-guide/schemas/) 에서 볼 수 있습니다. 뷰 스키마를 제외하고 싶을 땐 `None` 을 사용하면 됩니다.

```python
@api_view(['GET'])
@schema(None)
def view(request):
    return Response({"message": "Will not appear in schema!"})
```


## 참고
1. [DRF 공식 문서](http://www.django-rest-framework.org/api-guide/views)
