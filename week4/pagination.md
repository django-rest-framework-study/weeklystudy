# Pagination
Pagination은 결과 값의 크기가 너무 클 때, 그런데 그 전부가 항상 필요한 건 아닐 때 자주 사용되는 기법입니다.

[Django 공식 문서](https://docs.djangoproject.com/en/1.11/topics/pagination/)에 따르면,
Django는 기본적으로 data를 page로 나눠서 관리할 수 있는 class를 몇가지 제공합니다. Page 간 이동(Previous/Next를 사용)까지 되는 버전으로요.

또한 rest_framework에서도 커스텀된 paging 방법을 제공하죠.
큰 결과 값들을 어떻게 쪼갤지에 대해서 custom 할 수 있습니다.

Pagination API는 다음 두가지 방식을 지원합니다.
* Pagination link가 response 값의 content안에 제공되는 방식 (built-in된 방식)
* Pagination link가 response 값의 header안에 포함되는 방식

Pagination은 일반 APIView가 아닌 GenericAPIView나 viewset을 사용해야만 자동으로 사용할 수 있습니다.
APIView를 사용한다면 따로 Pagination API를 호출해야합니다.

## Pagination 스타일 설정하기

기본적으로 Pagination은 크게 두 가지 속성을 기억하면 됩니다.

`DEFAULT_PAGINATION_CLASS`와 `PAGE_SIZE`.

만약 내장된 pagination을 사용하고 싶다면 settings.py에 다음과 같이 설정하면 됩니다.
```python
# settings.py
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination',
    'PAGE_SIZE': 100
}
```

그러면 위에서 말한대로, response content안에 값들이 생기게 됩니다.
<img src="images/4.png" alt="pagination 설정" width= "80%" />

## Modifying the pagination style
## API Reference
## PageNumberPagination
## LimitOffsetPagination
## CursorPagination
## Custom pagination styles
## Example
## Using your custom pagination class
## Pagination & schemas
## HTML pagination controls
## Customizing the controls
## Third party packages
## DRF-extensions
## drf-proxy-pagination
## link-header-pagination
