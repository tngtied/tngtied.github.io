---
layout: post
categories: [Django]
title: MTV 개발에 있어서 중복되는 응답 데이터 처리하기
author: tngtied
date: 2024-06-06
---

최근 나는 [jump-to-django](https://wikidocs.net/book/4223)를 참고하며 커뮤니티 어플리케이션을 개발하고 있다. 주요 기능으로는 게시글 목록 조회, 댓글 작성, 질문 글에 답변 작성, 추천 순으로 답글 정렬, 검색, 회원가입, 회원 정보 조회, 최근 댓글 및 답글 조회를 구현하였다.

또한 게시물에 카테고리 속성을 넣고 카테고리마다 그에 속한 게시물들을 조회할 수 있는 기능도 구현하면서 이 "카테고리에 속한 게시물 조회" 버튼을 네비게이션 바에 추가하고자 하였다. 그러나 카테고리의 리스트란 언제든지 더 추가되거나 제거될 수 있는 객체이기 때문에 동적으로 view에서 template으로 주입해주어야 하였다. 네비게이션 바의 경우 모든 템플릿에 있어서 들어가는 요소이므로, 모든 뷰에서 네비게이션 바에 들어가는 카테고리 리스트라는 정보를 조회하고 context에 집어넣어주어야 한다는 점에서 매 view 함수마다 코드 중복성이 발생하게 되었다.

{% raw %}

```html
{% for category in category_list %}
<li class="nav-item">
  <a class="nav-link" href="{% url 'common:category' category.id %}"
    >{{ category.name }}</a
  >
</li>
{% endfor %}
```

{% endraw %}

_navbar.html의 문제가 되는 코드_

해당 코드 중복성을 해결하기 위해 나는 데코레이터를 작성하기로 하였다.

# decorator 작성

데코레이터는 함수를 감싸고, 함수 실행 전후에 있어서 작업을 한다. 처음에는 view 함수로 하여금 render된 HttpResponse 객체 내부의 body나 context를 확인하고 그 안에 카테고리와 관련된 추가 데이터를 삽입하려고 하였다. 그러나 한 번 렌더된 HttpResponse 객체 내부의 데이터에는 접근할 수 없었다.

그럼에 따라, 해당 문제를 해결하기 위해 나는 원본 함수는 데이터의 딕셔너리만 반환하고 render 함수를 decorator에서 실행하도록 코드를 짜는 방향을 선택하였다.

## view 함수

```python
@render_with_common
def recent_comment(request):
    comment_list = Comment.objects.order_by('-create_date')
    paginator = Paginator(comment_list, 10)
    page_object = paginator.get_page(1)
    context = {'comment_list': page_object}
    return {'context': context, 'template': 'pybo/recent_comment.html'}
    # return render(request, 'pybo/recent_comment.html', context)
```

각주처리된 return 값은 해당 작업 이전에 리턴되던 값이다. 이 대신에, context와 template을 key로 가지는 딕셔너리를 리턴하도록 변경하였다.

## 데코레이터

```python
def render_with_common(func):
    @wraps(func)
    def wrapper(request, *args, **kwargs):
        request_data = func(request, *args, **kwargs)
        if isinstance(request_data, dict):
            additional_data = {'category_list': Category.objects.all()}
            request_data['context'].update(additional_data)
            return render(request, request_data['template'], request_data['context'])
        return request_data
    return wrapper
```

view 함수의 리턴값 중 `render()`가 아닌 `redirect()`를 사용하는 경우도 존재하므로, 해당 부분을 분기로 확인하기 위해 `if isinstance(request_data, dict):`로 리턴값의 타입을 확인해주고, context 키의 값인 딕셔너리에 추가적인 데이터를 update해주고 렌더해서 리턴하도록 변경하였다.

## @wraps

해당 데코레이터 내부에 사용된 `@wraps` 데코레이터는 원본 함수의 메타데이터들, 이를테면 `__module__`, `__name__`, `__qualname__`, `__annotations__`, `__type_params__`, `__doc__` 같은 attr들을 wrapped 함수에 복사해주며, `__wrapped__` attribute에 원본 함수를 저장한다.

### @wraps의 구현

기존 메타데이터 attribute들을 복사하는 것은 `@wraps`를 구성하는 `update_wrapper()`의 기능이며, `@wraps`는 `update_wraps()`를 데코레이터로 호출하며 wrapper function을 `functools.partial`을 사용하여 정의내린다. 이 두 함수를 사용하는 `@wraps` 데코레이터는 `partial(update_wrapper, wrapped=wrapped, assigned=assigned, updated=updated)`와 동일한 의미를 지니는데, 해당 코드는 update_wrapper 함수의 `wrapped`인자, `assigned`인자, `updated`인자를 넣어서 "부분적으로" 완성시키는 것을 의미한다.
