---
layout: page
title: /seminar&lecture(WIP)
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
    [1-5. See Also](#1-5-see-also)
        [1-5-1. Relative Extensions in Visual Studio Code](#1-5-1-relative-extensions-in-visual-studio-code)
        [1-5-2. Rouge](#1-5-2-rouge)
        [1-5-3. SCSS](#1-5-3-scss)
    [1-6. Remark](#1-6-remark)
[2. Jenkins, AWS](#2-jenkins-aws)
    [2-1. Reference](#2-1-reference)
    [2-2. Introduction](#2-2-introduction)
    [2-3. How to Use Jenkins](#2-3-how-to-use---jenkins)
    [2-4. Explanation AWS S3 & CDN](#2-4-explanation---aws-s3--cdn)
    [2-5. Additional Theory](#2-5-additional-theory)
    [2-6. Remark](#2-6-remark)
[3. Debug](#3-debug)
    [3-1. Reference](#3-1-reference)
    [3-2. Introduction](#3-2-introduction)
        [Java Platform Debugger Architecture (JPDA)](#java-platform-debugger-architecture-jpda)
        [Debugger Breakpoint](#debugger---breakpoint)
        [Debug perspective의 구성 요소](#debug-perspective의-구성-요소)
    [3-3. Explanation](#3-3-explanation)
    [3-4. Syntax](#3-4-syntax)
    [3-5. Remark](#3-5-remark)
[4. Docker](#4-docker)
    [4-1. Reference](#4-1-reference)
    [4-2. Introduction](#4-2-introduction)
    [4-3. Explanation](#4-3-explanation)
        [4-3-1. Containers vs. Virtual Machines (VMs): What's the Difference?](#4-3-1-containers-vs-virtual-machines-vms-whats-the-difference)
        [4-3-2. Architecture of Docker](#4-3-2-architecture-of-docker)
        [4-3-3. Docker Hub](#4-3-3-docker-hub)
    [4-4. Syntax](#4-4-syntax)
    [4-5. Remark](#4-5-remark)

---
# 1. Setting up a GitHub Pages site with Jekyll

## 1-0. Reference
[Jekyll Library Official Doc](https://jekyllrb.com/docs/)
[Jekyll Themes(Free, Open source)](http://jekyllthemes.org/)
[Jekyll 기반의 GitHub Page 생성(1) 환경설정](https://moon9342.github.io/jekyll-start)
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
4. 작성한 코드를 `bundler exec jekyll serve`로 번들링 및 실행한다.
5. 완성된 jekyll 사이트는 로컬에서 `localhost:4000`에서 접속 및 확인이 가능하다. 실제 도메인으로 접속했을 경우는 별도의 custom 도메인을 연결하지 않았다면 2의 repository 명으로, 별도의 도메인을 연결한 경우에는 해당 도메인을 주소창에 입력하면 반영된 것을 확인 할 수 있다. 단, 로컬에서 실 도메인 반영까지는 약간의 시간이 걸린다.
6. 로컬에서 bundling 후 push하면 작업 내용이 실제 페이지에 반영된다.
   
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

**timestamp**
{% highlight ruby %}
{% raw %}{{ site.time | date_to_rfc822 }}{% endraw %}
// e.g. {{ site.time | date_to_rfc822 }}
{% endhighlight %}
<br/>
## 1-4. Conclusion & Result
[codepark.kr](https://www.codepark.kr)
## 1-5. See Also
### 1-5-1. Relative Extensions in Visual Studio Code

**Markdown All in One**
![markdown-ext](/uploads/markdown-ext.png) TOC(Table Of Content)의 생성, 단축키로 markdown 텍스트 효과 대입, 작성 중인 .md(.markdown) 파일의 실시간 프리뷰를 제공 해 주는 extension으로, vscode를 사용해서 Jekyll 블로그를 운영하는 경우 필수인 Extension이다.
<br/>

**markdownlint**
![markdownlint](/uploads/markdownlint.png) 마크다운 문법 전용 Lint.
<br/>

**Code Spell Checker**
![code-spellchecker](/uploads/code-spellchecker.png) Jekyll 블로그를 운영하면서 영어를 작성하는 경우가 많다면 추천하는 Spell Checker(맞춤법 검사기). 이 Extension이 여타 English Spell Checker Extension과 차별되는 점은 코드에 사용되는 특수한 영어 단어 등을 용인한다는 것에 있다.

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

# 2. Jenkins, AWS
![jenkinsman](/uploads/jenkinsman.jpg)


## 2-1. Reference
[Docker Jenkins Official Documentation](https://www.jenkins.io/doc/book/installing/docker/)
[Jenkins란 무엇이며 왜 사용해야 할까요?](https://jjeongil.tistory.com/810#:~:text=Jenkins%EB%A5%BC%20%EC%82%AC%EC%9A%A9%ED%95%98%EB%A9%B4%20%EC%BD%94%EB%93%9C,%EC%97%90%20%EB%B6%80%EC%9D%91%ED%95%A0%20%EC%88%98%20%EC%9E%88%EC%8A%B5%EB%8B%88%EB%8B%A4.)
[CI/CD 파이프라인이란?](https://www.redhat.com/ko/topics/devops/what-cicd-pipeline)
<br/>

## 2-2. Introduction
1.**Jenkins:** 소프트웨어 개발 시 지속적으로 통합 서비스를 제공하는 툴. 일종의 스케줄러. Continuos Integration, Continuos Deployment(CI/CD:지속적 통합)을 가능하게 하는 파이프라인을 작성된 스케줄에 따라 실행하도록 한다. 
1. **AWS S3(Amazon Web Service Simple Storage Service):** 인터넷 스토리지 서비스. 
2. **AWS CDN:** 전 세계적으로 분산되어 있는 서버 네트워크. 캐시 서버를 활용한 컨텐츠 전송 기술. CDN은 이미지 등의 컨텐츠의 원본 이미지 파일을 해당 국가의 서버에 저장하고, 이를 복사하여 전 세계 서버에 복사(Caching)해 주는 서비스. 따라서 해당 이미지가 저장된 국가의 서버가 아니더라도 접속자와 가장 가까운 서버에서 해당 컨텐츠에 접근할 수 있도록 한다.
3. **WIX Tools:** 파일 리스트를 exe, msi 설치 파일로 생성 할 수 있도록 만들 수 있는 툴
4. **AutoUpdate:** 프로그램 업데이트 오픈 소스 활용

<br/>

## 2-3. How to Use Jenkins
1. jenkins의 설치를 위한 shell을 입력한다. *(ATCH.1)*
2. Docker에 jenkins Init Admin Password를 입력한다.
3. Dashboard -> General에서 해당 프로젝트를 어떻게 사용할 지에 대한 Description을 작성한다.
4. 고급 프로젝트 옵션 -> 소스코드 관리 메뉴에서 작성한 프로젝트를 젠킨스에 있는 폴더 위치에서 클론해서 가져온다.
5. 프로젝트를 빌드한 후 다른 프로젝트를 연쇄하여 빌드하는 경우 관련된 옵션을 설정 해 줄 수 있다.
6. 빌드 시에 자신이 만든 shell을 실행해서 그에 맞는 빌드를 할 수 있도록 옵션을 설정 할 수 있다. 이 중, shell execution을 설정하는 경우 원하는 command를 직접 작성 후 설정해서 빌드 이후 실행이 가능하다. shell script의 경우 본인에게 편한 방식으로 작성하면 되며, window shell 역시 지원된다.

<br/>

## 2-4. Explanation AWS S3 & CDN
1. CloudFront Distributions에서 설정되어 있는 도메인이 CDN을 통해서 S3를 가져올 수 있도록 설정이 되어있으며 이를 확인 할 수 있다.
2. CDN을 사용할 때는 `Domain Name@CloudFront Distributions`의 도메인으로 접근이 가능하다.
3. 초기에 도메인 접속 시에는 setip파일이 존재하지 않기 때문에 일정 시간이 소요되고, 해당 파일이 다운로드 된 후에는 보다 향상된 속도로 접근이 가능하다.

<br/>

*(ATCH.1)*
~~~shell
sudo apt install jenkins

docker pull jenkins/jenkins:lts
docker volume create jenkins_home
jenkins/jenkins_lts

source vi jenkins_install_script.sh

docker exec -it jenkins-master-2 bash
~~~ 

<br/>

## 2-5. Additional Theory

<br/>

## 2-6. Remark

<br/>
<hr/>

# 3. Debug
## 3-1. Reference

## 3-2. Introduction
Debug란, 프로그래밍 과정 중에 발생하는 오류나 비정상적인 연산, 즉 버그를 차고 수정하는 과정을 말한다. 즉, 프로그램을 조사하면서 프로그램이 실행되는 과정을 지켜보는 것을 말 한다.
디버깅 방법에는 다음과 같은 크게 3가지 분류가 존재한다:

1. 코드 줄 출력 정보를 출력하는 줄을 추가하는 식으로 프로그램을 수정하는 방법
2. 로깅: 로그의 형태로 프로그램 실행을 언제나 확인할 수 있는 방법
3. 디버깅 도구(Debugger) 사용

<br/>

### Java Platform Debugger Architecture (JPDA)
1. java Virtual Machine Tool Interface (JVMTI, JVM Tool Interface)
2. Java Debug Line Protocol (JDWP, Java Debug Write Protocol)
3. Java Debug Interface (JDI, Java Debug Interface)

<br/>
![java-debug-architecture](/uploads/debug-architecture.jpg)
<br/>

### Debugger Breakpoint
1. Step Over line 단위 실행
2. Step into 함수 내부로 진입
3. Step out (Step Return) 함수를 끝까지 실행시키고 호출 시킨 곳으로 되돌아감
4. Resume: 다음 중단점(Breakpoint)를 만날 때까지 실행
5. 특정 조건을 추가로 걸어 중단점(Breakpoint)를 설정할 수 있다.

<br/>

### Debug perspective의 구성 요소
1. Frames(= Call-stack in Frame)
2. Variables(Watch, Variables, Expressions)

<br/>

## 3-3. Explanation

## 3-4. Syntax 
## 3-5. Remark

<br/>
<hr/>
<br/>

# 4. Docker

## 4-1. Reference
[Containers vs. Virtual Machines (VMs): What's the Difference?](https://www.netapp.com/blog/containers-vs-vms/)
[도커와 도커 컨테이너의 이해](https://www.itworld.co.kr/insight/110748)
[Docker Overview](https://docs.docker.com/get-started/overview/)

<br/>

## 4-2. Introduction
도커 컨테이너는 일종의 소프트웨어를 소프트웨어 실행에 필요한 모든 것을 포함한 완전한 파일 시스템 안에 감싸, 코드, 런타임, 시스템 도구, 시스템 라이브러리 등 서버에 설치되는 모든 것을 아울러 포장하는 것이다. 이는 실행 중인 환경에 상관없이 어플리케이션이 실행 가능하도록 한다.

<br/>

## 4-3. Explanation
### 4-3-1. Containers vs. Virtual Machines (VMs): What's the Difference?
![docker-vm](/uploads/docker-vm.jpg)
Virtual machines and containers differ in several ways, but the **primary difference is that containers provide a way to virtualize an OS so that multiple workloads can run on a single OS instance. With VMs, the hardware is being virtualized to run multiple OS instances.** 가상 머신과 컨테이너는 몇 차이점이 있지만, 가장 주요한 차이점은 container는 os 위에 가상화하는 방법을 제공하여 여러 workload가 하나의 OS 인스턴스에 실행될 수 있게 한다. VM은, 여러 OS 인스턴스에 실행될 수 있게 하드웨어가 가상화된다. 즉, 가상 머신은 완전한 컴퓨터로 항상 guest OS를 설치해야 한다. 따라서 이미지 내부에 OS가 포함되기 때문에 이미지의 용량이 커지게 되며, OS 가상화에만 주력하므로 배포 관리 기능이 부족해지게 된다. 이에 반해 리눅스 컨테이너인 Docker는 실행 파일을 호스트에서 직접 실행, 즉 운영체제로부터 어플리케이션을 분리하기 때문에 컨테이너 런타임 환경을 지원하는 모든 리눅스 서버로 컨테이너를 옮길 수 있다.

<br/>

### 4-3-2. Architecture of Docker


~~~shell
docker build
docker pull
docker run
~~~

### 4-3-3. Docker Hub

## 4-4. Syntax
## 4-5. Remark
