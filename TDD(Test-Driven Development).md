# TDD (Test-Driven Development, 테스트 주도 개발)

## 1부

## 2장 unittest 모듈을 이용한 기능 테스트 확장
### Functional test(FT) : 기능 테스트
셀레늄을 이용하 테스트에서는 실제 웹 브라우저를 실행해서 애플리케이션이 어떻게 "동작(functiona)"하는지 사용자 관점에서 확인 할 수 있다.
FT는 사람이 이해 할 수 있는 스토리를 가지고 있어야 한다. 이것을 분명하게 정의하기 위해 테스트 코드에 주석을 기록한다. 프로그래머가 아니더라도 이해할 수 있어야한다.(애플리케이션 요구사항과 특징을 FT를 보고 논의 할 수 있을 정도)

```python
from selenium import webdriver
import unittest

class NewVisitorTest(unittest.TestCase):

    def setUp(self):
        self.browser = webdriver.Chrome()
        self.browser.implicitly_wait(3)

    def tearDown(self):
        self.browser.quit()

    def test_can_start_a_list_and_retrieve_it_later(self):
        # 에디스(Edith)는 멋진 작업 목록 온라인 앱이 나왔다는 소식을 듣고
        # 해당 앱 사이트를 확인하러 간다
        self.browser.get('http://localhost:8000')

        # 웹 페이지 타이틀과 헤더가 'To-Do'를 표시하고 있다
        self.assertIn('To-Do', self.browser.title)

```

#### 부분적 설명

`setUp`와 `tearDown`은 특수한 메소드이다. 각 테스트 시작 전과 후에 실행된다.  
여기선는 브라우저를 시작하고 닫을 때 사용하고 있다. 테스트에 에러가 발생해도 tearDown이 실행된다.

```python
        self.fail('Finish th test!')
        # 강제적으로 테스트 실해를 발생시켜 에러 메세지를 출력한다
        # 테스트가 끝났다는 것을 알리기 위해 사용하고 있다
```


```python
        self.browser.implicitly_wait(3)
        # 셀레늄 테스트의 기본적인 로직이다
        # 페이지 로딩이 끝날때까지 기다렸다가 테스트를 실행한다
        # 하지만 완벽하지 않다
        # 이후에 '명시적인' 대기 알고리즘을 별도로 작성해야한다
```

### 유용한 TDD 개념
#### 사용자 스토리(User story)
사용자 관점에서 어떻게 애플리케이션이 동작해야 하는지  기술한 것이다. 기능 테스트 구조화를 위해 사용한다.

#### 예측된 실패(Expected failure)
의도적으로 구현한 테스트 실패를 의미한다.

## 3장 단위 테스트를 이용한 간단한 홈페이지 테스트

### 단위 테스트와 기능 테스트의 차이
**기능 테스트**는 **사용자 관점에서 애플리게이션 외부**를 테스트하는 것이고, **단위 테스트**는 **프로그래머 관점에서 그 내부**를 테스트한다는 것이다.

#### 작업 순서

1. 기능 테스트를 작성해서 사용자 관점의 새로운 가능성을 정의하는 것부터 시작한다.
2. 기능 테스트가 실패하고 나면 어떻게 코드를 작성해야 테스트를 통과할지(또는 적어도 현재 문제를 해결할 수 있는 방법)을 생각해본다. 이 시점에서 하나 또는 그 이상의 단위 테스트를 이용해서 어떻게 코드가 동작해야 하는지 정의한다(기본적으로 모든 코드가(적어도) 하나 이상의 단위 테스트에 의해 테스트 되어야 한다.)
3. 단위 테스트가 실패하고 나면 단위 테스트를 통과할 수 있을 정도의 최소한의 코드만 작성한다. 기능 테스트가 완전해질 때까지 과정 2,3을 반복해야 할 수도 있다.
4. 기능 테스트를 재실행해서 통과하는지 또는 제대로 동작하는지 확인한다. 이 과정에서 새로운 단위 테스트를 작성해야 할 수도 있다.

이 과정을 보면, 기능 테스트는 상위 레벨의 개발을 주도하고, 단위 테스트는 하위 레벨을 주도한다는 것을 알 수 있다.
기능 테스트와 단위 테스트가 전혀 가른 목적을 가지고 있어서 서로 다른 결과를 초래할 수 있기 때문에 꼭 필요한 과정이다.

기능 테스트는 제대로 된 기능성을 갖춘 애플리케이션을 구축하도록 도우며, 그 기능성이 망가지지 않도록 보장해준다. 반면, 단위 테스트는 깔끔하고 버그 없는 코드를 작성하도록 돕는다.

### Django에서의 단위 테스트
#### Django의 처리 흐름
1. 특정 URL에 HTTP "요청"을 받는다.
2. Django는 특정 규칙을 이용해서 해당 요청에 어떤 뷰 함수를 실행항지 결정한다.
3. 이 뷰 기능이 요청을 처리해서 HTTP "응답"으로 반환한다.

