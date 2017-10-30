<!-- * Serialize : python 객체를 json 으로 변환하는 것 (deserialize - json 을 python 객체로 변환하는 것.) -->
1. What is Renderer?

- 클라이언트에 데이터가 전달되기 전에, '렌더링'된다. response 로 전달할 데이터를 내가 원하는 형태로 렌더할 수 있다.
- `Accept` header에서 media_type을 보고 결정한다. url에서 format을 명시할 수도 있다. http://localhost:8000/api/stations/?format=json
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

- Renderers
    - How the renderer is determined
    - Setting the renderers
    - Ordering of renderer classes


- API Reference
    - JSONRenderer
    - TemplateHTMLRenderer
    - StaticHTMLRenderer
    - BrowsableAPIRenderer
    - AdminRenderer
    - HTMLFormRenderer
    - MultiPartRenderer
- Custom renderers
    - data
    - media_type=None
    - renderer_context=None
    - Example
    - Setting the character set
- Advanced renderer usage
    - Varying behaviour by media type
    - Underspecifying the media type
    - Designing your media types
    - HTML error views
- Third party packages
    - YAML
    - XML
    - JSONP
    - MessagePack
    - CSV
    - UltraJSON
    - CamelCase JSON
    - Pandas (CSV, Excel, PNG)
    - LaTeX


---

Request headers (500에러 났을 때)
```plain
Accept:*/*
Accept-Encoding:gzip, deflate, br
Accept-Language:en,ko;q=0.8,en-US;q=0.6,es;q=0.4,zh;q=0.2,is;q=0.2,fr;q=0.2,la;q=0.2
Cache-Control:no-cache
Connection:keep-alive
Content-Length:75
**Content-Type:application/json
Cookie:_ga=GA1.1.1324380169.1505131294; csrftoken=BEYL7DQ502zPVr6Mtpp8BQ0DW95cpYm5uXkaF6N4C57S1gs7k1vUby8ulas8twZ7
Host:127.0.0.1:8000
Origin:http://127.0.0.1:8000
Pragma:no-cache
Referer:http://127.0.0.1:8000/docs/
User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100 Safari/537.36
```

admin 페이지에서 post 날렸을 때
```plain
General
Request URL:http://localhost:8000/admin/stations/station/add/
**Request Method:POST
**Status Code:302 Found
Remote Address:127.0.0.1:8000
Referrer Policy:no-referrer-when-downgrade

Response Headers
Cache-Control:max-age=0, no-cache, no-store, must-revalidate
Content-Length:0
Content-Type:text/html; charset=utf-8
Date:Mon, 30 Oct 2017 03:27:09 GMT
Expires:Mon, 30 Oct 2017 03:27:09 GMT
Location:/admin/stations/station/
Server:WSGIServer/0.2 CPython/3.6.1
Set-Cookie:messages="6a1bb0ed64bdc407fd29c0a26c1db08186226184$[[\"__json_message\"\0541\05425\054\"The station \\\"<a href=\\\"/admin/stations/station/17/change/\\\">\\ud569\\uc815</a>\\\" was added successfully.\"]]"; HttpOnly; Path=/
Vary:Cookie
X-Frame-Options:SAMEORIGIN

Request Headers
view source
**Accept:text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8
Accept-Encoding:gzip, deflate, br
Accept-Language:en,ko;q=0.8,en-US;q=0.6,es;q=0.4,zh;q=0.2,is;q=0.2,fr;q=0.2,la;q=0.2
Cache-Control:no-cache
**Connection:keep-alive
Content-Length:985
**Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryUBF4ZKCEWVPUvxhb
Cookie:_xsrf=2|f385af49|9aee6031877b1e6b97384070010e06f1|1508398036; username-localhost-8889="2|1:0|10:1508399367|23:username-localhost-8889|44:NGNhNDQ3MWNjYjYwNDk2MGI4YWIwOWNiNmE0N2U2M2I=|56b323d2445c4b192f607a801539cb1762b3c01163551364a3de32ba254f6ab8"; username-localhost-8888="2|1:0|10:1508406533|23:username-localhost-8888|44:ZTk0MzNmMDQxOWMxNDVjOWE4ZWU2OThlNDE4YWNiMDE=|85734531bd705cf6611858ebeffacc8553ead6e476250c786c456d1ad24b4c62"; haveSeenNotification=true; __atuvc=84%7C39%2C67%7C40%2C64%7C41%2C6%7C42%2C45%7C43; __insp_uid=1335234838; sidebar_pinned=true; __insp_wid=1598222036; __insp_nv=false; __insp_targlpu=aHR0cDovL2xvY2FsaG9zdDozMDAwLw%3D%3D; __insp_targlpt=SG9tZSAtIFJlcGljay5jbw%3D%3D; __insp_norec_sess=true; _ga=GA1.1.253132700.1501829063; _gid=GA1.1.1745012908.1509263822; __insp_slim=1509293470434; mp_80f040485809bcea45596281a425d998_mixpanel=%7B%22distinct_id%22%3A%20%2215bf5c57ac5f6-036630a308c408-30627509-13c680-15bf5c57ac7138%22%2C%22%24initial_referrer%22%3A%20%22%24direct%22%2C%22%24initial_referring_domain%22%3A%20%22%24direct%22%7D; mp_mixpanel__c=0; sessionid=uh7vjcc1hupre3czdtx8v6pk62xv5ola; csrftoken=kc3F8faBa4ivOpP5clu0zKEfUMLHDagUJhAQchpOfiJtYg4idMcOPayPqEknbR3j
Host:localhost:8000
Origin:http://localhost:8000
Pragma:no-cache
Referer:http://localhost:8000/admin/stations/station/add/
Upgrade-Insecure-Requests:1
User-Agent:Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/61.0.3163.100 Safari/537.36
```
