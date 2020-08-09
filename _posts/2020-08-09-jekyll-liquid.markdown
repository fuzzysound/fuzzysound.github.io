---
layout: post
title:  "Jekyll의 Liquid"
date:   2020-08-09
tags: [ruby, jekyll, github]
---
{% raw %}
Jekyll은 그냥 써도 충분히 괜찮은 블로그이다. 하지만 디자인에 욕심이 많은 사람이라거나 개발자로서의 자존심이 센 사람이라면 자신의 블로그를 원하는대로 바꿔보고 싶을 것이다. 그런 생각으로 Jekyll을 이곳저곳 건드려 보면 결국엔 템플릿 레이아웃을 건드려야 한다는 결론에 도달하게 된다. 그리고 그러려면 Liquid에 대해 알아야 한다.

# Liquid란?
깃헙에 올라와있는 아무 Jekyll 테마나 받은 다음, `_layouts/post.html`에 들어가 보자. 일반적인 html 문법과는 다른 것들이 눈에 띌 것이다.
```html
<div class="post-container">
  <a class="post-link" href="{{ page.url | relative_url }}">
    <h2 class="post-title">{{ page.title | escape }}</h2>
  </a>
  <div class="post-meta">
    <div class="post-date"><i class="icon-calendar"></i>{{ page.date | date: date_format }}</div>
    {%- if page.categories.size > 0 -%}
    <ul class="post-categories">
      {%- for tag in page.categories -%}
      <li>{{ tag }}</li>
      {%- endfor -%}
    </ul>
    {%- endif -%}
  </div>
  ...
```
예를 들어 `{% ... %}` 라든가 `{{ ... }}` 같은 것들은 html 문법이 아니다. 얘네들은 뭘까? 바로 Liquid 문법이다. Liquid란 루비 기반의 템플릿 언어이다. django 유저라면 DTL이나 Jinja 같은 걸 떠올리면 된다. Jekyll은 이 Liquid 문법으로 쓰여진 html 파일을 렌더링하여 최종적으로 보여질 html 파일을 생성한다. 그리고 그 파일은 `_site` 디렉토리에서 확인할 수 있다.

Liquid가 얼마나 대단한 일을 해주길래 굳이 쓰는 걸까? Liquid는 다음과 같은 기능을 제공해준다.

1. 변수를 사용할 수 있게 해준다. (오브젝트)
2. 논리와 흐름 제어를 제공한다. (태그)
3. 함수를 사용할 수 있게 해준다. (필터)

즉 Liquid는 정적인 html을 동적으로 만들어준다고 할 수 있다. 만약 Liquid 없이 블로그를 만든다면 극한의 하드코딩 지옥이 될 것이다. 아래에서는 각각의 기능들에 대해 자세히 알아볼 것이다.

# 변수 (오브젝트)

템플릿에 제공되는 변수를 사용할 수 있도록 해준다. 예를 들어 내가 제목으로 다음과 같은 문구를 쓰고 싶다고 하자.
```html
<h1>This is Fuzzysound's blog!</h1>
```

그런데 여기서 내 이름 Fuzzysound는 언제든 바뀔 수 있는 가능성이 있다. 그런데 그 때마다 일일이 Fuzzysound를 다른 이름으로 수정해야 한다면 귀찮을 것이다. 게다가 저 이름이 제목 한 군데만 쓰이는 것이 아니라면, 일일이 쓰임새를 찾아가며 수정해야 하니 배로 귀찮다. 그래서 저 부분은 다음과 같이 오브젝트를 이용해 수정해줄 수 있다.
```html
<h1>This is {{ name }}'s blog!</h1>
```
그리고 이제 `name` 변수의 값을 `"Fuzzysound"`로 제공해주고, 변경이 필요할 땐 이 변수의 값만 바꿔주면 된다.

그렇다면 여기서 자연스럽게 드는 의문은 '어디서 어떻게 변수를 제공하는가'일 것이다. 여기엔 크게 두 가지 방법이 있다.

## 1. yml 파일 이용하기

모든 Jekyll 블로그에는 기본적으로 `_config.yml`파일이 필요하다. 이 파일에는 블로그에 관한 기본적인 설정값들이 들어간다. Jekyll 블로그에 관한 글들을 보면 이 `_config.yml` 파일의 변수 값을 변경해서 블로그를 커스터마이즈하라고 많이들 설명하고 있다. 그 원리가 바로 이 yml 파일의 값들을 Liquid 템플릿에서 사용하기 때문이다.