#### 따라서 우리가 테스트해야 할 것

- URL의 사이트 루트("/")를 해석해서 특정 뷰 기능에 매칭시킬 수 있는가?
- 이 뷰 기능이 특정 HTML을 반환하게 해서 기능 테스트를 통과할 수 있는가?

```python
from django.core.urlresolvers import resolve
from django.test import TestCase
from lists.views import home_page

class HomePageTest(TestCase):
    
    def test_root_url_resolves_to_home_page_view(self):
        found = resolve('/')
        self.assertEqual(found.func, home_page)
```
#### 부분적인 설명
resolve는 Django가 내부적으로 사용하는 함수로, URL을 해석해서 일치하는 뷰 함수를 찾는다. 여기서는 "/"(사이트 루트)가 호출될 때 resolve를 실행해서 home_page라는 함수를 호출한다.

### 트레이스백 읽기
트레이스백(Traceback)은 TDD에 있어 중요한 역할을 하는것으로 자주 접하게 된다. 트레이스백을 빠르게 읽어서 필요한 단서를 찾는 방법을 금방 배울수 있다.

```
======================================================================
ERROR: test_root_url_resolves_to_home_page_view (lists.tests.HomePageTest)1
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "/workspace/superlists/lists/tests.py", line 8, in
test_root_url_resolves_to_home_page_view
    found = resolve('/')2
  File "/usr/local/lib/python3.4/dist-packages/django/core/urlresolvers.py",
line 522, in resolve
    return get_resolver(urlconf).resolve(path)
  File "/usr/local/lib/python3.4/dist-packages/django/core/urlresolvers.py",
line 388, in resolve
    raise Resolver404({'tried': tried, 'path': new_path})
django.core.urlresolvers.Resolver404: {'tried': [[<RegexURLResolver3
<RegexURLPattern list> (admin:admin) ^admin/>]], 'path': ''}4
 ---------------------------------------------------------------------
[...]
```


3.4. 먼저 확인해야 할 것이 **에러**다. 이 에러 부분이 핵심인데, 어떤 때는 이 부분만 읽고 바로 문제를 파악할 수 있다. 하지만 이 예제의 경우는 에러만으로는 충분히 문제 파악이 어렵다.
1. 다음으로 확인할 것은 **"어떤 테스트가 실패하고 있는가?"**다. 이 실패가 예측된 실패인지를 확인해야 한다. 이 예에서는 예측된 실패이다.
2. 마지막으로 **실패를 발생시키는 "테스트 코드"를 찾는다.** 트레이스백 상단부터 아래로 내려가면서 테스트 파일명을 찾는다. 그래서 어떤 테스트 함수의 몇 번째 코드 라인에서 실패가 발생하는지 확인한다. 이 예에선 resolve 함수를 호출해서 "/" URL을 해석하는 부분이다.

문제 있는 코드를 확인할때 일반적으로 사용하는 4단계 과정이다.
이렇게 모든 정보를 취합해 트레이스백을 해석한 결과, "/"를 확인하려고 할 때 Django가 404 에러를 발생시키고 있다는 것을 알 수 있다.

#### 부분적인 설명
```python
 def test_home_page_returns_correct_html(self):
        request = HttpRequest()  #1
        response = home_page(request)  #2
        self.assertTrue(response.content.startswith(b'<html>'))  #3
        self.assertIn(b'<title>To-Do lists</title>', response.content)  #4
        self.assertTrue(response.content.endswith(b'</html>'))  #5
```
1. HttpRequest 객체를 생성해서 사용자가 어떤 요청을 브라우저에 보내는지 확인한다.
2. 이것을 home_page 뷰에 전달해서 응답을 취득한다. 이 객체는 HttpResponse라는 클래스의 인스턴스이다. 응답내용(HTML 형태로 사용자에세 보내는것)이 특정 속성을 가지고 있는지 확인한다.
3.5. 그 다음은 응답 내용이 <html>으로 시작하고  </html>으로 끝나는지 확인한다. response.content는 byte형 데이터로, 파이썬 문자열이 아니다. 따라서 b''구문을 사용해서 비교한다.
4. 반환 내용의 `<title>` 태그에 "To-Do lists"라는 단어가 있는지 확인한다. 앞선 기능 테스트에서 확인한 것이기 때문에 단위 테스트로 확인해주어야 한다. 

여기서부터 **TDD 단위 테스트 - 코드 주기** 에 대해 생각해야 한다.

1. 터미널에서 단위 테스트를 실행해서 어떻게 실패하는지 확인한다.
2. 편집기상에서 현재 실패 테스트를 수정하기 위한 최소한의 코드를 변경한다.

