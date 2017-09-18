# Requests
클라이언트가 웹 서버로 요청을 보내는 방법은 다음과 같습니다.
1. 특정 url 로 요청을 보내는 경우 
예) -X GET https://django-study.kr/chapter/
2. (1) + header 에 정보를 담아 보내는 경우
예) -H "Accept: application/json" -H "Content-Type: application/json" -X GET https://django-study.kr/chapter/
3. url query_param 을 이용하는 경우
예) -x https://django-study.kr/chapter/?accept=application/json
4. url slug field  를 이용하는 경우 (https://django-study.kr/chapter/{chapter_no}/)
예) -X GET https://django-study.kr/chapter/1/
5. POST 방식등 body에 데이터를 담아 보내는 경우
예) curl -H "Content-Type: application/json" -X POST -d '{"username":"xyz","password":"xyz"}' https://django-study.kr/chapter/1/

위 5가지 케이스에 대한 클라이언트 요청처리와 reuqest에 대해 알아보겠습니다.

## 1. 특정 url 로 요청을 보내는 경우 
- 장고는 urls.py 에서 라우팅 처리를 통해 정규표현식과 일치하는 각각의 `view`에 대해 요청을 보냅니다.
- `view`에서는 `requets.method`를 통해 각 요청의 request method 를 확인, 분기 처리합니다.
```python
@api_view(['GET', 'POST'])
def snippet_list(request):
    """
    List all snippets, or create a new snippet.
    """
    if request.method == 'GET':
        snippets = Snippet.objects.all()
        serializer = SnippetSerializer(snippets, many=True)
        return Response(serializer.data)

    elif request.method == 'POST':
        serializer = SnippetSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response(serializer.data, status=status.HTTP_201_CREATED)
        return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

```
- drf 에서 제공 하는 `APIView` 는 request.method 에 대한 처리가 이미 구현되어 있기 때문에 손쉽게 각 메소드의 분기를 처리할 수 있습니다.
```python
# drf APIView 의 dispatch method
      def dispatch(self, request, *args, **kwargs):
        """
        `.dispatch()` is pretty much the same as Django's regular dispatch,
        but with extra hooks for startup, finalize, and exception handling.
        """
        self.args = args
        self.kwargs = kwargs
        request = self.initialize_request(request, *args, **kwargs)
        self.request = request
        self.headers = self.default_response_headers  # deprecate?

        try:
            self.initial(request, *args, **kwargs)

            # 각각의 request.method 에 따라 def get, def post, def update 등등의 메소드와 매핑 
            # Get the appropriate handler method
            if request.method.lower() in self.http_method_names:
                handler = getattr(self, request.method.lower(),
                                  self.http_method_not_allowed)
            else:
                handler = self.http_method_not_allowed

            response = handler(request, *args, **kwargs)

        except Exception as exc:
            response = self.handle_exception(exc)

        self.response = self.finalize_response(request, response, *args, **kwargs)
        return self.response


```
## 2. header 에 정보를 담아 보내는 경우
- 각 헤더에 담긴 정보는 장고의 `request` 와 마찬가지로, `request.META` 에 담기게 됩니다.
- 지난주 토큰 인증에서 알아보았듯, 헤더에  `Authorization` 라는 key 로  value 에 토큰을 담아 auth 처리를 하거나, `Accept` key로 response 받을 data-type을 지정하거나,  `Content-Type` key로 보내는 body 데이터의 type 정보를 제공하기도 합니다.
 - `request.META` 를 통해 각 request header 정보에 접근 할 수 있습니다.
   - 하지만 ` request.META` 에는 `header`이외에 장고 프로세스에 필요한`여러 환경 변수` 또한 담겨 있습니다.
   - 이때문에 클라이언트의 요청에서 보내는 `custom request header` 장고의 `여러 환경 변수`들과 구붓짓기 위해 장고는 다음과 같은 전처리 과정을 거칩니다.
      1. 모든 문자는 대문자로 (Device-Access-token -> DEVICE-ACCESS-TOKEN)
      2. 대시바는 언더바로 (DEVICE-ACCESS-TOKEN -> DEVICE_ACCESS_TOKEN)
      3. 앞에 ' HTTP_ '  를 더한다 (HTTP_DEVICE_ACCESS_TOKEN)
      4. 예시) -H "Accept: application/json" -X GET https://django-study.kr/chapter/ 의 경우 request.META.get('HTTP_ACCEPT)을 통해 요청 헤더값을 알아 낼 수 있습니다.
 > 단 drf APITestCase 를 활용하여 테스트 코드 작성 시, 직접 client 에서  HTTP_DEVICE_ACCESS_TOKEN 을 입력해야합니다. (아마 장고 미들웨어 어디에선가 처리되는 듯 한데, 언제 시간나실때 찾아주시면 감사하겠습니다.)
