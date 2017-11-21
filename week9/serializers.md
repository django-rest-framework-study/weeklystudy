# [Serializers](http://www.django-rest-framework.org/api-guide/serializers/#serializers)

 Serializer란?? 

처음 듣는 용어. 직렬 변환기? 

>  In computer science, in the context of data storage, serialization is the process of translating data structures or object state into a format that can be stored (for example, in a file or memory buffer) or transmitted (for example, across a network connection link) and reconstructed later (possibly in a different computer environment).[1]

serializers: 쿼리셋, 모델 인스턴스 등의 복잡한 데이터를 JSON, XML 등의 컨텐트 타입으로 쉽게 변환 가능한 python datatype으로 변환시켜줌

deserialization: 받은 데이터를 validating 한 후에 parsed data를 complex type으로 다시 변환

1. Declaring Serializers - 선언

Comment object example

```python
from datetime import datetime

class Comment(object):
    def __init__(self, email, content, created=None):
    self.email = email
    self.content = content
    self.created = created or datetime.now()
   
comment = Comment(email='leila@example.com', content='foo bar')
```

데이터를 serialize, deserialize 하기 위해 serializer를 선언, form 선언하는것과 비슷하게 serializer를 선언

```python
from rest_framework import serializers

class CommentSerializer(serializers.Serializer):
    email = serializers.EmailField()
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
```



## [1. Serializing objects](http://www.django-rest-framework.org/api-guide/serializers/#serializing-objects)

위에서 만든 CommentSerializer를 통해 comment (list)를 serialize 할 수 있다. (form class 이용과 흡사하게)

```python
serializer = CommentSerializer(comment)
serializer.data
# {'email': 'leila@example.com', 'content': 'foo bar', 'created': '2016-01-27T15:17:10.375877'}
```

=> 모델 인스턴스를 python native datatypes로 변환. 최종적으로 data를 json 타입으로 변환

```python
from rest_framework.renderers import JSONRenderer

json = JSONRenderer().render(serializer.data)
json
# b'{"email":"leila@example.com","content":"foo bar","created":"2016-01-27T15:17:10.375877"}'
```

## 2. [Deserializing objects](http://www.django-rest-framework.org/api-guide/serializers/#deserializing-objects)

deserialization도 비슷하게. 

```python
from django.utils.six import BytesIO
from rest_framework.parsers import JSONParser

stream = BytesIO(json)
data = JSONParser().parse(stream)
```

validaed data를 저장

```python
serializer = CommentSerializer(data=data)
serializer.is_valid()
# True
serializer.validated_data
# {'content': 'foo bar', 'email': 'leila@example.com', 'created': datetime.datetime(2012, 08, 22, 16, 20, 09, 822243)}
```

## [3. Saving instances](http://www.django-rest-framework.org/api-guide/serializers/#saving-instances)

create()나 update() 메소드를 이용해서 object instance를 반환 가능. (Optional)

```python
class CommentSerializer(serializers.Serializer):
    email = serializers.EmailField()
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()

    def create(self, validated_data):
        return Comment(**validated_data)

    def update(self, instance, validated_data):
        instance.email = validated_data.get('email', instance.email)
        instance.content = validated_data.get('content', instance.content)
        instance.created = validated_data.get('created', instance.created)
        return instance
```

 이후 data를 deserializing 할 때 save()를 호출해서 저장 가능. save 는 instance가 존재하면 update를, 그렇지 않으면 create를 해줌.

- save()에 추가적인 atrribute 사용 가능

 `serializer.save(owner=request.user)`

- save()를 직접 overriding 가능

```python
class ContactForm(serializers.Serializer):
    email = serializers.EmailField()
    message = serializers.CharField()

    def save(self):
        email = self.validated_data['email']
        message = self.validated_data['message']
        send_email(from=email, message=message)
```

## 4. [Validation](http://www.django-rest-framework.org/api-guide/serializers/#validation)

data를 deserializing 할 때, instance를 저장하기 전항상 `is_valid()`를 호출해야함

에러가 발생하면 .errors로 에러 메세지 호출 가능. `{'field name': ['error message']}` 형식으로.

- `Raise_exception`: is_valid() 메소드에 optional로 raise_exception 

True일 경우 REST framework가 제공하는 기본 exception handler => 400에러 반환

```python
# Return a 400 response if the data was invalid.
serializer.is_valid(raise_exception=True)
```

- Field-level validation: 각 field별로 validate 생성

`.validate_<field_name>` 형태로 필드별 validation 가능

```python
from rest_framework import serializers

class BlogPostSerializer(serializers.Serializer):
    title = serializers.CharField(max_length=100)
    content = serializers.CharField()

    def validate_title(self, value):
        """
        Check that the blog post is about Django.
        """
        if 'django' not in value.lower():
            raise serializers.ValidationError("Blog post is not about Django")
        return value
```