코드 품질을 높이고 싶다면 코드 변경을 최고화해야한다. 또한 이렇게 최소화한 코드는 하나하나 테스트에 의해 검증돼야 한다. 매우 고된 작업이라고 생각될 수 있지만, 한번 익숙해지기 시작하면 속도는 빨라진다. 따라서 아무리 자신 있는 부분이라도 작은 단위로 나누어 코드를 변경하도록 한다.

### 기억해두면 좊은 명령어
##### 기능 테스트 실행
python3 functional_test.py
##### 단위 테스트
python3 manage.py test

## 4장 왜 테스트를 하는 것인가?
TDD가 훌륭한 이유 중 하나가 다음에 무엇을 해야 할지 잊어버릴 걱정이 없다는 것이다. 테스트를 다시 실행하기만 하면 다음 작업이 무엇이닞 가르쳐준다.

```python

    def test_can_start_a_list_and_retrieve_it_later(self):
        # 에디스(Edith)는 멋진 작업 목록 온라인 앱이 나왔다는 소식을 듣고
        # 해당 앱 사이트를 확인하러 간다
        self.browser.get('http://localhost:8000')

        # 웹 페이지 타이틀과 헤더가 'To-Do'를 표시하고 있다
        self.assertIn('To-Do', self.browser.title)
        header_text = self.browser.find_element_by_tag_name('h1').text
        self.assertIn('To-Do', header_text)

        # 그녀는 바로 추가하기로 한다
        inputbox = self.browser.find_element_by_id('id_new_item')
        self.assertEqual(
            inputbox.get_attribute('placeholder'),
            'Enter a to-do item'
        )

        # "공작깃털 사기"라고 텍스트 상자에 입력한다
        # (에디스의 취미는 날치 잡이용 그물을 만드는 것이다)
        inputbox.send_keys('Buy peacock feathers')

        # 엔터키를 치면 페이지가 갱신되고 작업 목록에
        # "1: 공작깃털 사기" 아이템이 추가된다
        inputbox.send_keys(Keys.ENTER)

        table = self.browser.find_element_by_id('id_list_table')
        rows = table.find_elements_by_tag_name('tr')
        self.assertTrue(
            any(row.text == '1: Buy peacock feathers' for row in rows)
        )

        # 추가 아이템을 입력할 수 있는 여분의 텍스트 상자가 존재한다
        # 대시 "공작깃털을 이용해서 그물 만들기"라고 입력한다(에디스는 매우 체계적인 사람이다)
        self.fail('Finish the test!')
```

셀레늄이 제공하는 다양한 메소드를 사용하고 있다. `find_element_by_tag_name`, `find_element_by_id`, `find_elements_by_tag)bane`등이다. 셀레늄의 입력 요소를 타이핑하는 방법인 `send_keys`라는 것도 사용한다.
> element와 elements의 차이점에 유의해야한다. 전자는 하나의 요소만 반환하며 요소가 없으면 예외를 발생시킨디. 반면 후자는 리스트를 반환하며 이 리스트가 비어 있어도 괜찮다.

### "상수는 테스트하지 마라"는 규칙과 탈출구로 사용할 템플릿
단위 테스트는 로직이나 흐름 제어, 설정 등을 테스트하기 위한 것이다. 정확히 어떤 글자들이 HTML 문자열이 배열되어 있는지 체크하는 어썰션은 아무 의미가 없다.

#### 템플릿을 사용하기 위한 리팩터링
리팩터링(Refactoring)이란 **"기능(결과물)은 바꾸지 않고"** 코드 자체를 개선하는 작업을 일컫는다.


#### 트래이스백 분석

```
$ python3 manage.py test
[...]
======================================================================
ERROR: test_home_page_returns_correct_html (lists.tests.HomePageTest) #1
 ---------------------------------------------------------------------
Traceback (most recent call last):
  File "/workspace/superlists/lists/tests.py", line 17, in
test_home_page_returns_correct_html
    response = home_page(request) #2
  File "/workspace/superlists/lists/views.py", line 5, in home_page
    return render(request, 'home.html') #3
  File "/usr/local/lib/python3.3/dist-packages/django/shortcuts.py", line 48,
in render
    return HttpResponse(loader.render_to_string(*args, **kwargs),
  File "/usr/local/lib/python3.3/dist-packages/django/template/loader.py", line
170, in render_to_string
    t = get_template(template_name, dirs)
  File "/usr/local/lib/python3.3/dist-packages/django/template/loader.py", line
144, in get_template
    template, origin = find_template(template_name, dirs)
  File "/usr/local/lib/python3.3/dist-packages/django/template/loader.py", line
136, in find_template
    raise TemplateDoesNotExist(name)
django.template.base.TemplateDoesNotExist: home.html #4 

 ---------------------------------------------------------------------
Ran 2 tests in 0.004s
```
4. 먼저 에러를 확인하자: 템플릿을 발견할 수 없어서 에러가 발생하고 있다.
1. 어떤 테스트가 실패하는지 다시 확인한다: HTML 뷰 테스트 부분이다.
2. 테스트의 어느 코드에 문제가 있는지 확인한다: home_page 함수 호출에 문제가 있다.
3. 마지막으로 애플리케이션의 어느 부분에서 에러가 발생하는지 확인한다: render 호출 부분에서 에러가 발생한다.