예를 들어 `_config.yml` 파일에서 다음과 같이 썼다고 해보자.
```yml
title: fuzzysound
```
그러면 이제 모든 템플릿에서 `{{ site.title }}`을 통해 `title` 값을 사용할 수 있다. 여기서 `site`란 Jekyll에서 사용되는 전역 변수의 일종으로, `_config.yml`에서 설정한 변수는 모두 `site`의 멤버로 들어간다.

Liquid 변수는 또한 계층 구조를 표현할 수 있다. 이미 위에서 `site` 변수가 그걸 보여주고 있고, 우리가 직접 설정할 수도 있다. 예를 들어 `_config.yml`에서 다음과 같이 변수를 설정하면
```yml
social_links:
  github:  fuzzysound
  linkedIn: in/fuzzysound
```
이제 `{{ site.social_links.github }}`와 같이 변수에 접근할 수 있다.

계층 구조를 표현하는 또 다른 방법은 디렉토리를 이용하는 것이다. 예를 들어 `_data`라는 디렉토리를 만들고 하위에 `_data/format.yml`이라는 파일을 다음과 같이 만들면
```yml
github: Github
```
이제 `{{ site.data.format.github }}`를 통해 이 변수에 접근할 수 있다.

## 2. Front Matter 이용하기

Jekyll 블로그의 글은 `_posts` 디렉토리에 저장된다. 아무 블로그나 들어가서 `_posts` 디렉토리 하의 아무 markdown 파일의 코드나 보면 가장 위에 다음과 비슷한 코드를 볼 수 있을 것이다.
```html
---
layout: post
title:  "Jekyll의 Liquid"
date:   2020-08-09
tags: [ruby, jekyll, github]
---
```
이것을 Jekyll에서는 Front Matter라고 부른다. 각 파일에 속해 있는 하나의 작은 yml 파일이라고 보면 된다. 첫 줄부터 시작해서 세 개의 하이픈으로 위아래로 감싸야 Front Matter로 인식이 되며, makrdown 파일뿐만 아니라 html이나 js 파일에서도 설정할 수 있다(css에서는 테스트해보진 않았지만 아마 될 것이다).

각 포스트의 markdown 파일에서 Front Matter로 설정한 변수는 `page` 변수의 멤버가 된다. 그리고 `page` 변수는 `site.posts` 변수를 순회하며 접근할 수 있다 (즉 `site.posts` 변수는 배열이며, 이 배열의 원소들이 모든 `page` 변수라고 보면 된다). 또한 `page` 변수는 해당 Front Matter를 설정한 파일 내에서 전역 변수로 사용할 수 있다. 예를 들어 내가 `{{ page.title }}`이라고 쓰면 렌더링된 html에서는 {% endraw %} "{{ page.title }}"라고 쓰여질 것이다.

{% raw %}
이 Front Matter에는 숨겨진 기능이 하나 있다. 어떤 파일이든 Front Matter를 설정하면 그 파일에서는 Liquid 문법을 사용할 수 있다. 즉 Jekyll은 Front Matter가 설정된 파일만 Liquid가 사용되었다고 인식해 렌더링한다. 그래서 만약 js 파일에서 Liquid 변수를 사용하고 싶다면 js 파일에 Front Matter 설정만 해주면 된다. 설정할 변수가 없으면 그냥
```js
---
---
```
이렇게만 해주면 된다.


# 논리와 흐름 제어 (태그)