- Object-level validation: object 전역에 validate

multiple 필드에 validation이 필요하면 subclass로 .validate() 사용,

data를 arg로 받음=> field value dictionary. validate해서 ValidationError를 반환하거나 data 반환

```python
from rest_framework import serializers

class EventSerializer(serializers.Serializer):
    description = serializers.CharField(max_length=100)
    start = serializers.DateTimeField()
    finish = serializers.DateTimeField()

    def validate(self, data):
        """
        Check that the start is before the stop.
        """
        if data['start'] > data['finish']:
            raise serializers.ValidationError("finish must occur after start")
        return data
```



- validator: 따로 validator를 만들어서 field 정의시 삽입

```python
def multiple_of_ten(value):
    if value % 10 != 0:
        raise serializers.ValidationError('Not a multiple of ten')

class GameRecord(serializers.Serializer):
    score = IntegerField(validators=[multiple_of_ten])
    ...
```

 => Meta에 넣어서 완성된 field data에 적용시킬수도 있음

```python
class EventSerializer(serializers.Serializer):
    name = serializers.CharField()
    room_number = serializers.IntegerField(choices=[101, 102, 103, 201])
    date = serializers.DateField()

    class Meta:
        # Each room only has one event per day.
        validators = UniqueTogetherValidator(
            queryset=Event.objects.all(),
            fields=['room_number', 'date']
        )
```

## 5. [Accessing the initial data and instance](http://www.django-rest-framework.org/api-guide/serializers/#accessing-the-initial-data-and-instance)

initial_data로 initial data 사용 가능. 없다면 None 반환

## 6. [Partial updates](http://www.django-rest-framework.org/api-guide/serializers/#partial-updates)

default로 모든 required fields를 넣어주지 않으면 validation error. 

partial arg를 통해서 업데이트 가능

```python
# Update `comment` with partial data
serializer = CommentSerializer(comment, data={'content': u'foo bar'}, partial=True)
```

## 7. [Dealing with nested objects](http://www.django-rest-framework.org/api-guide/serializers/#dealing-with-nested-objects)

다른 serializer class를 field로 받을 수 있음

```python
class UserSerializer(serializers.Serializer):
    email = serializers.EmailField()
    username = serializers.CharField(max_length=100)

class CommentSerializer(serializers.Serializer):
    user = UserSerializer()
    content = serializers.CharField(max_length=200)
    created = serializers.DateTimeField()
```

 required=False, many=True 로 사용 가능

## 8. [Writable nested representations](http://www.django-rest-framework.org/api-guide/serializers/#writable-nested-representations)

nested된 serializer data에서 error 발생시 nested된 field name으로 나옴. validated_data도 마찬가지

```python
serializer = CommentSerializer(data={'user': {'email': 'foobar', 'username': 'doe'}, 'content': 'baz'})
serializer.is_valid()
# False
serializer.errors
# {'user': {'email': [u'Enter a valid e-mail address.']}, 'created': [u'This field is required.']}
```

```python
class UserSerializer(serializers.ModelSerializer):
    profile = ProfileSerializer()

    class Meta:
        model = User
        fields = ('username', 'email', 'profile')

    def create(self, validated_data):
        profile_data = validated_data.pop('profile')
        user = User.objects.create(**validated_data)
        Profile.objects.create(user=user, **profile_data)
        return user
```

- nested representation의 .update() medtod

update인 경우에는 조금 더 문제갸복잡하다. 만약에 관계가 있는 field 가 None일 경우에, 

1. relationship을 db에서 NULL처리?
2. 연관된 istance를 삭제?
3. 데이터를 무시하고 instance를 그대로 놔두기?
4. validation error?

```python
 def update(self, instance, validated_data):
        profile_data = validated_data.pop('profile')
        # Unless the application properly enforces that this field is
        # always set, the follow could raise a `DoesNotExist`, which
        # would need to be handled.
        profile = instance.profile

        instance.username = validated_data.get('username', instance.username)
        instance.email = validated_data.get('email', instance.email)
        instance.save()

        profile.is_premium_member = profile_data.get(
            'is_premium_member',
            profile.is_premium_member
        )
        profile.has_support_contract = profile_data.get(
            'has_support_contract',
            profile.has_support_contract
         )
        profile.save()

        return instance
```

=> create, update가 모호하고 related model간에는 복잡한 의존도가 필요하기 때문에 REST에서는 항상 method를 명시적으로 사용 해야함.

- Handling saving related instance in model manager class

related instance를 여러 개 저장하는 다른 방법은 custom model manager를 사용하는 방법

```python
class UserManager(models.Manager):
    ...

    def create(self, username, email, is_premium_member=False, has_support_contract=False):
        user = User(username=username, email=email)
        user.save()
        profile = Profile(
            user=user,
            is_premium_member=is_premium_member,
            has_support_contract=has_support_contract
        )
        profile.save()
        return user
```

