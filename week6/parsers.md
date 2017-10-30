1. What is Parser?
- request header에 담긴 내용(미디어 타입 등)을 parse하여 요청한 대로 response를 보내준다.
- 사용 : 디폴트 렌더러는 `settings.py` REST_FRAMEWORK에 'DEFAULT_PARSER_CLASSES'로 넣어주고, `views.py`에서 특정 view에만 지정할 수도 있다.

    1. 디폴트

    ```python
    REST_FRAMEWORK = {
        'DEFAULT_PARSER_CLASSES': (
            'rest_framework.parsers.JSONParser',
        )
    }
    ```

    2. 특정 뷰

    ```python
    from rest_framework.parsers import JSONParser

    class ExampleView(APIView):
        parser_classes = (JSONParser,)

        def post(self, request, format=None):
            return Response({'received data': request.data})
    ```

2. 각 요청에 따른 Parse

    1. GET 요청 시
        - request header 에 담긴 내용을 parse 한다.
        - Accept, Content-type 등이 담겨 있다.
            - Accept: default는 `rest_framework.parsers.JSONParser`이기 때문에 `application/json`이 찍혀있음.
            - Content-type: text/html, application/json 등이 있음

    2. POST 요청 시
        -
        -

---

- Parsers
    - How the parser is determined
    - Setting the parsers

- DRF에서 제공하는 Parser 종류
    1. JSONParser
    2. FormParser : html form content
        - request.data will be populated with a QueryDict.
        - FormParser and MultiPartParser 같이 쓰임
        - .media_type: `application/x-www-form-urlencoded` 로 지정해준다.
    3. MultiPartParser
        - request.data will be populated with a QueryDict.
        - .media_type: `multipart/form-data`
    4. FileUploadParser

- Custom parsers도 만들 수 있다!
: `.parse(self, stream, media_type, parser_context)`로 오버라이딩 하기
    - stream : A stream-like object representing the body of the request.
    - media_type : Optional. media type of the incoming request content. _Depending on the request's Content-Type: header_, this may be more specific than the renderer's media_type attribute, and may include media type parameters. For example "text/plain; charset=utf-8".
    - parser_context : Optional.

    ```python
    class PlainTextParser(BaseParser):
    """
    Plain text parser.
    """
    media_type = 'text/plain'

    def parse(self, stream, media_type=None, parser_context=None):
        """
        Simply return a string representing the body of the request.
        """
        return stream.read()
    ```
