<!-- * Serialize : python 객체를 json 으로 변환하는 것 (deserialize - json 을 python 객체로 변환하는 것.) -->
## 1. What is Renderer?

- 클라이언트에 데이터가 전달되기 전에, '렌더링'된다. response 로 전달할 데이터를 내가 원하는 형태로 렌더할 수 있다.
- `Accept` header에서 media_type을 보고 결정한다. url에서 format을 명시할 수도 있다.
```plain
http://localhost:8000/api/stations/?format=json
```
- 사용 : 디폴트 렌더러는 `settings.py` REST_FRAMEWORK에 'DEFAULT_RENDERER_CLASSES'로 넣어주고, `views.py`에서 특정 view에만 지정할 수도 있다.

    1. 디폴트
    ```python
    REST_FRAMEWORK = {
        'DEFAULT_RENDERER_CLASSES': (
            'rest_framework.renderers.JSONRenderer',
            'rest_framework.renderers.BrowsableAPIRenderer',
        )
    }
    ```
    2. 특정 뷰
    ```python
    from rest_framework.renderers import JSONRenderer

    class UserCountView(APIView):
        renderer_classes = (JSONRenderer, )

        def get(self, request, format=None):
            user_count = User.objects.filter(active=True).count()
            content = {'user_count': user_count}
            return Response(content)
    ```

    - 만약 요청에서 media_type를 지정하지 않을 경우(헤더에 `Accept: */*`를 실어 보낼 경우), renderer_classes 에서 지정한 클래스 순서에 따라 적용이 된다.

---

## 2. DRF에서 제공하는 Rederer 종류

- JSONRenderer : Default. DRF 3.0부터는 UnicodeJSONRenderer가 사라지고 unicode가 디폴트가 됨
    - .media_type: `application/json`
    - .format: `'.json'`
    - .charset: `None`

- TemplateHTMLRenderer : 특정 html 템플릿 안에 렌더링하는 것. api 응답 페이지를 커스터마이징 할 수 있다. 그래서 serializing을 하지 않아도 됨. ** `template_name` 인자를 꼭 넘겨줘야 한다.

    ```python
    class UserDetail(generics.RetrieveAPIView):

        queryset = User.objects.all()
        renderer_classes = (TemplateHTMLRenderer,)

        def get(self, request, *args, **kwargs):
            self.object = self.get_object()
            return Response({'user': self.object}, template_name='user_detail.html')
    ```

- StaticHTMLRenderer : html template 없이 html를 string으로 넘겨줌

    ```python
    data = '<html><body><h1>Hello, world</h1></body></html>'
    ```

- AdminRenderer : 어드민 화면과 미슷하게 렌더링 하기(CRUD 지원). nested serializer는 깨짐.

---

## 3. Rederer 커스터마이징하기
```python
def render(self, data, accepted_media_type=None, renderer_context=None):
```
로 커스터마이징 하기 ~아마 건드릴일이 없을 듯?~

    - BaseRenderer에는 아무것도 없음

    ```python
    class BaseRenderer(object):
        media_type = None
        format = None
        charset = 'utf-8'
        render_style = 'text'

        def render(self, data, accepted_media_type=None, renderer_context=None):
            raise NotImplementedError('Renderer class requires .render() to be implemented')
    ```

    - Example
    ```python
    # from django.utils.encoding import smart_unicode <- python3.x에선 지원 안함
    from rest_framework import renderers

    class PlainTextRenderer(renderers.BaseRenderer):
        media_type = 'text/plain'
        format = 'txt'
        charset = 'utf-8'

        def render(self, data, media_type=None, renderer_context=None):
            return data.encode(self.charset)
    ```

---

4. 더 나아가기
    - request 헤더의 media type에 따라 다르게 시리얼라이징 할 수 있다.

    ```python
    def list_users(request):

    queryset = Users.objects.filter(active=True)

    if request.accepted_renderer.format == 'html':  # 이렇게!
        data = {'users': queryset}
        return Response(data, template_name='list_users.html')
    ```

    - media_type 열어두기
    media_type을 */* 같은 식으로 줘서 어떤 request 든 받을 수 있도록 할 수 있다. 그런데 이럴 경우에는 response의 content_type을 꼭 지정해줘야한다.

    ```python
    return Response(data, content_type='image/png')
    ```
---
