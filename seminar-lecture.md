---
layout: page
title: /seminar&lecture
permalink: /seminar-lecture
---
### Seminar & Lecture (frequent update)

`{{ site.time | date_to_rfc822 }}`
@yb park 

---

CONTENTS
[1. Setting up a GitHub Pages site with Jekyll](#1-setting-up-a-github-pages-site-with-jekyll)
    [1-0. Reference](#1-0-reference)
    [1-1. Introduction](#1-1-introduction)
    [1-2. Setting up Development Environment](#1-2-setting-up-development-environment)
        [1-2-1. Installation : Ruby](#1-2-1-installation--ruby)
        [1-2-2. Create/Fork a Repository as GitHub Pages](#1-2-2-createfork-a-repository-as-github-pages)
        [1-2-3. Install gems & Bundling, Execution](#1-2-3-install-gems--bundling-execution)
    [1-3. Liquid Syntax](#1-3-liquid-syntax)
        [1-3-1. syntax highlighter + code block](#1-3-1-syntax-highlighter--code-block)
    [1-4. Conclusion & Result](#1-4-conclusion--result)
    [1-5. Addition](#1-5-addition)
        [1-5-1. Relative Extensions in Visual Studio Code](#1-relative-extensions-in-visual-studio-code)
        [1-5-2. Rouge](#2-rouge)
        [1-5-3. SCSS](#3-scss)
    [1-6. Remark](#1-6-remark)

---
# 1. Setting up a GitHub Pages site with Jekyll

## 1-0. Reference
[Jekyll Library Official Doc](https://jekyllrb.com/docs/)
[Jekyll Themes(Free, Open source)](http://jekyllthemes.org/)
[Jekyll 기반의 GitHub Page 생성(1) - 환경설정](https://moon9342.github.io/jekyll-start)
[What is a Gem?](https://guides.rubygems.org/what-is-a-gem/)
[Liquid Syntax :: Official Docs](https://jekyllrb.com/docs/liquid/)

<br/>

## 1-1. Introduction
Ruby 기반의 Jekyll과 Github Pages를 연동하여 쉽고 빠르게 사용자 정의 웹 사이트를 생성하는 방법에 대해 소개한다. 
<br/>
## 1-2. Setting up Development Environment
### 1-2-1. Installation : Ruby
 Jekyll은 Ruby 기반이므로 [공식 홈페이지](https://rubyinstaller.org/downloads/)에서 **Ruby+Devkit 2.7.X (x64) installer를 다운로드** 한다. 공식적으로 현 시점(2021.07.) 최신 버전인 3.0.X보다 2.7.X 버전을 권장하고 있다. 이는 2.7.X 버전이 1. 가장 많은 수의 *gems¹*를 제공하고 있으며, MSYS2 Devkit을 함께 설치하므로 C 언어 기반의 확장 프로그램들 또한 즉각 컴파일이 가능하기 때문이다. (We recommend that you use the Ruby+Devkit 2.7.X (x64) installer. It provides the biggest number of compatible gems and installs the MSYS2 Devkit alongside Ruby, so gems with C-extensions can be compiled immediately. -official page) 또한, 설치 과정에서 *(ATCH.1)*와 같은 `Add Ruby executables to your PATH` 옵션은 반드시 체크하고 설치를 진행하도록 한다. 이후, bundler 설치 등의 명령어를 window shell로 실행하기 위해서 필요하다.
<br/>

### 1-2-2. Create/Fork a Repository as GitHub Pages
GitHub Pages를 통한 Jekyll 페이지의 생성은 직접 Ruby 기반으로 작성해도 되지만, 우선 마음에 드는 오픈소스 테마를 골라 페이지를 생성한 후 개인이 원하는 디자인 또는 레이아웃을 추가하여 사용하는 것이 일반적이다. 기존의 테마를 사용하는 경우 페이지 생성 절차는 다음과 같다:

1. [Jekyll Themes](http://jekyllthemes.org/)에서 마음에 드는 테마를 fork하거나, 아카이브 파일을 받아서 새로운 Repository를 생성한다.
2. Settings에서 Repository의 이름을 `Github 아이디.io` 등으로 변경한다. *(ATCH.2)*
3. Github Pages와 연동한다. *(ATCH.3)*
<br/>
<br/>

### 1-2-3. Install gems & Bundling, Execution
1. window powershell을 열고, `gem install bundler`를 입력해 bundler gem을 설치한다. 
2. 새로 생성한 저장소의 `_config.yml` 파일의 `plugin` 또는 `gems`에 선언된 플러그인을 `gem install [PLUGIN NAME]`의 형식으로 설치한다.
3. 만약, custom domain을 가지고 있다면 연결한다. 도메인을 구입한 사이트의 DNS 관리 창에서 레코드 타입을 `CNAME`으로 설정하고, 설정한 저장소명을 그대로 입력 해 주면 된다. *(ATCH.4)*
4. 작성한 코드를 `bundle` 명령어로 bundling하고, `jekyll serve`로 jekyll을 실행한다. 이는 `bundler exec jekyll serve`로 한 번에 실행도 가능하다. 
5. 완성된 jekyll 사이트는 로컬에서 `localhost:4000`에서 접속 및 확인이 가능하다. 실제 도메인으로 접속했을 경우는 별도의 custom 도메인을 연결하지 않았다면 2의 repository 명으로, 별도의 도메인을 연결한 경우에는 해당 도메인을 주소창에 입력하면 반영된 것을 확인 할 수 있다. 단, 로컬에서 실 도메인 반영까지는 약간의 시간이 걸린다.
6. 로컬에서 bundling 후 push하면 작업 내용이 실 도메인에 반영된다.
   
<br/>
_(ATCH.1)_
![rubyset](/uploads/rubyset.png)
_(ATCH.2)_
![changename](/uploads/changename.png)
*(ATCH.3)*
![generatepage](/uploads/generatepage.png)
*(ATCH.4)*
![recordmanage](/uploads/recordmanage.png)

<br/>
 
## 1-3. Liquid Syntax
### 1-3-1. syntax highlighter + code block

**Highlight syntax to Code Block**
{% highlight ruby %}
{% raw %}{% highlight java %}{% endraw %}
[CODE ON HERE]
{% raw %}{% endhighlight %}{% endraw %}
{% endhighlight %}

**Linking**
{% highlight ruby %}
{% raw %}{% link [ADDRESS TO LINK] %}{% endraw %}
// e.g. {% raw %}{% link /assets/files/doc.pdf %}{% endraw %}
{% endhighlight %}

**gem timestamp**
~~~ruby
{{ site.time | date_to_rfc822 }}
~~~
<br/>
## 1-4. Conclusion & Result
[codepark.kr](https://www.codepark.kr)
## 1-5. Addition
### 1-5-1. Relative Extensions in Visual Studio Code

**Markdown All in One**
![markdown-ext](/uploads/markdown-ext.png) TOC(Table Of Content)의 생성, 단축키로 markdown 텍스트 효과 대입, 작성 중인 .md(.markdown) 파일의 실시간 프리뷰를 제공 해 주는 extension으로, vscode를 사용해서 Jekyll 블로그를 운영하는 경우 필수인 Extension이다.
<br/>

**markdownlint**
![markdownlint](/uploads/markdownlint.png) 마크다운 문법 전용 Lint.
<br/>

**Code Spell Checker**
![code-spellchecker](/uploads/code-spellchecker.png) Jekyll 블로그를 운영하면서 영어로 작성하는 경우가 많다면 추천하는 Spell Checker(맞춤법 검사기). 이 Extension이 여타 English Spell Checker Extension와 차별되는 점은 코드에 사용되는 특수한 영어 단어 등을 용인한다는 것에 있다.

<br/>
### 1-5-2. Rouge
**Rouge는 pure-ruby syntax highlighter로, 간단하게 Syntax Highlighter를 제공하는 gem이다.**
이를 사용하기 위해서는 다음과 같은 절차를 거친다:

1. window powershell에 `gem install rouge` 커맨드를 입력하여 rouge gem을 설치한다.
2. `rougify help style`을 입력하여 사용 가능한 스타일의 목록을 체크한다. 
3. [Rouge Syntax Highlighter 스타일 목록](http://rouge.jneen.net/)에서 원하는 스타일을 고른다.
4. `rougify style [STYLE NAME]`을 command line으로 입력하여 css 코드를 복사한 후, 붙여넣고 사용한다.

<br/>
### 1-5-3. SCSS
모든 Jekyll 테마는 아니지만, 상당수의 테마는 순수 CSS가 아닌 SCSS 또는 SASS로 작성되어 있다. Jekyll 사이트를 운영하기 위해서 필요한 문법 수준은 크게 어렵지 않아, 다양한 수치 또는 색상 코드 등을 변수로 선언해 두고 사용한다는 점만 알면 된다. 
이 문법은 다음과 같이 작성하고 사용한다:

~~~css
/**
 * Style variables declaration
 */
$base-font-family:     'Source Serif Pro', 'Noto Sans KR' !default;
$syntax-highlight-font-family: 'Source Code Pro', 'Noto Sans KR';
$base-font-size:       17px !default;
$mobile-font-size:     14px !default;
$base-line-height:     1.6 !default;
$container-width:      80% !default;
$container-max-width:  910px !default;

/** Use */
h1, h2, h3, h4, h5, h6 {  margin: 0px;  font-weight: bold;  color: var(--text-color); color: rgb(190, 180, 158); }
p, ul, ol { margin: 0px;  color: var(--text-color); }
a { text-decoration: underline;  color: var(--link-color); cursor: pointer;}
a:hover { color: var(--background-color); background-color: var(--base-color); }
~~~

<br/>
## 1-6. Remark
* *gems¹* : Ruby 내 라이브러리, 즉 플러그인의 통칭. yml 설정 파일에서의 항목명은 plugin으로 변경되었다. 
<br/>
<hr>