위에서 create를 재정의 한 후에, 

```python
def create(self, validated_data):
    return User.objects.create(
        username=validated_data['username'],
        email=validated_data['email']
        is_premium_member=validated_data['profile']['is_premium_member']
        has_support_contract=validated_data['profile']['has_support_contract']
    )
```

 이런 식으로 model manager에서 정의한 create를 호출해서 사용 가능.

자세한 내용은 django model manager 문서 참조

## 9. [Dealing with multiple objects](http://www.django-rest-framework.org/api-guide/serializers/#dealing-with-multiple-objects)

serializer는 object들의 list도 serializing / deserializing 가능

- 여러 objects serializing

`many=True` flag를 추가. 쿼리셋이나 리스트를 serializing 가능

```python
queryset = Book.objects.all()
serializer = BookSerializer(queryset, many=True)
serializer.data
# [
#     {'id': 0, 'title': 'The electric kool-aid acid test', 'author': 'Tom Wolfe'},
#     {'id': 1, 'title': 'If this is a man', 'author': 'Primo Levi'},
#     {'id': 2, 'title': 'The wind-up bird chronicle', 'author': 'Haruki Murakami'}
# ]
```

- deserializing

multiple create는 가능, update는 불가능. 아래 ListSerializer에서 자세히 설명

## 10. [Including extra context](http://www.django-rest-framework.org/api-guide/serializers/#including-extra-context)

serializer를 처음 만들 때 context arg로 다른 context를 추가 가능하다. 

```python
serializer = AccountSerializer(account, context={'request': request})
serializer.data
# {'id': 6, 'owner': u'denvercoder9', 'created': datetime.datetime(2013, 2, 12, 09, 44, 56, 678870), 'details': 'http://example.com/accounts/6/details'}
```





------

# [ModelSerializer](http://www.django-rest-framework.org/api-guide/serializers/#modelserializer)

serializer를 django model 정의와 비슷하게 사용하고 싶을 때

기본 serializer class와 다른점

- model을 기반으로 fields를 자동 생성
- validator를 자동 생성
- create(), update() 를 디폴트로 선언
- 다음과 같이 쉽게 선언 가능하다.

```python
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ('id', 'account_name', 'users', 'created')
```

 default로 모든 field가 serializer 필드에 매핑됨

- Inspecting `ModelSerializer`

django shell에서 serializer class를 만들고 object repr으로 print 해볼수 있다 

```python
>>> from myapp.serializers import AccountSerializer
>>> serializer = AccountSerializer()
>>> print(repr(serializer))
AccountSerializer():
    id = IntegerField(label='ID', read_only=True)
    name = CharField(allow_blank=True, max_length=100, required=False)
    owner = PrimaryKeyRelatedField(queryset=User.objects.all())
```

## 1. [Specifying which fields to include](http://www.django-rest-framework.org/api-guide/serializers/#specifying-which-fields-to-include)

`ModelForm`에서 하듯이 `fields`나 `exclude` 옵션을 사용해서 원하는 필드만 사용 가능

 fields attribute를 사용해서 모든 field를 명시하는것을 권장
모든 field를 사용할 경우에는 `'__all__'` 사용

## 2. [Specifying nested serialization](http://www.django-rest-framework.org/api-guide/serializers/#specifying-nested-serialization)

기본으로 `ModelSerializer`는 `primary key`를 relationship에 사용

nested representating을 `depth` 옵션을 사용해서 쉽게 생성 가능하다.

```python
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ('id', 'account_name', 'users', 'created')
        depth = 1
```

## 3. [Specifying fields explicitly](http://www.django-rest-framework.org/api-guide/serializers/#specifying-fields-explicitly)

Serializer에서 했던 것처럼 다른 필드를 추가적으로 선언 가능

```python
class AccountSerializer(serializers.ModelSerializer):
    url = serializers.CharField(source='get_absolute_url', read_only=True)
    groups = serializers.PrimaryKeyRelatedField(many=True)

    class Meta:
        model = Account
```

## 4. [Specifying read only fields](http://www.django-rest-framework.org/api-guide/serializers/#specifying-read-only-fields)

각각의 필드에 `read_only=True`라고 추가 하거나,

Meta에서 `read_only_fields` 에서 multiple read only 필드를 정할 수 있음

```python
class AccountSerializer(serializers.ModelSerializer):
    class Meta:
        model = Account
        fields = ('id', 'account_name', 'users', 'created')
        read_only_fields = ('account_name',)
```

> editable=False를 가진 필드나 AutoField는 자동으로 read-only
>
> unique_together의 일부인 field는 모델 레벨의 read-only에 제약이 있음
>
> 이런 경우에는 read_only=True와 default=…를 같이 선언해줘야함