- request.META.get('HTTP_CONTENT_TYPE') content-type 에 대한 요청은 request.content_type() 으로 구현되어 있습니다.
```python 
# drf 의 Request class
   @property
    def content_type(self):
        meta = self._request.META
        return meta.get('CONTENT_TYPE', meta.get('HTTP_CONTENT_TYPE', ''))
```


## 3. url query_aram 을 이용하는 경우
- request.GET 에 여러 url query param 이 담겨있습니다.
- 예를들어 -x https://django-study.kr/chapter/?accept=application/json의 경우 request.GET.get('accept')를 통해 해당 parameter를 얻을 수 있습니다.
- 하지만, GET.get 모양도 안이쁘고 명시적이지도 않기에 좀더 명시적으로 request.query_param을 구현해 놓았습니다.


```python
   # drf 의 Request class 
   @property
    def query_params(self):
        """
        More semantically correct name for request.GET.
        """
        return self._request.GET
```

> **The Zen of Python**
> Explicit is better than implicit.
> Unless explicitly silenced.
// 저 또한 requst.GET.get() 으로 썼었는데 수정해야겠네요
// 버즈빌 포스팅에서도 GET.get 으로 사용했네요
-> https://www.buzzvil.com/2016/12/26/how-to-use-django-rest-framework-buzzvil/

## 4. url slug field  를 이용하는 경우 (https://django-study.kr/chapter/{chapter_no}/)
- 다들 잘 아시다시피 url slug field 는 `urls.py` 에서 처리됩니다.
- view 에서 해당  불러오려면, self.kwargs.get('chater_no') -- (self는 View class) 를 이용합니다.
- 자세한 내용는 view 나 url 챕터에서 부탁드립니다...

## 5. POST 방식등 body에 데이터를 담아 보내는 경우
- `body contetns` 가 `request.content_type`에 따라 `request.parsers`에서 지정된 타입으로 파싱됩니다.
- `request.POST` 와 달리 drf 에서는 request.PUT, request.PATCH 등 모든 `http request`의 body content 에 대해  일괄적으로 `request.DATA`를 통해 접근합니다. 


## 추가 확인 사항
### request.user, request.auth, request. request.authenticators
- `request.user`는 auth 나 세션에서 생성된 user model instance 를 가지고 있습니다. auth 를 안하거나 실패한 경우 AnonymousUser 를 가집니다.
- `request.auth` 는 특수한 경우(인증 과정에 추가적인 컨텍스트를 리턴? 하는 경우)를 제외 하고 일반적인 auth 인증 경우 None 값을 가집니다.
- `request.authenticators` 는 처리되는 auth 리스트를 확인 할 수 있습니다.
 - 
### DRF 의 Request 가 적용 되는 과정
Response 의 경우 직접 view method 에서 직접 instance 화 시켜서 리턴하지만, Request 의 경우 view 내에서 호출되어 instace 화 되기때문에 코드를 통해 어떠한 방식으로 drf Request가 instance 가 되는지 공유하겠습니다.
- 1. 기본 django View class Method Flowchart를 보면 처음 dispatch 를 호출
![image](https://user-images.githubusercontent.com/14916432/30532227-0378d1b2-9c8e-11e7-9dad-143e26e4d3ad.png)
- 2. django View 를 상속받은 drf APIView

```python
class APIView(View):
...
    # drf APIView의  dispatch method
    def dispatch(self, request, *args, **kwargs):
        """
        `.dispatch()` is pretty much the same as Django's regular dispatch,
        but with extra hooks for startup, finalize, and exception handling.
        """
        self.args = args
        self.kwargs = kwargs

       # request 초기화 메소드 호출
        request = self.initialize_request(request, *args, **kwargs)
        self.request = request
        self.headers = self.default_response_headers  # deprecate?

        try:
            self.initial(request, *args, **kwargs)

            # Get the appropriate handler method
            if request.method.lower() in self.http_method_names:
                handler = getattr(self, request.method.lower(),
                                  self.http_method_not_allowed)
            else:
                handler = self.http_method_not_allowed

            response = handler(request, *args, **kwargs)

        except Exception as exc:
            response = self.handle_exception(exc)

        self.response = self.finalize_response(request, response, *args, **kwargs)
        return self.response
```
- 3. drf 의 initialize_request

```python
from rest_framework.request import Request

class APIView(View):
...
    # drf APIView의  initialize_request method
    
      def initialize_request(self, request, *args, **kwargs):
        """
        Returns the initial request object.
        """
        parser_context = self.get_parser_context(request)
                 # DRF의 Request 를 불러옴
        return Request(
            request,
            parsers=self.get_parsers(),
            authenticators=self.get_authenticators(),
            negotiator=self.get_content_negotiator(),
            parser_context=parser_context
        )


```