쉽게 말해 태그란 if나 for문 등을 쓸 수 있게 해준다. 이런 식으로 말이다.
```html
{% if site.weight == "heavy" %}
  I am heavy!
{% endif %}
```
태그 내에서는 변수를 사용하여 조건 등을 설정할 수 있다. 또한 태그 내에서 사용할 수 있는 연산자는 정해져 있으니 잘 알고 사용해야 한다 ([참고](https://shopify.github.io/liquid/basics/operators/){:target="_blank"}).
If문의 용례야 쉽게 생각할 수 있는데, for문은 어떻게 쓸 수 있을까? 앞에서 모든 `post` 변수들은 `site.posts`를 순회하며 접근할 수 있다고 했다. 만약 메인 페이지에서 모든 포스트를 표시하고 싶다면 다음과 같이 할 수 있다.
```html
{% for post in site.posts %}
<a href="{{ post.url }}">
    <h2>{{ post.title }}</h2>
</a>
{% endfor %}
```
매우 단순화한 예시이므로 실제로는 작동하지 않을 수 있다. 어쨌든 대략 이런 식으로 사용할 수 있다는 것이다.

Liquid는 if나 for 외에도 다양한 태그를 제공한다. 공식 문서에서 모두 확인할 수 있다. 또한 Liquid에서 제공하는 것 외에 Jekyll도 자신만의 특별한 태그를 제공한다. 예를 들어 `includes` 태그는 다른 템플릿을 현재 템플릿에 포함시킬 수 있고, `post_url` 태그는 다른 포스트로의 링크를 생성해준다.

# 함수 (필터)

예를 들어 어떤 변수의 문자열을 모두 대문자화해서 나타내고 싶다고 해보자. 필터를 이용하면 별도의 변수가 없어도 이걸 할 수 있다.
```html
{{ site.title | upcase }}
```
여기서 | (vertical bar)는 필터를 사용하기 위한 기호다. 변수명 오른쪽에 |를 표시하고 필터명을 작성하면 된다.

다음과 같이 필터를 체이닝할 수도 있다.
```html
{{ site.title | upcase | prepend: "WELCOME TO " }}
```
`prepend` 필터는 인자로 주어진 문자열을 변수 앞에 붙인다. 이와 같이 일부 필터는 인자를 받는다. 인자는 필터 오른쪽에 콜론을 표시하고 작성하면 된다.

Liquid에서 기본적으로 제공하는 것들 외에 Jekyll에서는 블로그를 만드는 데 유용한 필터들을 따로 제공한다. 대표적으로 `absolute_url` 필터는 상대경로를 절대경로로 변경해준다. `uri_escape` 필터는 문자열에 포함된 스페이스를 `%20`으로 인코딩해준다.

django 유저라면 여기까지 읽었을 때 익숙함을 느꼈을 것이다. 사실 Liquid는 DTL(Django Template Language)과 문법이 매우 유사하다. 99% 유사하다고 하는 게 더 정확할 것이다. 변수를 제공받는 방법이랑 템플릿을 상속하는 방법 등 사소한 것들을 제외하면 모든 것이 동일하다. 

{% endraw %}

# 실제로 활용하기: 포스트 요약 뒤에 more 붙이기

현재 내 블로그에는 포스트 목록이 다음과 같이 나온다.

![posts](/assets/images/posts.png){:class="img-responsive"}

포스트 목록에는 포스트 제목과 날짜, 포스트 요약본, 태그가 표시되고 있다. 그런데 나는 여기서 포스트 요약 밑에 ...(more) 같은 것을 붙이고 싶다. 현재의 모습은 저 요약본이 포스트 내용의 전체라고 오해할 만하기 때문이다.

내 블로그에서 포스트 목록을 표시하는 템플릿 코드는 `_layouts/home.html`에 포함되어 있다. 이 부분은 다음과 같다.

{% raw %}
```html
{%- for post in site.posts -%}
      <li>
        <div class="post-wrapper"
          {% if post.tags %}
            {% for tag in post.tags %}
              data-{{ tag }}
            {% endfor %}
          {% endif %}
        >
          {%- assign date_format = site.plainwhite.date_format | default: "%b %-d, %Y" -%}
          <a class="post-link" href="{{ post.url | relative_url }}">
            <h2 class="post-title">{{ post.title | escape }}</h2>
          </a>
          <div class="post-meta">
            <div class="post-date">
              <i class="icon-calendar"></i>
              {{ post.date | date: date_format }}
            </div>
            {%- if post.categories.size > 0-%}
            <ul class="post-categories">
              {%- for tag in post.categories -%}
              <li>{{ tag }}</li>
              {%- endfor -%}
            </ul>
            {%- endif -%}
          </div>
          <div class="post">
            {%- if site.show_excerpts -%}
              {{ post.excerpt }}
            {%- endif -%}
          </div>
          <div class="tag-container">
            {% for tag in post.tags %}
            <a>
              <span class="tag" data-tag="{{tag}}">
                {{ site.data.format[tag] }}
              </span>
            </a>
            {% endfor %}
          </div>
        </div>
      </li>
    {%- endfor -%}
```
여기서 포스트 요약을 나타내는 부분은 `{{ post.excerpt }}`이다. 이 부분의 바로 아래에 다음 내용을 넣으면 된다.
```html
          <div class="post">
            {%- if site.show_excerpts -%}
              {{ post.excerpt }}
              <a class="post-link" href={{ post.url | relative_url }}>
                ...(more)
              </a>
            {%- endif -%}
          </div>
```
Liquid 문법이 들어간 부분은 `a` 태그의 `href` 속성 부분이다. 포스트의 URL을 나타내는 `post.url` 변수에 `relative_url` 필터를 먹여 상대경로로 바꿔주었다. 이렇게 하면 모든 포스트에 대해 해당하는 링크를 줄 수 있다.

![posts](/assets/images/posts_with_more.png){:class="img-responsive"}

실제로도 잘 들어간 것을 확인할 수 있다.

{% endraw %}