## 5.  [Additional keyword arguments](http://www.django-rest-framework.org/api-guide/serializers/#additional-keyword-arguments)

extra_kwargs 옵션을 이용해서 추가적인 속성 부여 가능

```python
class CreateUserSerializer(serializers.ModelSerializer):
    class Meta:
        model = User
        fields = ('email', 'username', 'password')
        extra_kwargs = {'password': {'write_only': True}}

    def create(self, validated_data):
        user = User(
            email=validated_data['email'],
            username=validated_data['username']
        )
        user.set_password(validated_data['password'])
        user.save()
        return user
```

## 6. [Relational fields](http://www.django-rest-framework.org/api-guide/serializers/#relational-fields)

For full details see the [serializer relations documentation.](http://www.django-rest-framework.org/api-guide/relations/)

## 7. [Customizing field mappings](http://www.django-rest-framework.org/api-guide/serializers/#customizing-field-mappings)

보통 `ModelSerializer`는 따로 fields를 만들어주거나 하지 않아서 명시적으로 추가를 해주거나 일반 `Serializer` 클래스를 사용해야함 

(한번도 써보지 않아서 어떤 상황에서 어떤 방식으로 사용해야할지 잘 모르겠음. 이런게 있다 정도만 알아두고 추후 customizing이 필요시 자세하게 살펴보고 이용해야할듯)

몇몇 경우에 주어진 model 에 대해 특정한 serializer fields를 정의 가능

`.serializer_field_mapping`

=> django model class를 REST framework serializer class와 매핑시켜줌. default serialize class를 원하는 것으로 override 가능

`.serializer_related_field`

=> default는 `PrimaryKeyRelatedField` (`HyperlinkedModelSerializer`인 경우에는 `serializers.HyperlinkedRelatedField`)

`serializer_url_field`

=> serializer의 url 필드에서 사용. default는 `serializers.HyperlinkedIdentityField`

`serializer_choice_field`

serializer의 choice field에서 사용. default는 `serializers.ChoiceField`

### [The field_class and field_kwargs API](http://www.django-rest-framework.org/api-guide/serializers/#the-field_class-and-field_kwargs-api)

다음 method들은 각각의 자동으로 serializer에 포함되는 field들의 kwargs를 정의할때 호출

각각의 method들은 `(field_class, field_kwargs)` 튜플을 반환해야함

`.build_standard_field(self, field_name, model_field)`

=> 전형적인 model field에 매핑되는 serializer field를 생성. default는 `serializer_field_mapping`속성

`.build_relational_field(self, field_name, relation_info)`

=> relational model field에 매핑된 serializer field 생성. default는 `serializer_relational_field` 속성

`relation_info`는 named tuple: `model_field`, `related_model`, `to_many`, `has_through_model` 속성을 포함

`.build_nested_field(self, field_name, relation_info, nested_depth)`

=> depth 옵션이 포함된 relational model field에 매핑된 serializer field를 생성할때 호출. 

`.build_property_field(self, field_name, model_class)`

=> zero-arg method나 속성에 mapping 된 serializer를 생성할때 호출됨. default는 `ReadOnlyField`

`.build_url_field(self, field_nmae, model_class)`

=> serializers의 own url field의 serializer field를 생성할때 호출됨. default는 `HyperlinkedIdentityField`

`.build_unknown_field(self, field_name, model_class)`

=> field name이 모델의 field나 property에 매핑되지 않을 때 호출됨. default로 error가 나게 되어있음. 



# [HyperlinkedModelSerializer](http://www.django-rest-framework.org/api-guide/serializers/#hyperlinkedmodelserializer)

`ModelSerializer`와 비슷, 다만 relationship을 나타낼 때 primary key가 아니라 hyperlink를 사용

default로 serializer는 pk field가 아닌url field를 포함

url field는 `HyperlinkedIdentityField` 로 지정 가능 

```python
# 예시 /api/speech/hyper/

class SpeechHyperlinkSerializers(serializers.HyperlinkedModelSerializer):
  url = serializers.HyperlinkedIdentityField(view_name="speech:detail")
  class Meta:
      model = Speech
      fields = (
          'url',
          'id',
          'lecture_date',
          'memo',
      )
```

## 1. [Absolute and relative URLs](http://www.django-rest-framework.org/api-guide/serializers/#absolute-and-relative-urls)

HyperlinkedModelSerializer를 instantiating 할때는 현재 request를 포함해줘야한다. 이렇게

```python
serializer = AccountSerializer(queryset, context={'request': request})
```

=> 그래야 현재 적합한 hostname을 포함해서 fully qualified URL을 생성해줌

만약 relative URL (`/accounts/1`)을 쓰고 싶다면 `{'request': None}`으로 쓰면 됨

## 2. [How hyperlinked views are determined](http://www.django-rest-framework.org/api-guide/serializers/#how-hyperlinked-views-are-determined)

model instance에 하이퍼링킹 될 때 사용되는 view를 결정해줌

default: `{model_name}-detail` view를 사용 (pk를 kwargs로 받아서)

이를 view name이나 lookup-field 를 사용해서 override 가능: `extra_kwargs`로 추가해주면 됨, 혹은 `HyperlinkedIdentityField` 로 명시 가능

```python
# hyperlinkModelSerializer
class SpeechHyperlinkSerializers(serializers.HyperlinkedModelSerializer):
    # HyperlinkedIdentityField로 view를 지정
    # url = serializers.HyperlinkedIdentityField(
        # view_name="speech:detail",
        # lookup_field="pk",
    # )
    class Meta:
        model = Speech
        fields = (
            'url',
            'id',
            'lecture_date',
            'memo',
        )
        # 혹은 extra_kwargs로 view_name, lookup field 지정 가능
        # 복잡한 경우 이렇게 사용하면 좋을듯
        extra_kwargs = {
            'url': {
                'view_name': 'speech:detail',
                'lookup_field': 'pk',
            }
        }
```

>  Tip: url conf와 매칭하는게 종종 복잡할 때가 있는데 그럴때는 `HyperlinkedModelSerializer` 인스턴스의 `repr`을 print 해보면 명확하게 확인이 가능하다.

```python
In [1]: from speech.api.serializers import SpeechHyperlinkSerializers
In [2]: sr = SpeechHyperlinkSerializers()
In [5]: print(repr(sr))
# SpeechHyperlinkSerializers():
# url = HyperlinkedIdentityField(lookup_field='pk', view_name='speech:detail')
# id = IntegerField(label='ID', read_only=True)
# lecture_date = DateField(label='날짜')
# memo = CharField(allow_blank=True, allow_null=True, label='메모', max_length=128, required=False)
```

## 3. [Changing the URL field name](http://www.django-rest-framework.org/api-guide/serializers/#changing-the-url-field-name)

URL field의 이름은 기본으로 'url'. setting에서 `URL_FIELD_NAME`을 통해 전역으로 override 가능



------

# [ListSerializer](http://www.django-rest-framework.org/api-guide/serializers/#listserializer)

multiple object들을 한 번에 serializing, validating 가능하게 해줌.

보통은 `ListSerializer`를 사용하지 않고 일반 serializer에서 `many=True`를 사용

`many=True`를 사용하면 `ListSerializer` 인스턴스가 생성됨. 즉, `many=True`일 경우나 `ListSerializer`를 사용할 경우나 모두 아래 속성들 사용 가능

`allow_empty`

default로 True , False로 해놓을시 empty list 허용 안됨. 

### 1. [Customizing `ListSerializer` behavior](http://www.django-rest-framework.org/api-guide/serializers/#customizing-listserializer-behavior)

`ListSerializer`를 customize할 few case가 있음, 예를들면

- list에 대한 특정 validation을 하고싶을 때 (리스트 내의 element가 다른 element와 충돌하지 않는다는 등)
- multiple object에 대해 `create`, `update`를 customize 하고싶을 때

이런 상황에서는 `Meta` 클래스에 `list_serializer_class` 옵션을 따로 주면 `many=True`가 있는 것들에 사용됨

```python
class CustomListSerializer(serializers.ListSerializer):
    ...

class CustomSerializer(serializers.Serializer):
    ...
    class Meta:
        list_serializer_class = CustomListSerializer
```

- customizing multiple create

default로 multiple object creation일 경우에도 각각의 리스트 아이템에 `.create()`가 호출됨

이를 customize 하기 위해서는 `ListSerializer` 클래스 내의 `create()`를 커스터마이징 해야함

예를들면 

```python
class BookListSerializer(serializers.ListSerializer):
    def create(self, validated_data):
        books = [Book(**item) for item in validated_data]
        return Book.objects.bulk_create(books)

class BookSerializer(serializers.Serializer):
    ...
    class Meta:
        list_serializer_class = BookListSerializer
```



- customizing multiple update

default로 `ListSerializer` 클래스는 multiple update를 지원하지 않음. `deletion`, `insertion`에 대항 행동들이 모호하기 때문

이를 지원하기 위해서는 so explicitly 하게 사용해야함. 다음을 확실히 하고 사용할것

1. How do you determine which instance should be updated for each item in the list of data?
2. How should insertions be handled? Are they invalid, or do they create new objects?
3. How should removals be handled? Do they imply object deletion, or removing a relationship? Should they be silently ignored, or are they invalid?
4. How should ordering be handled? Does changing the position of two items imply any state change or is it ignored?

= > serializer 인스턴스에 `id` 필드를 명시적으로 추가해줘야함. 기본적으로 `id`필드는 `read_only`인데, 이는 `update`될 때 removed됨. 명시적으로 선언이 되면 `update` 메소드에서 사용 가능

예시 (실제로 update를 이런식으로 사용할 일이 있을지는 의문. 추후 필요할 때 보고 사용하면 될듯)

```python
class BookListSerializer(serializers.ListSerializer):
    def update(self, instance, validated_data):
        # Maps for id->instance and id->data item.
        book_mapping = {book.id: book for book in instance}
        data_mapping = {item['id']: item for item in validated_data}

        # Perform creations and updates.
        ret = []
        for book_id, data in data_mapping.items():
            book = book_mapping.get(book_id, None)
            if book is None:
                ret.append(self.child.create(data))
            else:
                ret.append(self.child.update(book, data))

        # Perform deletions.
        for book_id, book in book_mapping.items():
            if book_id not in data_mapping:
                book.delete()

        return ret

class BookSerializer(serializers.Serializer):
    # We need to identify elements in the list using their primary key,
    # so use a writable field here, rather than the default which would be read-only.
    id = serializers.IntegerField()
    ...

    class Meta:
        list_serializer_class = BookListSerializer
```

multiple update opreation을 도와주는 third party package가 있다고 함  (REST 2버전의 `allow_add_remove`와 비슷한것)

- Customizing ListSerializer initialization

`many=True`일때, `Serializer`, `ListSerializer` 클래스에 어떤 args나 kwargs가 `.__init__()` 메소드에 사용되는지 결정해야함

default는 `validators`와 custom kwargs 를 제외하고는 모두 사용됨. 

(무슨말인지 잘 모르겠음.) 

```Python
@classmethod
def many_init(cls, *args, **kwargs):
    # Instantiate the child serializer.
    kwargs['child'] = cls()
    # Instantiate the parent list serializer.
    return CustomListSerializer(*args, **kwargs)
```





# [BaseSerializer](http://www.django-rest-framework.org/api-guide/serializers/#baseserializer)

(`Serializer`, `ModelSerializer`를 사용하지 않고 실제로 `BaseSerializer`를 사용하여 Serializer를 만드는 경우가 있는지 궁금. )

`Serializer` 클래스와 비슷한 basic API를 가짐

- `.data` - Returns the outgoing primitive representation.
- `.is_valid()` - Deserializes and validates incoming data.
- `.validated_data` - Returns the validated incoming data.
- `.errors` - Returns any errors during validation.
- `.save()` - Persists the validated data into an object instance.

serializer class가 지원하길 원하는 기능들을 override 가능

- `.to_representation()` - read operation을 위한 serialization 지원
- `.to_internal_value()` - write operation을 위한 serialization 지원
- `.create() `and `.update()` 

=> `Serializer` 클래스와 같은 인터페이스, 일반 `Serializer` 나 `ModelSerializer`와 동일하게 사용 가능. 

다른 점은 `BaseSerializer`는 browsable API 내의 HTML form을 만들어주지 않음

이외 read-only, read-write BaseSerializer 클래스나 새로운 `BaseSerializer`의 예시는 필요에 따라 drf 공식문서 를 참고해서 만들면 될듯





# [Advanced serializer usage](http://www.django-rest-framework.org/api-guide/serializers/#advanced-serializer-usage)

## 1. [Overriding serialization and deserialization behavior](http://www.django-rest-framework.org/api-guide/serializers/#overriding-serialization-and-deserialization-behavior)

serialization, deserialization 를 대체해야 할 일이 있을 때 `.to_representation()`이나 `to_internal_value()` 메소드를 overriding 가능. 

이럴때 유용하다

- 새로운 serializer base 클래스에 새로운 behavior를 추가
- 존재하는 class의 behavior를 살짝 바꿀 때
- 많은 양의 데이터를 호출하고 자주 접속되는 API endpoint의 serialization 퍼포먼스 향상

**`.to_representation(self, obj)`**

=> serialization에 필요한 object instance를 받아서 primitive representation을 리턴. (통상적으로 이는 built-in 파이썬 데이터타입) 

drf의 기본 Serializer에는 이렇게 정의

```python
    def to_representation(self, instance):
        """
        Object instance -> Dict of primitive datatypes.
        """
        ret = OrderedDict()
        fields = self._readable_fields

        for field in fields:
            try:
                attribute = field.get_attribute(instance)
            except SkipField:
                continue

            # We skip `to_representation` for `None` values so that fields do
            # not have to explicitly deal with that case.
            #
            # For related fields with `use_pk_only_optimization` we need to
            # resolve the pk value.
            check_for_none = attribute.pk if isinstance(attribute, PKOnlyObject) else attribute
            if check_for_none is None:
                ret[field.field_name] = None
            else:
                ret[field.field_name] = field.to_representation(attribute)

        return ret
```

**`.to_internal_value(self, data)`**

unvalidated data를 인풋으로 받아서 `serializer.validated_data`로 사용 가능한 validated data를 리턴함

만약 `save()`가 호출되면 이 리턴된 value가 `create()`, `update()` method에도 사용

validation이 실패하면, `serializers.ValidationError(errors)` 발생. deserialization behavior을 변경할 필요가 없거나 object-level의 validation을 제공하고 싶으면 그냥 `validate()` 메쏘드를 override 하길 추천

기본 drf 코드

```python
    def to_internal_value(self, data):
        """
        Dict of native values <- Dict of primitive datatypes.
        """
        if not isinstance(data, Mapping):
            message = self.error_messages['invalid'].format(
                datatype=type(data).__name__
            )
            raise ValidationError({
                api_settings.NON_FIELD_ERRORS_KEY: [message]
            }, code='invalid')

        ret = OrderedDict()
        errors = OrderedDict()
        fields = self._writable_fields

        for field in fields:
            validate_method = getattr(self, 'validate_' + field.field_name, None)
            primitive_value = field.get_value(data)
            try:
                validated_value = field.run_validation(primitive_value)
                if validate_method is not None:
                    validated_value = validate_method(validated_value)
            except ValidationError as exc:
                errors[field.field_name] = exc.detail
            except DjangoValidationError as exc:
                errors[field.field_name] = get_error_detail(exc)
            except SkipField:
                pass
            else:
                set_value(ret, field.source_attrs, validated_value)

        if errors:
            raise ValidationError(errors)

        return ret
```

## 2. [Serializer Inheritance](http://www.django-rest-framework.org/api-guide/serializers/#serializer-inheritance)

장고 폼과 비슷하게, `serializer`도 상속 가능. 

```python
class MyBaseSerializer(Serializer):
    my_field = serializers.CharField()

    def validate_my_field(self):
        ...

class MySerializer(MyBaseSerializer):
    ...
```

장고 모델, 모델폼과 같이 `Meta` 클래스는 명시적으로 상속되지 않는다. 상속하고 싶다면 명시해야함  i.e. 

```python
class AccountSerializer(MyBaseSerializer):
    class Meta(MyBaseSerializer.Meta):
        model = Account
```

=> drf 공식 문서에서는 `Meta` 클래스를 상속하지 않는걸 추천. 대신 모든 option을 명시적으로 쓸것

또한 serializer 상속에 몇가지 주의사항

- 일반적인 python name resolution rule 적용: 여러 배이스 클래스에 `Meta` 클래스를 추가하면 하나만 사용됨. child의 `Meta`가 있다면 그걸 사용, 없다면 첫번째 parent의 `Meta` 사용

- `Field`를 `None`으로 정의하여 상속받은 부모 클라스의 필드를 없앨 수 있음`

  ```python
  class MyBaseSerializer(ModelSerializer):
      my_field = serializers.CharField()

  class MySerializer(MyBaseSerializer):
      my_field = None
  ```

  => 하지만 그냥 opt-out 방식을 사용할수 있음 (`fields`, `exclude`)

## 3. [Dynamically modifying fields](http://www.django-rest-framework.org/api-guide/serializers/#dynamically-modifying-fields)

serializer가 생상되면 `.fields` 속성으로 fields dictionary에 접근 가능하다. 

이 fields arg를 수정함으로써 serializer를 선언할 때가 아닌, runtime에서 serializer args를 수정 가능

예시 - initializing 할 때 어떤 필드가 세팅 되는지를 세팅 가능

```Python
class DynamicFieldsModelSerializer(serializers.ModelSerializer):
    """
    A ModelSerializer that takes an additional `fields` argument that
    controls which fields should be displayed.
    """

    def __init__(self, *args, **kwargs):
        # Don't pass the 'fields' arg up to the superclass
        fields = kwargs.pop('fields', None)

        # Instantiate the superclass normally
        super(DynamicFieldsModelSerializer, self).__init__(*args, **kwargs)

        if fields is not None:
            # Drop any fields that are not specified in the `fields` argument.
            allowed = set(fields)
            existing = set(self.fields.keys())
            for field_name in existing - allowed:
                self.fields.pop(field_name)
```

 다음과 같이 initializing 할 때 세팅할 필드를 추가 가능

```python
>>> class UserSerializer(DynamicFieldsModelSerializer):
>>>     class Meta:
>>>         model = User
>>>         fields = ('id', 'username', 'email')
>>>
>>> print UserSerializer(user)
{'id': 2, 'username': 'jonwatts', 'email': 'jon@example.com'}
>>>
>>> print UserSerializer(user, fields=('id', 'email'))
{'id': 2, 'email': 'jon@example.com'}
```





# [Third party packages](http://www.django-rest-framework.org/api-guide/serializers/#third-party-packages)

> 이런것들이 있다 정도만 알아놓고 필요할 때 찾아서 사용하면 될듯

## [Django REST marshmallow](http://www.django-rest-framework.org/api-guide/serializers/#django-rest-marshmallow)

The [django-rest-marshmallow](http://tomchristie.github.io/django-rest-marshmallow/) package provides an alternative implementation for serializers, using the python [marshmallow](https://marshmallow.readthedocs.io/en/latest/) library. It exposes the same API as the REST framework serializers, and can be used as a drop-in replacement in some use-cases.

## [Serpy](http://www.django-rest-framework.org/api-guide/serializers/#serpy)

The [serpy](https://github.com/clarkduvall/serpy) package is an alternative implementation for serializers that is built for speed. [Serpy](https://github.com/clarkduvall/serpy) serializes complex datatypes to simple native types. The native types can be easily converted to JSON or any other format needed.

## [MongoengineModelSerializer](http://www.django-rest-framework.org/api-guide/serializers/#mongoenginemodelserializer)

The [django-rest-framework-mongoengine](https://github.com/umutbozkurt/django-rest-framework-mongoengine) package provides a `MongoEngineModelSerializer` serializer class that supports using MongoDB as the storage layer for Django REST framework.

## [GeoFeatureModelSerializer](http://www.django-rest-framework.org/api-guide/serializers/#geofeaturemodelserializer)

The [django-rest-framework-gis](https://github.com/djangonauts/django-rest-framework-gis) package provides a `GeoFeatureModelSerializer` serializer class that supports GeoJSON both for read and write operations.

## [HStoreSerializer](http://www.django-rest-framework.org/api-guide/serializers/#hstoreserializer)

The [django-rest-framework-hstore](https://github.com/djangonauts/django-rest-framework-hstore) package provides an `HStoreSerializer` to support [django-hstore](https://github.com/djangonauts/django-hstore) `DictionaryField` model field and its `schema-mode` feature.

## [Dynamic REST](http://www.django-rest-framework.org/api-guide/serializers/#dynamic-rest)

The [dynamic-rest](https://github.com/AltSchool/dynamic-rest) package extends the ModelSerializer and ModelViewSet interfaces, adding API query parameters for filtering, sorting, and including / excluding all fields and relationships defined by your serializers.

## [Dynamic Fields Mixin](http://www.django-rest-framework.org/api-guide/serializers/#dynamic-fields-mixin)

The [drf-dynamic-fields](https://github.com/dbrgn/drf-dynamic-fields) package provides a mixin to dynamically limit the fields per serializer to a subset specified by an URL parameter.

## [DRF FlexFields](http://www.django-rest-framework.org/api-guide/serializers/#drf-flexfields)

The [drf-flex-fields](https://github.com/rsinger86/drf-flex-fields) package extends the ModelSerializer and ModelViewSet to provide commonly used functionality for dynamically setting fields and expanding primitive fields to nested models, both from URL parameters and your serializer class definitions.

## [Serializer Extensions](http://www.django-rest-framework.org/api-guide/serializers/#serializer-extensions)

The [django-rest-framework-serializer-extensions](https://github.com/evenicoulddoit/django-rest-framework-serializer-extensions) package provides a collection of tools to DRY up your serializers, by allowing fields to be defined on a per-view/request basis. Fields can be whitelisted, blacklisted and child serializers can be optionally expanded.

## [HTML JSON Forms](http://www.django-rest-framework.org/api-guide/serializers/#html-json-forms)

The [html-json-forms](https://github.com/wq/html-json-forms) package provides an algorithm and serializer for processing `<form>`submissions per the (inactive) [HTML JSON Form specification](https://www.w3.org/TR/html-json-forms/). The serializer facilitates processing of arbitrarily nested JSON structures within HTML. For example, `<input name="items[0][id]" value="5">` will be interpreted as `{"items": [{"id": "5"}]}`.

## [DRF-Base64](http://www.django-rest-framework.org/api-guide/serializers/#drf-base64)

[DRF-Base64](https://bitbucket.org/levit_scs/drf_base64) provides a set of field and model serializers that handles the upload of base64-encoded files.

## [QueryFields](http://www.django-rest-framework.org/api-guide/serializers/#queryfields)

[djangorestframework-queryfields](http://djangorestframework-queryfields.readthedocs.io/) allows API clients to specify which fields will be sent in the response via inclusion/exclusion query parameters.

## [DRF Writable Nested](http://www.django-rest-framework.org/api-guide/serializers/#drf-writable-nested)

The [drf-writable-nested](http://github.com/beda-software/drf-writable-nested) package provides writable nested model serializer which allows to create/update models with nested related data.