```python
    def test_home_page_returns_correct_html(self):
        request = HttpRequest()
        response = home_page(request)
        expexted_html = render_to_string('home.html')
        self.assertEqual(response.content.decode(), expexted_html)
```
`.decode()`를 이용해서 `response.content` 바이트 데이터를 파이썬 유니코드 문자열로 반환한다. 이를 통해 바이트와 바이트를 비교하는것이 아니라 문자열과 문자열을 서로 비교할 수 있다.
여기서 중요한것은 상수를 태스트하는 것이 아니라 구현 결과물을 비교하는 것이다.

### 리팩터링에 관해

리팩터링 시에는 앱 코드와 테스트 코드를 한 번에 수정하는 것이 아니라 하나씩 수정해야 한다.

### 정리: TDD 프로세스

- 기능 테스트(Functional tests)
- 단위 테스트(Unit tests)
- 단위 테스트-코드 주기(Unit-test/code cycle)
- 리팩터링(Refectoring)

![](./images/tdd_cycle.png)

## 5장 사용자 입력 저장하기

기능 테스트에서 예측하지 못한 에러가 발생하면, 다음과 같은 사항을 디버깅해야한다.

- print문을 사용해서 현재 페이지 텍스트 등을 확인해본다.
- 에러 메세지를 개선해서 더 자세한 정보를 출력하도록 한다.
- 수동으로 사이트를 열어본다.
- time.sleep을 이용해서 실행 중에 있는 테스트를 잠시 정지시킨다.

Django의 CSRF(Cross-Site Request Forgery)보호는 사이트의 각 폼이 생성하는 POST 요청을 확인할 수 있는 토큰을 자동 생성한다. 이것은 중괄호와 퍼센트 표시를 사용해서 기술한다.  `{% ... %}` 세상에서 가장 입력하기 어려운 두 가지 키 조합이라고도 알려져 있다.

### 서버에서 POST 요청 처리

HttpRequest에 몇 가지 특수한 속성들을 사용하고 있는 것을 알 수 있다. 바로 .method와 .POST이다.(Django의 'request and response'참조)
또한 POST 요청에 의해 반환되는 HTML에서 텍스트를 확인한다. 이 테스트는 예측한 결과를 보여준다.

### 레드/그린/리팩터와 삼각법
단위 테스트-코드 주기를 레드(Red), 그린(Green), 리팩터(Refactor)로 설명하는 경우도 있다.

- 실패할 단위 테스트를 작성함으로써 작업을 시작한다(레드).
- 이 테스트를 통과할 최소 코드를 작성한다(그린). 편법이라도 상관없다.
- 코드를 리팩터링해서 이해할 수 있는 코드로 만든다.

그러면 리팩터 단계에서는 무얼하면 될까? '편법'이라고 한 것을 모두가 만족하는 방법으로 변경하기 위해선 어떻게 해야 할까?

한 가지 방법은 "중복을 제거"하는 것이다. 테스트가 마법의 상수(아이템 앞의 "1:" 같이)를 사용하고, 애플리케이션도 동일한 코드를 사용한다면 기술이 중복되기 때문에 정당화가 가능하다. 하지만 마법의 상수를 애플리케이션 코드에서 제거한다면 편법 사용을 중단해야 한다.
이 방법은 애매한 측면이 있다. 그래서 필자가 선호하는 방법은 "삼각법"(Triangulation)이다. 테스트가 마법의 상수 같은 편법 코드를 허용하지 않는다면, 다른 테스트를 작성해서 더 나은 코드를 작성하는 방법이다. 즉, "2:"로 시작하는 두 번째 아이템이 추가되더라도 이것을 확인할 수 있도록 FT를 확장하는 것이다.

