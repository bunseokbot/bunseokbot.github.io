---
title: 세계로 뻗어나가는 웹 서비스를 만들어보자
description: Django Translation을 이용한 다국어 지원 방법
categories:
 - Django
tags: Django, Python, Translation, Language
---

## 서론

21세기는 지구촌 세상이라지만 세상엔 다양한 언어가 있다. (영어, 중국어, 한국어, 독일어, 불어 등)

우리가 이렇게 다양한 언어를 Django에서 어떻게 모두 지원할 수 있는지 알아보도록 하겠다.

이 글을 만약에 읽기 귀찮다면 [Official Reference](https://docs.djangoproject.com/en/2.1/topics/i18n/translation/) 라도 읽어 보도록 하자. (영어다)



## 기존의 다국어 도입 방식

본격적으로 시작하기 전에, 이 글을 작성하면서 사용한 환경은 아래와 같다.

- Python 3.6.0
- Django 2.1.5

일단 Django 프로젝트를 생성하고, 앱을 생성하는 과정과 기초적인 지식은 있다는 가정 하에 설명하겠다.



우리가 이런 HTML 파일을 view를 통해 보여주려고 가정해보자.

```html
<html>
	<head>
		<title>Hello World</title>
	</head>
	<body>
		Hello.<br>
		My name is Namjun Kim.<br>
		Thanks.<br>
	</body>
</html>
```

근데 우리 서비스를 사용하는 사람 중에 영어를 못하는 사람들이 있어서 (아니면 주로 비영어권 사람에게 서비스를 해야 하거나..?)  이 내용을 하나도 못 알아 듣는다면 서비스를 제대로 이용할 수 없게 된다.

그래서 일반적으로(*역사적으로)는 다국어 서비스를 제공할 때 아래와 같이 구성해 왔다.

`/ko/index.html` (한국어)

```html
<html>
	<head>
		<title>안녕 세상</title>
	</head>
	<body>
		안녕하세요.<br>
		제 이름은 김남준입니다.<br>
		감사합니다.<br>
	</body>
</html>
```

`/de/index.html` (독일어)

```html
<html>
	<head>
		<title>Hallo Welt</title>
	</head>
	<body>
		Guten Tag.<br>
		Mein Name ist Namjun Kim.<br>
		danke schön.<br>
	</body>
</html>
```

이렇게 언어마다 다르게 파일을 만들어서 view에서 사용자의 접속 언어에 따라 다른 template HTML을 보여주도록 구성함으로써 다양한 언어권의 사람들에게 서비스를 제공할 수 있었다.



그러나 이러한 방식에는 가장 큰 문제점이 있었는데..

지원하는 언어가 늘어나는 경우에 일일이 파일을 만들어서 해결하면 되지만 코드가 복잡해지고 같은 template을 언어에 따라 달라지기 때문에 관리할 point가 기하급수적으로 증가하게 되는 큰 문제점이 있다.

이러한 문제를 해결하기 위해 Django에서는 Translation 이라는 기능을 자체적으로 제공하는데 아래와 같이 사용할 수 있다.



## Translation 시작하기

복잡할 것이 없이 그냥 간단하게 View, Model에 아래를 import 하면 된다.
(단, 어떠한 것을 사용해야 할지에 대해서는 아래 ugettext vs ugettext_lazy 관한 문단을 보고 판단하자.)

```python
from django.utils.translation import ugettext as _
```

그리고 나서 다국어 지원이 필요한 부분에 호출하여 사용하면 된다. (HTML에서도 사용할 수 있다.)

`app/models.py`

```python
from django.db import models
from django.contrib.auth.models import User
from django.utils.translation import ugettext_lazy as _


class Article(models.Model):
    """
    Article model class on website articles.
    """
    id = models.AutoField(primary_key=True)
    upload_time = models.DateTimeField(auto_now_add=True, verbose_name=_("Upload Time"))
    author = models.ForeignKey(User, on_delete=models.CASCADE, verbose_name=_("Author"))
    title = models.CharField(max_length=255, default='', null=False, verbose_name=_("Title"))
    content = models.TextField(default='', verbose_name=_("Content"))
    is_published = models.BooleanField(default=False, verbose_name=_("Publish"))

    def __str__(self):
        return self.title

```

`app/index.html`

```html
{% raw %}
{% load i18n %}
<html>
	<head>
		<title>{{ _("Hello World") }}</title>
	</head>
	<body>
		{% blocktrans %}
            Hello.<br>
            My name is Namjun Kim.<br>
            Thanks.<br>
		{% endblocktrans %}
	</body>
</html>
{% endraw %}
```

HTML에서 사용할 때는 반드시 i18n을 load해야 문제 없이 사용할 수 있다.

만약, 여러 문장을 동시에 다국어로 지원하고 싶다면 (규약, 약관 등) blocktrans라는 기능을 이용하면 된다.



## ugettext vs ugettext_lazy

Translation 모듈에 있는 ugettext와 ugettext_lazy에는 약간의 차이가 있다.

* ugettext를 사용해야 하는 경우
  * views.py와 같이 매번 새로 실행되는 경우
* ugettext_lazy를 사용해야 하는 경우
  * Django start-up 때 한번만 실행되는 코드의 경우 (apps.py, models.py, forms.py)



## 언어 파일 생성하기

일단 다국어를 적용할 Text에 모두 gettext를 적용하였다면 1차 관문은 끝난 것이다.

언어 파일을 생성하기 전에 settings.py 파일에 일부 적용해줘야 할 것이 있다.

1. middleware에 LocaleMiddleware 추가하기

   ```python
   'django.middleware.locale.LocaleMiddleware'
   ```

2. 언어 파일 저장 경로 정의하기

   ```python 
   LOCALE_PATHS = (
   	os.path.join(BASE_DIR, 'locale'),
   )
   ```

3. 첫 접속자의 기본 언어 정의

   ```python
   LANGUAGE_CODE = 'en-us'
   ```

이렇게 설정 파일에 적용하게 되면 언어 파일 생성을 위한 준비가 완료된다. (단, gettext 라는 GNU Tool를 추가적으로 설치해야 할 수도 있다. 설치 방법은 검색하면 잘 나오니 무서워 하지 말고 설치하도록 하자.)

자, 이제 진짜 언어 파일을 생성해야 할 때다.

```bash
python manage.py makemessages -l ko
```

이 명령어 하나면 locale 파일 아래에 `django.po` 라는 파일이 생긴 것을 볼 수 있다.  (여기서 ko는 한국어를 말함.)

```python
#: .\templates\article\base.html:6
msgid "My Private Website"
msgstr ""

#: .\templates\article\index.html:5
msgid ""
"\n"
"\t\tHello.<br>\n"
"\t\tThis is my private website<br>\n"
"\t"
msgstr ""

#: .\templates\article\index.html:9
msgid "Show My Articles"
msgstr ""

#: .\templates\article\list.html:5
msgid "MY ARTICLES"
msgstr ""

#: .\templates\article\view.html:7
msgid "Article"
msgstr ""
```

이렇게 생긴 것을 볼 수 있으며, 여기에 msgstr에는 한국어를 써 넣으면 언어 파일이 완성된다.
※ 주의할 점은 blocktrans로 생성된 msgid의 msgstr에 첫 줄에 공백이 들어가면 에러가 발생한다.

```python
msgid ""
"\n"
"\t\tHello.<br>\n"
"\t\tThis is my private website<br>\n"
"\t"
msgstr ""
"\n"
"\t\t안녕하세요.<br>\n"
"\t\t여기는 내 개인 사이트입니다<br>\n"
"\t"
```

이렇게 하지 말고 

```python
msgid ""
"\n"
"\t\tHello.<br>\n"
"\t\tThis is my private website<br>\n"
"\t"
msgstr "\n"
"\t\t안녕하세요.<br>\n"
"\t\t여기는 내 개인 사이트입니다<br>\n"
"\t"
```

이렇게 해야 정상적으로 컴파일 할 때 오류가 나지 않는다.



## 언어 파일 컴파일하기

뭔가 라임이 잘 맞는거 같다. 언어 파일을 컴파일해야 Django에서 사용할 수 있다.

컴파일하기 위해서는 아래와 같은 명령어를 내려주면 된다.

```python
python manage.py compilemessages
```

실행이 완료되면 `django.mo` 라는 파일이 생성되고 이걸 기반으로 다국어 지원을 위한 모든 작업은 완료 되었다.



## 언어 변경 View 만들기

때론 한국어를 사용하고 싶은 사람도 있지만, 영어로 언어를 변경하고 싶은 사람이 있을수도 있다.

이럴땐 아래와 같이 View를 새로 만들어 주고, urlpatterns에 등록시켜 사용자가 원할 때 다른 언어로 바꿀 수 있게 해줘야 하는 경우도 발생할 수 있다. (한국인이 사용하다가 영어권 사람이 사용할 수도 있다.)

`views.py`

```python
from django.utils import translation
from django.http import HttpResponse

def set_language(request, language_code):
	# TO-DO: check vaild language code
	translation.activate(language_code)
    request.session[translation.LANGUAGE_SESSION_KEY] = language_code
	return HttpResponse(status=200)
```

`urls.py`

```python
path('/language/<str:language_code>', views.set_language, name='set_language'),
```

이렇게 해주면 다국어 지원 웹 페이지를 만드는데 모든 과정은 완료된다.



## 부록: Right-to-Left language 지원하기

아마 한국인이 이러한 이슈를 가져가는 일이 흔하진 않겠지만, 개인적으로 궁금한 점이 있어서 찾아보게 되었다. (사실 이러한 이슈를 가지고 있는 한국인이 없어서 그런가 한국어로 된 article이 잘 안나온다..)

다들 너무 익숙해서 잘 모르겠지만 아랍어, 히브리어는 오른쪽부터 쓴다. (한국어나 영어는 왼쪽부터)

그래서 Django에서는 이러한 경우를 대비해서 `get_current_language_bidi` 라는 기능을 제공해서 해당 언어가 오른쪽부터 있는지 아님 왼쪽부터 시작하는지 여부를 체크할 수 있다.

```html
{% raw %}
{% get_current_language_bidi as LANGUAGE_BIDI %}

{% if LANGUAGE_BIDI %}
	<div class="article right-to-left">
        صباح الخير
	</div>
{% else %}
	<div class="article">
        Hello.
	</div>
{% endif %}

<style>
    .right-to-left {
        text-align: right;
    }
</style>
{% endraw %}
```

물론 CSS는 따로 잡아줘야 하지만, class를 다르게 함으로써 자동으로 구분지어 지원할 수 있다.



