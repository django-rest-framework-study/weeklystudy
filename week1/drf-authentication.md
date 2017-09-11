# Authentication
- Authentication은 View의 시작, Permission이 수행되기 전에 실행된다.
- 인증 스키마는 클래스 단위로 정의되며, 각 클래스에 대해 인증을 시도한다. 성공적으로 인증을 한 클래스의 반환 값을 사용하여 `request.user` 및 `request.auth`를 설정한다.
- **클래스가 인증되지 않으면** request.user(User와 관련이 있다.)는 `AnonymousUser 인스턴스`로 설정되고, request.auth(Token과 관련이 있다.)는 `None`으로 설정된다.

## 언제 Authentication을 사용해야 할까?
일반적으로 우리가 제작한 API 데이터를 편집하거나 삭제할 수 있으며 아무런 제한이 없다. 따라서 다음과 같은 추가 기능이 필요하다.

- 가공된 데이터는, 데이터를 제작한 사람과 늘 관련이 있다.
- 오직 인증된 유저만 데이터를 가공할 수 있다.
- 데이터를 제작한 사람만 수정하거나 삭제할 수 있다.
- 인증되지 않은 요청들(로그인 안 한 유저 혹은 해당 데이터 작성자가 아닌 유저)은 '읽기 전용'으로만 접근해야한다.

## Authentication 설정하기

기본 인증 스키마는 `settings`에서 `DEFAULT_AUTHENTICATION_CLASSES`를 사용하여 설정할 수 있다.

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.BasicAuthentication',
        'rest_framework.authentication.SessionAuthentication',
        [... 여러 종류의 인증 스키마]
    )
}
```

## API 뷰에 Authentication 적용하기

### CBV

`APIView`에서 `authentication_classes`를 통해 튜플 형태의 단위별 Authentication을 설정할 수 있다. 

```python
class ExampleView(APIView):
	# 첫 번째 인증 클래스는 응답 유형을 결정할 때 사용한다.
    authentication_classes = (SessionAuthentication, BasicAuthentication)
    permission_classes = (IsAuthenticated,)
    
    def get(self, request):
    	[...]
```

### FBV

또는 `@api_view` 데코레이터를 통해 FBV에서 설정할 수도 있다.

```python
@api_view(['GET'])
@authentication_classes((SessionAuthentication, BasicAuthentication))
@permission_classes((IsAuthenticated,))
def example_view(request):
	[...]
```

## 인증되지 않고 거부된 요청에 따른 응답
인증되지 않은 요청으로 권한이 거부될 때, 2개의 에러코드가 발생할 수 있다.

- HTTP 401 Unauthorized : `WWW-Authenticate` 헤더가 항상 포함, 클라이언트에 인증 방법 제시한다.
- HTTp 403 Permissions Denied : `WWW-Authenticate` 헤더가 포함되지 않는다. 요청은 성공적으로 인증 됐지만 요청을 수행 할 수있는 권한이 거부 된 경우 사용된다.

## API Reference
### BasicAuthentication
- 이 인증 체계는 username과 password에 대해 HTTP 기본 인증을 사용한다. 하지만 기본 인증은 일반적으로 테스트에 적합하다.
- 인증을 성공하면, `request.user`는 `장고 User 인스턴스`가 될 것이고, `request.auth`는 `None`이 될 것이다.  
- 인증되지 않은 요청에 대하여 권한이 거부되면 HTTP 401 Unauthorized 응답을 WWW-Authenticate 헤더와 같이 돌려받는다.

```
WWW-Authenticate: Basic realm="api"
```

### TokenAuthentication

> **토큰 기반 인증을 왜 사용할까? : 프론트엔드 프로젝트와 백엔드의 독립적인 개발**
>
- 토큰 기반 인증에서 `토큰`은 요청 `헤더`를 통해 전달된다. 이를 `Stateless`라고 하는데, HTTP 요청을 만들 수 있는 클라이언트라면 누구든지 서버로 요청을 보낼 수 있다는 것을 의미한다. 
- 대부분의 웹 어플리케이션 내에서 `View`는 백엔드에서 렌더링이 되고, 그 결과가 브라우저로 반환된다. 이는 프론트엔드의 로직이 백엔드 코드와 의존성이 있다는 것을 의미하는데, 
- 토큰 기반 인증에서 백엔드 코드는 렌더링된 HTML 대신 `JSON 응답을 반환`할 것이고, 프론트엔드는 반환된 HTML을 받아쓰는 것이 아닌, 경량화되고 압축된 버전의 독립적인 코드를 `CDN`에 넣어둘 수 있게된다. 
- 사용자가 웹 페이지에 접속하면 HTML 컨텐츠는 CDN에서 제공되고, 페이지 내용은 인증 헤더의 토큰을 사용하는 API 서버에 의해 생성되어 프론트엔드와 백엔드의 독립적인 운용을 가능케한다.
- 토큰 기반 인증에서 토큰은 헤더 내에 포함되기 때문에 CSRF를 방지할 수 있다.

**1. 토큰 인증 체계를 사용하기 위해서는 `settings`의 `INSTALLED_APPS 설정`에 `rest_framework.authtoken`을 추가해야 한다.**

```
INSTALLED_APPS = (
    [...]
    'rest_framework.authtoken'
)
```

**2. rest_framework.authtoken 앱은 `Django 데이터베이스 마이그레이션을 제공`하기 때문에 설정을 변경한 후에 `migrate를 실행`해야 한다.**

![](./images/after_add_authtoken_migration.png)

- python manage.py migrate 명령어 실행

![](./images/authtoken_running_migrations.png)

- migrate 실행 후 아래와 같이 데이터베이스 테이블이 생성된 것을 확인할 수 있다. 

![](./images/db_table_list.png)

- Token 테이블의 필드들

![](./images/db_authtoken_token_fields.png)

**3. 토큰 생성하기**

유저에 대하여 토큰을 생성해야하는데, **MyUser 모델에 토큰을 생성하는 함수를 정의** 하고 로그인 시 새로운 유저라면 토큰을 생성하고, 기존 유저라면 기존 토큰 값을 가져오도록 하려고 한다.

[MyUser 모델]

```python
class MyUser(AbstractUser):
	[...]
	
    def get_user_token(self, user_pk):
    	return Token.objects.get_or_create(user_id=user_pk)