### 스트라이크 세 개면 리팩터
무언가 나쁜 "코드 냄새"를 FT를 통해 맡을 수 있다.
DRY(Don't Repeat Yourself)라는 원리가 있는데, 즉 한번 정도는 복사-붙여넣기를 해줄 수 있지만, 같은 코드가 세번 등장하게 되면 중복을 제거해야한다는 이론이다.

기능 테스트를 리팩터링 한다. 핼퍼(Helper)메소드를 이용한다. `test_`으로 시작하는 메소드만 테스트로 실행된다는 것을 기억하자. 즉 `test_`외의 이름을 사용해서 사용자 정의 메소드를 만들 수 있다는 것이다.

### Django ORM과 첫 모델
객체 관계형 맵핑(Object-Relational Mapper, ORM)은 데이터베이스의 테이블, 레코드, 칼럼 형태로 저장돼 있는 데이터를 추상화한 것이다. 이것을 이용하면 익숙한 객체 지향 코드 방식을 이용해서 데이터베이스를 처리할 수 있다. 데이터베이스는 클래스로 표현하고, 칼럼은 속성, 레코드는 각 클래스의 인스턴스로 표현한다.

Django는 훌륭한 ORM을 탑재하고 있다. 또한 ORM을 이용해서 단위 테스트를 작성하면, 실제 ORM 사용법을 배울 수 있어서 좋다. 단위 테스트는 예측된 동작을 코드로 작성해야 하기 때문에 충분한 연습이 된다.

### 작업 메모장
코딩을 하는 동안 우리가 해야 할 작업을 기록해두는 곳. 이렇게 기록해두면 현재 작업하고 있는 것이나 마저 못한 작업을 나중에라도 끝낼 수 있다.

## 6장 최소 동작 사이트 구축
### 기능 테스트 내에서 테스트 격리
기능 테스트를 실행할 때마다 앞 테스트의 목록 아이템이 데이터베이스에 남아있었다. 이것은 다시 다음 테스트 결과 해석을 방해하게 된다.  
단위 테스트를 실행하면, Django 테스트 실행자가 자동으로 새로운 테스트 데이터베이스(실 데이터베이스와 별도)를 생성한다. 이를 통해 개별 테스트 실행 시마다 안전하게 데이터베이스가 초기화된다. 하지만 현대 기능 테스트는 "실제" 데이터베이스인 db.sqlite3를 이용하고 있다.

이 문제를 해결하기 위한 한 가지 방법은 자체적인 솔루션을 만들어서 functional_tests.py에 기존 결과물을 정리하는 코드를 추가하는 것이다. 이때 적합한 것이 setUp과 tearDown 메소드다.

Django 1.6 부터는 테스트 실행자가 'test'로 시작하는 파일을 자동으로 찾는다. 깔끔한 작업을 위해서 기능 테스트용 별도 폴더를 만들도록 한다. 즉 테스트를 앱으로 만드는 것과 같다. 이 폴도를 Django가 인식하게 하려면, 유효한 파이썬 패키지 디렉터리(디렉터리 내에 `__inint__.py` 파일이 있는것)로 만들어주면 된다.

LiveServerTestCase를 사용하기 위해 리팩터링
>LiveServerTestCase는 자동으로 테스트용 데이터베이스를 생성하고(단위 테스트와 마찬가지로)기능 테스트를 위한 개발 서버를 가동한다.

**tip! git diff -M : "이동한 것을 감지"하는 기능**
> 기억해두면 도움이 되는 명령어  
> - 기능 테스트 : python3 manage.py test functional_tests  
> - 단위 테스트 : pyhton3 manage.py test lists  
> 일반적으로 단위 테스트가 모두 성공한 후에 기능 테스트를 실행한다. 확신이 없다면 둘 다 실행해보면 된다.

### 필요한 경우에는 최소한의 설계를
TDD는 애자일(Agile) 개발 방법과 밀접한 관계가 있다. 가능한 빨리 사용자가 애플리케이션을 접할 수 있도록 하는 것이 특징이다. 긴 설계 대신에, "동작하는 최소한의 애플리케이션"을 빠르게 만들고, 이를 이용해서 얻은 실제 사용자 의견을 설계에 점진적으로 반영해 사는 방식이다.  
하지만 이것이 설계에 대해 생각하지 않아도 된다는 것을 의미하는 것은 아니다. 아무 생각없이 한 단계씩 진행하는 것이 오히려 답을 주는 경우가 있다. 하지만 설계에 대해 조금이라도 생각하는 것이 때로는 해답에 이르는 빠른 길을 보여주기도 한다. "최소한의 기능을 가직 작업 목록 앱"을 어떻게 설계할것인가?

- 각 사용자가 개별 목록을 저장하도록 한다(지금은 최소 하나의 목록을 유지할 수 있도록 한다)
- 하나의 목록은 여러 개의 작업 아이템으로 구성된다. 이 아이템들은 작업 내용을 설명하는 텍스트다
- 다음 방문 시에도 목록을 확인할 수 있도록 목록을 저장해두어야 한다. 현시점에선 각 목록에 해당하는 개별 URL을 사용자에게 제공하도록 한다. 이후에는 사용자를 자동으로 인식해서 해당 목록을 보여줄도록 수정할 필요가 있다.

작업 아이템을 제공하기 위해선 데이터베이스에 목록과 아이템을 저장해야한다. 각 목록은 개별 URL을 가지며, 작업 아이템은 특정 목록에 연계되는 텍스트로 표현한다.

### YAGNI!
설계에 대해 한번 생각하기 시작하면 생각을 멈추는 것이 쉽지 않다. 하지만 애자일 복음인 YAGNI("You ain't gonna need it"(그것을 사용할 일은 없을 것이다), 야그니)에 순종해야 한다. 아이디어가 정말 끝내주더라도 대개는 그걸 사용하지 않고 끝나는 경우가 많다.오히려 사용하지 않는 코드로 가득 차서 애플리케이션이 복잡해지도 한다. YAGNI는 창의적이지만 과도한 열정을 억제해주는 경전이라고 할 수 있다.

### REST
뷰와 컨트롤러는 어떻게 해야 할까?  

REST(Representational State Transfer)는 웹 설계 방법 중 하나이다. 일반적으로 웹 기반 API를 이용해서 설계하도록 유도한다. 사용자 중심의 사이트를 설계할 때는 REST 규칙을 엄격히 적용하는 것이 어렵지만, 그래도 설계에 도움이 되는 좋은 힌트를 준다.

REST는 데이터 구조를 URL 구조에 일치시키는 방식이다. 이 앱에선 작업 목록과 아이템을 URL 구조로 표현할 수 있다.
/lists/<목록 식별자>/
이것은 FT 에 정의한 요구사항에 부합한다. 목록(list)을 보려면 GET 요청을 사용한다.
새로운 목록을 만들려면 다음과 갗은 특별한 URL을 사용해서 POST 요청한다.
/list/new
기존 목록(list)에 새로운 작업 아이템을 추가하려면, POST 요청을 보낼 수 있는 별도 URL 을 사용한다.
/lists/<목록 식별자>/add_item
(여기선 REST 규칙을 완벽하게 따르지 않는다. REST는 POST 대신에 PUT을 사용한다. 단지 REST 방식을 활용하는 것 뿐이다.)

#### TDD를 이용한 새로운 설계 반영하기
![](./images/tdd_process.png)

```
self.assertRegex(edith_list_url, '/lists/.+')
```
assertRegex는 unittest에 부속된 헬퍼 함수로, 지정한 정규표현과 문자열이 일치하는지 확인한다. REST 설계가 제대로 구현됐는지 확인하기 위해 사용한다.

`##` : 메타(meta) 주석임을 나타낸다. FT에서 일반 주석은 사용자 스토리를 설명하지만, 메타 주석은 테스트가 어떻게 동작하는지 설명하기 위해 사용한다. 

#### 새로운 설계를 위한 반복

```
AssertionError: Regex didn't match: '/lists/.+' not found in 'http://localhost:8081/'
```
Regexp 불일치 문제를 가지고 있는 FT가, 두번째 아이템에 개별 URL과 식별자를 적용하는 것이 다음 작업이라고 말해주고 있다.

URL은 POST 후의 리디렉션으로부터 온다.

### Django 테스트 클라이언트를 이용한 뷰, 템플릿, URL 동시 테스트
#### 새로운 테스트 클래스

```python
		response = self.client.get('/list/the-only-list-in-the-world/')

        self.assertContains(response, 'itemey 1')
        self.assertContains(response, 'itemey 2')
```
1. 뷰 함수를 바로 호출하지 않고 Django TestCase의 속성인 self.client(테스트 클라이언트)를 사용하고 있다. 여기에 테스트할 URL을 .get 한다. 이것은 셀레늄에서 사용하는 API와 비슷하다.
2. 성가신 assertIn/response.content.decode()를 사용하지 않고, 대신에 Django가 제공하는 assertContains 메소드를 사용한다. 이 메소드는 응답(response) 내용을 어떻게 처리해야 하는지 않다.

#### 신규 URL
테스트나 urls.py에서 URL 표기 시 마지막에 붙이는 꼬리 슬래시에 주의하자. 버그의 원인이 될 수 있다.
#### 신규 뷰 함수
뷰를 구현해도 
`AssertionError: 404 != 200 : Couldn't retrieve content: Response code was 404 (expected 200)` 에러가 난다면
테스트에 입력한 url에 오타가 있는지 확인해 보자.
#### 그린? 아니면 리팩터?
`grep -E "class|def" lists/tests.py`
grep은 사용자가 원하는 라인만 골라서 출력해주는 리눅스 명령어이다.
시간 날때 한번 살펴봐야겠다[.](https://iamapark89.wordpress.com/2013/12/01/2-%EB%A6%AC%EB%88%85%EC%8A%A4-%EB%AA%85%EB%A0%B9%EC%96%B4-grep-%EC%9B%90%ED%95%98%EB%8A%94-%EB%9D%BC%EC%9D%B8%EB%A7%8C-%EC%B6%9C%EB%A0%A5%ED%95%98%EA%B8%B0-2/)

#### 목록 출력을 위한 별도 템플릿
메인 페이지와 목록 뷰 기능이 다르기 때문에, 각각 별개의 HTML 템플릿을 사용하는 것이 좋다.

### 목록 아이템을 추가하기 위한 URL과 뷰
#### 신규 목록 생성을 위한 테스트 클래스
꼬리 슬래시에 주의해야한다.
꼬리 슬래시를 사용하지 않는 경우는 데이터베이스에 변경을 가하는 "액션" URL인 경우다.

```
self.assertEqual(Item.objects.count(), 1)
AssertionError: 0 != 1
[...]
    self.assertEqual(response.status_code, 302)
AssertionError: 404 != 302
```
첫번째 오류는 데이터베이스에 아이템을 저장할 수 없다는 메시지고, 두번째는 뷰가 302 상태 코드 대신 404 코드를 반환하고 있다는 메시지다. lists/new가 아직 구축되지 않아서 client.post가 404 응답을 받기 때문이다.
>앞에서 두개의 테스트를 나누어 구현하지 않았다면, 0! = 1 에서 테스트가 멈춰서 디버깅이 훨씬 어려웠을 것이다.

#### 신규 목록 생성을 위한 URL가 뷰
... 마지막 예상하지 못한 두번째 오류가......발생하지 않는다........;;;;
**헬퍼 함수, 테스트 클라이언트**를 더 공부해야겠음.

#### 필요없는 코드와 테스트 삭제
#### 새로운 URL에 있는 폼 가리키기

### 모델조정하기

```python
+class ListAndItemModelsTest(TestCase):

     def test_saving_and_retrieving_items(self):
+        list_ = List()
+        list_.save()
+
         first_item = Item()
         first_item.text = 'The first (ever) list item'
+        first_item.list = list_
         first_item.save()

         second_item = Item()
         second_item.text = 'Item the second'
+        second_item.list = list_
         second_item.save()

+        saved_list = List.objects.first()
+        self.assertEqual(saved_list, list_)
+
         saved_items = Item.objects.all()
         self.assertEqual(saved_items.count(), 2)

         first_saved_item = saved_items[0]
         second_saved_item = saved_items[1]
         self.assertEqual(first_saved_item.text, 'The first (ever) list item')
+        self.assertEqual(first_saved_item.list, list_)
         self.assertEqual(second_saved_item.text, 'Item the second')
+        self.assertEqual(second_saved_item.list, list_)
```
새로운 List(목록) 객체를 생성하고, 각 아이템에 .list 속성을 부여해서 List 객체에 할당하고 있다. 목록이 제대로 저장됐는지와 두 개 작업 아이템이 목로겡 제대로 할당됐는지 확인하도록 한다. 목록 개체를 서로 비교하는 것이 가능하다.(saved_list와 list) 이 비교 처리는, 주 키(.id 속성)가 같은지 확인하고 있다.
>파이썬 기본 list 함수와 충돌하는 것을 막기 위해 `list_`라는 변수명을 사용하고 있다.

### 외래 키 관계

```
AssertionError: 'List object' != <List: List object>
```
!= 양쪽을 주의 깊게 보자. Django가  List 객체를 문자열로 해석해서 저장하고 있다. 객체 자체를 저장하려면 ForeignKey를 이용해서 두 클래스 간의 관계를 Django에세 알려줘야 한다.

>마이그레이션을 삭제하는 것은 위험하다. 데이터베이스에 이미 적용된 마이그레이션을 삭제하면, Django가 상태를 파악하지 못하고 이후 마이그레이션을 어떻게 적용해야 할지 혼란스러워하게 된다. 마이그레이션이 더 이상 필요하지 않다는 확신이 있을 때만 삭제하도록 하자. 일반적으론 **버전 관리 시스템에 커밋한 마이그레이션은 절대 삭제하지 말아야 한다.**

#### 나머지 세상을 위한 새로운 모델
```
  File "/Users/makingfunk/DK_Study/TDD/superlists/lists/views.py", line 14, in new_list
    Item.objects.create(text=request.POST['item_text'])
```
이 에러 메세지는 발생은 하나 찾기 힘들다. views 파일에서 일어난것을 힌트로 찾으면 된다. (2개의 에러 각각 중간쯤에 있다)

### 각 목록이 하나의 고유 URL을 가져야 한다
목록을 위한 고유 식별자로 어떤 것을 사용할까? 이 시점에서는 DB가 자동생성하는 id 필드일 것이다.
LiveViewTest를 수정해서 두 테스트가 새로운 URL을 가리키도록 하자.

#### URL에서 파라미터 취득하기
URL을 통해 어떻게 파라미터가 전달될까.

```
 url(r'^lists/(.+)/$', 'lists.views.view_list', name='view_list'),
```
URL용 정규표현을 수정해서 캡처그룹(capture group)(.+)을 추가한다. 캡처그룹은 '/' 뒤에 나오는 모든 문자에 일치된다. 취득한 텍스트는 인수 형태로 뷰에 전달된다.
즉, URL이 '/list/1/'이면 일반 요청 인수 다음에 있는 두번째 인수, 즉 '1'이 `view_list`에 전달된다. '/lists/foo/'인 경우 `view_list(request, "foo")`가 된다.

```python
	url(r'^lists/new$', 'lists.views.new_list', name='new_list'),
    url(r'^lists/(.+)/$', 'lists.views.view_list', name='view_list'),
    
->

    url(r'^lists/new$', views.new_list, name='new_list'),
    url(r'^lists/(.+)/$', views.view_list, name='view_list'),
```

view 연결할때 '를 빼야한다. 교재와 실습환경의 Django 버전차이인듯 하다.

```
ERROR: test_displays_only_items_for_that_list (lists.tests.ListViewTest)
ERROR: test_uses_list_template (lists.tests.ListViewTest)
ERROR: test_redirects_after_POST (lists.tests.NewListTest)
[...]
TypeError: view_list() takes 1 positional argument but 2 were given
```
views.py에 더미(dummy) 파라미터를 적용해서 이 문제를 쉽게 해결할 수 있다.

### 새로운 세상으로 가기 위한 `new_list`수정

```
#교재
AssertionError: '2: Use peacock feathers to make a fly' not found in ['1: Use
peacock feathers to make a fly']

#현실...
AssertionError: '1: Buy peacock feathers' not found in ['1: Use peacock feathers to make a fly']
```
어딘가 오타가 있는거 아닌가 했는데 찾지 못하겠다........

### 기존 목록에 아이템을 추가하기 위한 또 다른 뷰
### 욕심 많은 정규표현을 조심하자!
아직 lists/1/add_item URL  을 지정하지 않았으니 예상한 에러는 404 != 302이다. 왜 301 에러가 발생하였을까?

바로 URL에 포함된 '욕심 많은' 정규표현식 때문이다.

```
url(r'^lists/(.+)/$', views.view_list, name='view_list'),
```
Django는 영구적 리디렉션(301)에 대한 내부적인 이슈가 있어서 슬래시가 누락되는 경우 문제가 발생한다. 이 경우는 `/lists/1/add_item/` 이 `lists/(.+)/` 에 의해 매칭된다. `(.+)`가 `1/add_item`을 캡쳐하기 때문이다. 결국 Django는 우리가 마지막 꼬리 슬래시를 원한다는 '훌륭한' 추측을 한다.
이것은 URL에서 숫자만 추출하도록 수정하면 된다. 이때 쓰는 정규표현이 `\d`이다.

```
url(r'^lists/(\d+)/$', views.view_list, name='view_list'),
```

### 마지막 신규 URL
이제 예측한 대로 404 상태를 취하고 있다. 기존 목록에 신규 아이템을 추가하는 URL을 만들자.

마지막 에러는 책과 사이트가 서로 다르다.
실습에서는 사이트에서 나온 오류가 발생한다.

```
AttributeError: 'module' object has no attribute 'add_item'
```

### 마지막 신규 뷰
오타를...조심하자.....

### 폼에서 URL을 사용하는 방법

### URL includes 를 이용한 마지막 리팩터링
url 설정은 사이트에 나온 그대로 하면된다.
주의점은 import!!!!!!!!!

```python
# superlists/urls.py. 

from django.conf.urls import url, include
from django.contrib import admin
from lists import views as list_views
from lists import urls as list_urls

urlpatterns = [
    url(r'^$', list_views.home_page, name='home'),
    url(r'^lists/', include(list_urls)),
    # url(r'^admin/', admin.site.urls),
]
```

```python
# lists/urls.py

from django.conf.urls import url
from lists import views

urlpatterns = [
    url(r'^new$', views.new_list, name='new_list'),
    url(r'^(\d+)/$', views.view_list, name='view_list'),
    url(r'^(\d+)/add_item$', views.add_item, name='add_item'),
]
```

### 알아두면 유용한 TDD 개념과 일반적인 법칙
#### 테스트 격리(Test Isolation)와 전역 상태(Global State)
각각의 테스트가 다른 테스트에 영향을 끼쳐서는 안된다. 이것은 각 테스트 마지막에는 영구적인 상태를 초기화해야 한다는 것을 의미한다. Django의 테스트 실행자는 각 테스트 결과물을 제거해주는 테스트 데이터베이스를 생성함으로써 이 문제를 해결해 준다.

#### 동작 상태 확인 후 다음 동작 상태 확인(테스팅 고트님 VS 리팩터링 캣)
일반적으론 모든 것을 한번에 수정하는 것이 쉽다. 하지만 주의하지 않으면 결국 리팩터링 캣 처지가 돼서 오히려 많은 코드를 재수정해야 하거나 아무것도 동작하지 않는 상태가 된다. 테스팅 고트님은 우리가 한 단계씩 수정해서 동작하는지 확인 후에 다음 단계로 넘어가도록  격려하고 있다.


