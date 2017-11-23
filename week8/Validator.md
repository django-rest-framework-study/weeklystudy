## Validator

- ### Refer

  - ##### [Validators_DRFDocs](http://www.django-rest-framework.org/api-guide/validators/)

  ​


- ### Validation in REST framework

  - ##### DRF Serializer의 validation은 Django의 `ModelForm`과는 조금 다르게 처리된다. `ModelForm` 에서는 폼과, 모델 인스턴스에서 부분적으로 수행된다면, DRF에서는 전체적으로 serializer class에서 수행된다.

    - 적절한 구분을 통해 코드가 명확해진다.
    -  `ModelSerializer class`와 `Serializer class`사이의 변환이 자유롭고,  `ModelSerializer`에 사용되는 모든 유효성 검사들도 간단하게 복제가 가능하다.
    - serializer instance의 `repr`를 출력해보면 유효성 검사 규칙들이 명확하게 표시된다. model instance에 추가적으로 숨겨진 유효성 검사는 호출되지 않는다.

  - ##### Example

    - unique 필드를 가진 모델 클래스를 만들고 기본적인 ModelSerializer를 생성한다.

    ```python
    class CustomerReportRecord(models.Model):
        time_raised = models.DateTimeField(default=timezone.now, editable=False)
        reference = models.CharField(unique=True, max_length=20)
        description = models.TextField()
    ```

    - `CustomertReportRecord`

    ```python
    class CustomerReportSerializer(serializers.ModelSerializer):
        class Meta:
            model = CustomerReportRecord
    ```

    - `django shell`을 통해 `repr`을 확인해보면, `reference` field에 명시적인 validator가 적용돼있는 것이 확인 가능하다.

    ```python
    >>> from project.example.serializers import CustomerReportSerializer
    >>> serializer = CustomerReportSerializer()
    >>> print(repr(serializer))
    CustomerReportSerializer():
        id = IntegerField(label='ID', read_only=True)
        time_raised = DateTimeField(read_only=True)
        reference = CharField(max_length=20, validators=[<UniqueValidator(queryset=CustomerReportRecord.objects.all())>])
        description = CharField(style={'type': 'textarea'})
    ```

    ​


- ### UniqueValidator

  - ##### 이 validator는 모델 필드에  `unique=True`제약 조건을 적용하고 싶을 경우 사용할 수 있다.

    - `queryset` - 고유성을 적용 해야하는 쿼리셋(필수 필드)
    - `message` - 유효성 검사가 실패 할 때 사용하는 오류 메시지
    - `lookup`- 유효성이 검증 된 값으로 기존 인스턴스를 찾는 데 사용되는 조회. `default=exact`

  - ##### 다음과 같이 serializer field에 적용한다.

  ```python
  from rest_framework.validators import UniqueValidator

  slug = SlugField(
      max_length=100,
      validators=[UniqueValidator(queryset=BlogPost.objects.all())]
  )
  ```



- ### UniqueTogetherValidator

  - ##### 이 validator는 모델 인스턴스에 `unique_together`제약 조건을 적용할 수 있다.

    - `queryset` - 고유성을 적용해야하는 쿼리셋(필수 필드)
    - `fields` - serializer filed중 `unique_set`으로 사용할 field 명시(필수 필드)
    - `message` - 유효성 검사가 실패 할 때 사용하는 오류 메시지

  ```python
  from rest_framework.validators import UniqueTogetherValidator

  class ExampleSerializer(serializers.Serializer):
      # ...
      class Meta:
          # ToDo items belong to a parent list, and have an ordering defined
          # by the 'position' field. No two items in a given list may share
          # the same position.
          validators = [
              UniqueTogetherValidator(
                  queryset=ToDoItem.objects.all(),
                  fields=('list', 'position')
              )
          ]
  ```

  - ##### Note:`UniqueTogetherValidation`는  `default`속성이 있는 필드를 제외하고, 모든 필드가 항상 `required` 하도록 처리한다.

  ​

- ### UniqueForDateValidator

- ### UniqueForMonthValidator

- ### UniqueForYearValidator

  - ##### 이 validator는 모델 인스턴스에  `unique_for_date`, `unique_for_month`그리고 `unique_for_year`제약 조건을 적용할 수 있다.

    - `queryset` - 고유성을 적용해야하는 쿼리셋(필수 필드)
    - `field` - serializer filed중 지정된 날짜 범위의 고유성에 대한 유효성을 검사 할 필드 (필수 필드)
    - `date_field` - serializer filed 중 고유성 제한 조건의 날짜 범위를 결정하는 데 사용할 필드 (필수 필드)
    - `message` - 유효성 검사가 실패 할 때 사용하는 오류 메시지

  ```python
  from rest_framework.validators import UniqueForYearValidator

  class ExampleSerializer(serializers.Serializer):
      # ...
      class Meta:
          # Blog posts should have a slug that is unique for the current year.
          validators = [
              UniqueForYearValidator(
                  queryset=BlogPostItem.objects.all(),
                  field='slug',
                  date_field='published'
              )
          ]
  ```

  - ##### 유효성 검사에 사용되는 날짜 필드는 항상 `required`하고, 모델의 `default=...`에만 의존 할 수 없기에 몇 가지 방법이 존재.

    1. ##### 쓰기 가능한 날짜 필드와 함께 사용

       - `default`를 설정하거나, `required=True`를 명시한다.

       ```python
       published = serializers.DateTimeField(required=True)
       ```

    2. ##### 읽기 전용 날짜 필드와 함께 사용

       - `read_only=True`와 `default`값을 명시한다.

       ```python
       published = serializers.DateTimeField(read_only=True, default=timezone.now)
       ```

    3. ##### 숨겨진 날짜 필드와 함께 사용

       - ##### `HiddenField`를 사용하되, serializer에서 `default` 값으로 `validated_data`을 항상 반환하게 한다

       ```python
       published = serializers.HiddenField(default=timezone.now)
       ```


  - ##### Note:`UniqueFor<Range>Validation`는  `default`속성이 있는 필드를 제외하고, 모든 필드가 항상 `required` 하도록 처리한다.

  ​

