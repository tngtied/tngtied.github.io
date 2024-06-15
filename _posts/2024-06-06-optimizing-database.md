---
layout: post
categories: [Django]
title: 쿼리 최적화하기 - 인덱싱
author: tngtied
date: 2024-06-06
---

[MTV 개발에 있어서 중복되는 응답 데이터 처리하기](https://tngtied.github.io/django/adding-common-information)에서 기술했다시피, [jump-to-django](https://wikidocs.net/book/4223)를 참고하며 커뮤니티 어플리케이션을 개발하고 있다.
그 중, 전체 게시글 목록 조회와 그 과정의 페이지네이션에 있어서 데이터베이스에 부하가 가고 요청 처리 지연이 발생할 수도 있으리라는 판단이 섰고, 이를 확인하기 위해 테스트를 진행하였다.

# 기존의 코드

```python
@render_with_common
def index(request):
    page = request.GET.get('page', '1')
    kw = request.GET.get('kw', '')
    question_list = Question.objects.order_by('-create_date')
    if kw:
        question_list = question_list.filter(
            Q(subject__icontains=kw) |
            Q(content__icontains=kw) |
            Q(answer__content__icontains=kw) |
            Q(author__username__icontains=kw) |
            Q(answer__author__username__icontains=kw)
        ).distinct()

    paginator = Paginator(question_list, 10)
    page_object = paginator.get_page(page)

    context = {'question_list': page_object, 'page': page, 'kw' : kw}
    return {'context': context, 'template': 'pybo/question_list.html'}
```

# 테스트코드 작성

대량의 데이터가 조회되고 페이지네이터가 사용되는 상황을 가정하기 위해서 테스트코드를 작성하고, 해당 작업에 정확히 얼마나 부하가 가는지 확인하기 위해 시간을 측정하는 데코레이터를 작성하였다.

## 시간을 측정하는 데코레이터

```python
def log_time(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        start_time = time.time()
        print(f'[{start_time}] {func.__name__} 함수가 호출되었습니다.')
        request_data = func(*args, **kwargs)
        end_time = time.time()
        print(f'[{end_time}] {func.__name__} 함수가 종료되었습니다. 수행시간: {end_time - start_time}')
        return request_data
    return wrapper
```

데코레이터가 수식하는 함수 실행 이전의 시각을 저장하고, 함수를 실행하고, 그 직후의 시각을 저장하고 빼줌으로써 소요된 시간을 나타낸다.

## 데이터 백만개 추가하기

테스트 클래스를 작성하고, 테스트를 위한 데이터베이스에 필요한 데이터를 집어넣기 위해 처음에는 `SetUp(self)`에 아래와 같은 함수를 작성했다.

```python
def setUp(self):
    cls.author_instance = User.objects.create(
        username='testuser',
        password='1234',
        email = 'test@gmail.com'
    )
    for i in range(1000000):
        self.question_instance = Question.objects.create(
            subject='test subject',
            content='test content',
            author = self.author_instance,
            create_date=timezone.now()
        )

def test_get_question_page_1(self):
    print("test_get_question_page_1")
    response = self.client.get('/?page=1')
    self.assertEqual(response.status_code, 200)

def test_get_question_page_100000(self):
    print("test_get_question_page_100000")
    response = self.client.get('/?page=100000')
    self.assertEqual(response.status_code, 200)
```

그러나 테스트를 실행하는 데에 너무 오랜 시간이 걸렸고, 시간이 걸리는 것에 비해 테스트 수행시간은 1초 위주로 짧게 걸렸다. 어째서 테스트를 실행하는 시간과 측정되는 테스트 시간에 차이가 나는지 디버깅을 해 본 결과는 다음과 같았다.

```python
[1717685157.1376052] setUp 함수가 호출되었습니다.
[1717685341.73755] setUp 함수가 종료되었습니다. 수행시간: 184.5999448299408
test_get_question_page_1
[1717685341.7435606] index 함수가 호출되었습니다.
[1717685342.8645208] index 함수가 종료되었습니다. 수행시간: 1.1209602355957031

[1717685342.9305208] setUp 함수가 호출되었습니다.
[1717685527.3087375] setUp 함수가 종료되었습니다. 수행시간: 184.37821674346924
test_get_question_page_100000
[1717685527.3087375] index 함수가 호출되었습니다.
[1717685529.4687865] index 함수가 종료되었습니다. 수행시간: 2.1600489616394043
```

먼저 `setUp(self)`함수가 test 함수마다 호출되는 것이 보인다. 한 번 테스트 데이터를 생성하고 테스트 함수를 실행하는 게 아니라 매번 다시 데이터를 만들기를 반복하는 것이다. 또한, 테스트를 실행하기 이전에 테스트 데이터를 생성하는 과정에서 3분씩 시간이 소요되고 있음을 알 수 있다.

### 개선

이를 해결하기 위해, 먼저 매 테스트마다 새로이 테스트 데이터가 생성되는 것을 막기 위해 `setUp(self)`대신 `setUpTestData(cls)`를 사용하여 테스트 클래스마다 단 한 번씩만 테스트 데이터가 생성되도록 만들었다. 또한, 객체 생성마다 데이터베이스에 저장하고 커밋하기보다는 한 번에 저장하도록 `bulk_create()`를 사용하도록 변경하였다.

```python
def setUp(self):
    pass

@classmethod
@log_time
def setUpTestData(cls):
    cls.author_instance = User.objects.create(
        username='testuser',
        password='1234',
        email = 'test@gmail.com'
    )
    questions = []
    for i in range(1000000):
        question = Question(
            subject='test subject',
            content='test content',
            author=cls.author_instance,
            create_date=timezone.now()
        )
        questions.append(question)
    Question.objects.bulk_create(questions)
```

이와 같이 개선한 코드를 사용하여 테스트를 돌려본 결과는 다음과 같다.

```python
[1717744612.347144] setUpTestData 함수가 호출되었습니다.
[1717744660.659047] setUpTestData 함수가 종료되었습니다. 수행시간: 48.31190299987793
[1717744660.6592572] setUp 함수가 호출되었습니다.
[1717744660.659265] setUp 함수가 종료되었습니다. 수행시간: 7.867813110351562e-06
test_get_question_page_1
[1717744660.663213] index 함수가 호출되었습니다.
[1717744661.759168] index 함수가 종료되었습니다. 수행시간: 1.0959548950195312



[1717744661.8203242] setUp 함수가 호출되었습니다.
[1717744661.820338] setUp 함수가 종료되었습니다. 수행시간: 1.3828277587890625e-05
test_get_question_page_100000
[1717744661.820729] index 함수가 호출되었습니다.
[1717744663.56142] index 함수가 종료되었습니다. 수행시간: 1.7406909465789795
```

3분이 소요되던 것이 50초로 줄어든 것을 확인할 수 있다.

# 십만번째 페이지 조회에 2초가 소요되는 문제

위의 결과를 확인해보면, 십만번째 페이지의 경우 조회에 2초가 소요되는 것을 확인할 수 있다. 이는 UX관점에서 문제가 된다. 해당 부분을 개선하기 위해 실행되는 쿼리를 확인해보기로 했다.

```SQL
SELECT "pybo_question"."id",
    "pybo_question"."author_id",
    "pybo_question"."subject",
    "pybo_question"."content",
    "pybo_question"."create_date",
    "pybo_question"."modify_date",
    "pybo_question"."category_id",
    "pybo_question"."hits"
FROM "pybo_question"
ORDER BY "pybo_question"."create_date" DESC
```

`pybo_question` 테이블의 `create_date`필드를 기준으로 정렬하는 것을 알 수 있다. 최적화를 위해, 해당 필드를 인덱스로 설정하기로 결정하였다. 게시물의 생성일자의 경우, 게시물이 생성된 이후로는 절대 바뀌지 않는 값이기 때문에, 인덱스 칼럼 값의 수정으로 인한 추가 연산의 비용이 들지 않을 것으로 예상되었기 때문이다.

## Question model 수정

Question model을 다음과 같이 수정해주었다.

```python
class Question(models.Model):
    author = models.ForeignKey(User, on_delete = models.CASCADE, related_name='author_question')
    subject = models.CharField(max_length=200)
    content = models.TextField()
    create_date = models.DateTimeField(db_index=True)
    modify_date = models.DateTimeField(null=True, blank=True)
    voter = models.ManyToManyField(User, related_name='voter_question')
    comment = models.ManyToManyField('Comment', related_name='comment_question')
    category = models.ForeignKey('Category', on_delete = models.SET_NULL, null=True, blank=True)
    hits = models.IntegerField(default=0)
    def __str__(self):
        return self.subject
```

create_date 필드에 `db_index=True` 옵션을 넣어주고 마이그레이션을 진행했다.

## 결과

```python
[1717745068.2115579] setUpTestData 함수가 호출되었습니다.
[1717745116.626152] setUpTestData 함수가 종료되었습니다. 수행시간: 48.4145941734314
[1717745116.626373] setUp 함수가 호출되었습니다.
[1717745116.6263838] setUp 함수가 종료되었습니다. 수행시간: 1.0728836059570312e-05
test_get_question_page_1
[1717745116.6305878] index 함수가 호출되었습니다.
[1717745117.307965] index 함수가 종료되었습니다. 수행시간: 0.6773772239685059



[1717745117.368778] setUp 함수가 호출되었습니다.
[1717745117.368793] setUp 함수가 종료되었습니다. 수행시간: 1.5020370483398438e-05
test_get_question_page_100000
[1717745117.369237] index 함수가 호출되었습니다.
[1717745117.7813902] index 함수가 종료되었습니다. 수행시간: 0.4121532440185547
```

실행 결과, 1.7초가 소요되던 마지막 페이지 조회가 0.4초 소요로 줄어든 것을 확인할 수 있다.

# 결론

적절한 인덱싱은 데이터베이스 성능에 크게 도움이 된다는 것을 직접적으로 경험할 수 있었다.