```

[Login 시리얼라이저]

```python
class LoginSerializer(serializers.Serializer):
    """ 로그인 """
    username = serializers.CharField(max_length=36, write_only=True)
    password = serializers.CharField(max_length=64, write_only=True)
    user_type = serializers.CharField(max_length=1, )

    def validate(self, data):
        [...]
        
        token, created = user.get_user_token(user.pk)

        ret = {
            'token': token.key,
            'user': {
                'user_pk': user.pk,
                'username': username,
                [...]
            }
        }
        return ret
```

[Login 뷰]

```python
class LoginView(APIView):
    """ 로그인 """

    def post(self, request, format=None):
        serializer = LoginSerializer(data=request.data)
        if serializer.is_valid(raise_exception=True):
            ret = serializer.validated_data
        return Response(ret)
```

포스트맨으로 로그인 실행 예

![](./images/login_token_value.png)

- 성공적으로 인증이 이뤄졌다면, `request.user`는 `장고 유저 인스턴스`가 될 것이고 `request.auth`는 `rest_framework.authtoken.models.Token의 인스턴스`가 될 것이다.

**4. 클라이언트가 인증하려면 토큰 키가 인증 HTTP 헤더에 포함되어야한다. 키에는 두 문자열을 공백으로 구분하여 문자열 리터럴 "Token"을 접두어로 사용해야한다.**

```
Authorization: Token 9944b09199c62bcf9418ad846dd0e4bbdfc6ee4b
```

**5. View에서 토큰 기반 인증을 사용하기 위해서는 Settings에 토큰 기반  인증을 사용할 것임을 선언해야 한다. 그렇지 않다면 Header에 아무리 토큰을 실어도 request.user가 AnonymousUser로 인식된다.**

```
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework.authentication.TokenAuthentication',
    )
}
```

**6. 인증이 필요한 뷰 실행**

![](./images/register_tutor.png)

- 헤더에 토큰을 실지 않으면 MyUser가 필요한 코드를 실행할 때, 'MyUser matching query does not exist' 오류를 뿜뿜한다. 
- 토큰 인증을 기반으로 유저를 인증하는 것인데 토큰이 존재하지 않는 것이면 당연히 유저 또한 MyUser 테이블에 존재하지 않는 것이기 때문이다.

### SessionsAuthentication
- 세션 인증 체계는 장고 기본 세션 백엔드를 사용한다.
- 세션 인증은 웹 페이지와 동일한 Session Context에서 동작하는 AJAX 클라이언트 적합하다. (?)
- 성공적으로 인증이 이뤄졌다면, `request.user`는 `장고 유저 인스턴스`가 될 것이고 `request.auth`는 `None`이 될 것이다.
- 인증되지 않은 요청에 대하여 권한이 거부되면, HTTP 403 Forbidden 응답을 돌려받는다.
- 만약 세션 인증을 AJAX 스타일의 API와 사용한다면, PUT, PATCH, POST 또는 DELETE 요청과 같은 안전하지 않은 HTTP 메서드 호출을 할 경우 CSRF token을 포함시켜야 한다.

[...]

## 참고자료
- [Django rest framework 공식문서 - Authentication](http://www.django-rest-framework.org/api-guide/authentication/)