- ### Advanced field defaults

  - ##### 가끔 client에서 제공 받으면 안되는 데이터가 있는데, validator의 input으로 허용 가능하다. 보통 두가지 방법이 있는데, 

    1. `HiddenField`사용하기, 이 필드는 `validated_data`에는 나타나지만 serializer representation으로는 사용되지 않는다.
    2. `read_only=True`와 `default`인수를 같이 사용한다. 이 필드는 representation으로도 사용되지만 사용자가 직접 설정할 수 는 없다.

  - ##### CurrentUserDefault

    - 현재 사용자를 나타내는 데 사용할 수 있는 기본 클래스. 이것을 사용하려면 serializer를 인스턴트화 시킬 때 `request`가 context에 포함되어 있어야 한다.

  ```python
  owner = serializers.HiddenField(
      default=serializers.CurrentUserDefault()
  )
  ```

  ​

  - ##### CreateOnlyDefault

    - `default` 인수를 설정하는데 사용되는 클래스로,  `create`시에만 작동한다. `update` 시에는 생략.

  ```python
  created_at = serializers.DateTimeField(
      read_only=True,
      default=serializers.CreateOnlyDefault(timezone.now)
  )
  ```

  ​

- ### LImitations of validators

  - ##### `ModelSerializer`에서 자동 생성된 validator를 사용하는 대신 serializer에서 명시적으로 유효성 검사를 처리해야 할 때가 있다. 이 때는 serializer의 `Meta.validators` 속성을 비워 자동 생성 된  validator를 사용하지 않도록 할 수 있다.

  ​

- ### Optional fields

  - ##### 기본적으로 `unique_together`를 사용하는 경우 모든 필드는 `required=True`가 적용된다. 만약 하나의 필드만 `required=False`를 명시적으로 적용한다면 유효성 검사가 제대로 이루어지지 않을 수 있다.

  - ##### 이 경우 시리얼라이저 클래스의 validator를 제외시키고, .`validate()`메서드나 다른 뷰에서, 논리적으로 유효성 검사를 진행 해야한다.

  ```python
  class BillingRecordSerializer(serializers.ModelSerializer):
      def validate(self, data):
          # Apply custom validation either here, or in the view.

      class Meta:
          fields = ('client', 'date', 'amount')
          extra_kwargs = {'client': {'required': 'False'}}
          validators = []  # Remove a default "unique together" constraint.
  ```

  ​

- ### Updating nested serializers

  - ##### 현재 인스턴스를 업데이트 할 때 고유성 검사기는 현재 인스턴스를 제외하는데, 이 때 `instance=…` 속성을 이용해 적용할 수 있다.

    ##### but, 중첩된 serializer의 대한 업데이트 작업의 경우 인스턴스를 사용할 수 없으므로, 고유성 검사를 적용할 방법이 없다.

    ##### 하여 중첩된 serializer에 고유성 검사를 적용하려 한다면, 명시적으로 `.validator()`메서드나 뷰에 코드를 작성 하면된다.



- ### Debugging complex cases

  - ##### `ModelSerializer`class가 정확히 어떤 동작을 하는지 모르는 경우 `manage.py shell`을 실행하여,  serializer의 instance를 출력해보는게 좋다.

    ```python
    >>> serializer = MyComplexModelSerializer()
    >>> print(serializer)
    class MyComplexModelSerializer:
        my_fields = ...
    ```

  - ##### 또한 복잡한 경우 `ModelSerializer` 대신 명시적인 `Serializer class`를 사용하는게 더 나을 수 있다. 

  ​

- ### Writing custom validators

  - ##### Function Based

    - ##### `serializers.ValidationError`실패시 발생시키는 함수.

    ```python
    def even_number(value):
        if value % 2 != 0:
            raise serializers.ValidationError('This field must be an even number.')
    ```
    - ##### Field-level validation

      - 하위 클래스에 `.validate_<field_name>`메소드를 추가하여 사용자 정의 필드 레벨 유효성 검증을 지정할 수 있습니다 `Serializer`. 이 문제는 [Serializer 문서에](http://www.django-rest-framework.org/api-guide/serializers/#field-level-validation) 설명되어 있습니다.

  - ##### Class Based

    - ##### 클래스 기반 validator를 작성하려면 `__call__`메서드를 사용하면 된다. 

    ```python
    class MultipleOf(object):
        def __init__(self, base):
            self.base = base

        def __call__(self, value):
            if value % self.base != 0:
                message = 'This field must be a multiple of %d.' % self.base
                raise serializers.ValidationError(message)
    ```

    - ##### Using `set_context()`

      - 일부 고급 예제에서는 유효성 검사기가 추가 컨텍스트로 사용되는 serializer 필드를 전달할 수 있습니다. `set_context`클래스 기반 유효성 검사기 에서 메서드를 선언하여 그렇게 할 수 있습니다 .

    ```python
    def set_context(self, serializer_field):
        # Determine if this is an update or a create operation.
        # In `__call__` we can then use that information to modify the validation behavior.
        self.is_update = serializer_field.parent.instance is not None
    ```

    ​