---
layout: post
title:  "[KR] 브라우저는 어떻게 동작하는가? (last update: 07.13.2021.)"
date:   2021-07-11 20:55:00 +0900
categories:
---

## [KR] 브라우저는 어떻게 동작하는가? (last update: 07.13.2021.)
`13.07.2021`
@yb park 

---

CONTENTS


---

# 0. Introduction

이 글은 이스라엘 개발자 탈리 가르시엘이 [html5rocks.com](html5rocks.com)에 게시한 [https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/)을 원문으로, [https://d2.naver.com/helloworld/59361](https://d2.naver.com/helloworld/59361)을 참고하여 핵심 내용을 요약한 글입니다. 이는 우리가 작성한 웹 페이지가 어떤 과정을 거쳐서 렌더링되고, 또 동작되는지 그 구조와 과정을 상세하게 설명하고 있습니다. 글의 본문 중 중요하지 않은 부분은 없지만, 이 요약본에서는 전반적인 프로세스와 브라우저 별 렌더링 방법, 그리고 다소 생소할 수 있는 용어들에 대한 개념을 우선적으로 설명하겠습니다. 더욱 자세한 내용을 알고 싶으신 분들은 해당 사이트의 원문과 Naver D2에서 번역한 글을 직접 읽어보시기를 추천합니다.

## 0-1. The browser
최근에는 인터넷 익스플로러, 파이어폭스, 사파리, 크롬, 오페라 이렇게 다섯 개의 브라우저를 많이 사용하지만, [StatCounter](https://gs.statcounter.com/) 브라우저 통계에 따르면 선두를 달리고 있는 크롬을 포함하여 오픈소스 브라우저가 시장의 상당 부분을 차지하고 있습니다. 이 글에서는, 같은 렌더링 엔진을 공유하고 있는(사파리/크롬(WebKit)과 파이어폭스(Gecko)) 브라우저를 묶어서 언급하겠습니다. 렌더링 엔진에 관한 부가적인 설명은 [1. 렌더링 엔진]() 항목에서 이어집니다.

## 0-2. The browsers main functionality, URI
브라우저의 주요 기능은 사용자가 선택한 resource를 서버에 요청하고 브라우저에 표시하는 것입니다. 자원은 보통 HTML 문서지만, PDF나 이미지, 또는 그 외의 형태일 수 있습니다. 자원의 주소는 URI(Uniform Resource Identifier, 통합 자원 식별자)에 의해 정해집니다. 여기서 URI란, 하나의 리소스를 가리키는 문자열을 말합니다. 참고로 URI는 크게 URL(Uniform Resource Locator)과 URN(Uniform Resource Name)으로 분류됩니다. URL은 웹 상에서의 위치로 리소스를 식별하는 것을 의미하며, URN은 컨텐츠를 이루는 한 리소스에 대해, 위치에 영양 받지 않는 고유한 이름을 말합니다. 하지만 URN은 공식적으로 채택되지 않았으므로, 결론적으로 사용자가 선택한 자원은 URL에 의해 식별된다고 봐도 됩니다.

브라우저는 HTML과 CSS 명세에 따라 HTML 파일을 해석해서 표시하고, 이에 대한 명세는 웹 표준화 기구인 W3C(World Wide Web Consortium)에 의해 정해집니다([https://www.w3.org/standards/](https://www.w3.org/standards/)). 현재는 대부분의 브라우저가 표준 명세를 따르고 있습니다.


## 0-3. The browser's high level structure
먼저, 브라우저의 공통적인 구성 요소는 다음과 같습니다: 

1. **The user interface :** 유저 인터페이스. 이는 주소 표시줄, 이전/뒤로가기 버튼, 북마크 메뉴, 기타 등등을 포함합니다. 요청한 페이지를 보여주는 창, 즉 content body를 제외한 모든 부분이라고 보면 됩니다. 
2. **The browser engine :** 브라우저 엔진. UI와 렌더링 엔진 사이의 동작을 최선방에서 제어합니다.
3. **The rendering engine :** 렌더링 엔진. 요청 내용을 표출하는 책임을 지고 있습니다. 예를 들어 요청된 내용이 HTML이라면, 렌더링 엔진은 HTML과 CSS를 파싱(parsing)하여 파싱된 내용을 스크린에 표출합니다.
4. **Networking :** 네트워킹. 플랫폼-독립적인 인터페이스 뒤로 각기 다른 플랫폼을 위해 상이한 implementations를 이용하여 통신(HTTP request와 같은)을 호출합니다.
5. **UI backend :** 백엔드 유저 인터페이스. 콤보 박스와 창(windows)과 같은 기본적인 위젯들을 그리는 데에 사용됩니다. 이 백엔드는 플랫폼에 명시되지 않은 포괄적인 인터페이스를 표출합니다. 이는 OS의 유저 인터페이스 메소드들을 잠정적으로 사용합니다.
6. **JavaScript interpreter :** 자바스크립트 인터프리터, 즉 해석기. 자바스크립트 코드를 파싱하고 실행하는 데에 사용됩니다.
7. **Data storage :** 데이터 저장소. 이는 영속적인 레이어입니다. 브라우저는 로컬 데이터의 모든 정렬, 예를 들면 쿠키와 같은 것들을 저장하는 것이 필요할 것입니다. 또한 브라우저는 로컬 스토리지, 인덱싱된 데이터베이스, 웹 SQL, 그리고 파일 시스템과 같은 것들의 저장 메커니즘 또한 지원합니다.
<br/>
<br/>
![high-level](/uploads/layers%20(1).png) *Figure: 브라우저의 구성 요소*
<br/>
참고로, 크롬과 같은 브라우저들은 복수의 렌더링 엔진 인스턴스, 각 탭 별로 하나씩 분리된 프로세스를 실행한다는 점은 다소 괄목할 만 합니다.

<br/>

# 1. Rendering Engines

렌더링 엔진의 기본 동작 과정은 다음과 같습니다:
1. HTML 문서를 파싱하고, DOM node로 변환
2. render 트리 구축
3. render 트리 배치
4. render 트리 그리기
   
   ![renderingflow](/uploads/renderingflow.png) *Figure: 렌더링 엔진의 기본 동작 과정*
   

렌더링 엔진은 HTML 문서를 파싱하고 "contents tree = 본문 계층" 내부에서 각 태그를 DOM node로 변환합니다. 이후, 외부 CSS 파일과 함께 문서에 포함된 스타일 요소들을 parsing합니다. 스타일 정보와 HTML 표시 규칙은 "render tree = 렌더 계층"이라 부르는 또 다른 계층을 생성합니다. 이러한 render tree의 생성이 끝나면, 각 node가 화면의 정확한 위치에 표시되도록 배치하며, ui backend에서 render tree의 각 node를 통해 형상을 만들어 내는 그리기 작업을 수행합니다.

이 일련의 과정은 점진적으로 진행됩니다. 다시 말 해, 보다 나은 사용자 경험을 위해 가능한 빠르게 내용을 표시하기 때문에 HTML 파싱 및 DOM node 변환이 모두 끝나기를 기다린 후 render 트리를 구축하지 않고, 먼저 parsing 된 것부터 배치 및 그리기 과정을 진행합니다. 즉 네트워크로부터 나머지 내용이 전송되기를 기다리는 동시에 먼저 받은 내용 일부를 화면에 표시합니다.

이쯤에서 DOM node란 무엇인가에 대해 알 필요가 있습니다.
**DOM(Document Object Model)이란, HTML 문서를 객체 기반으로 표현한 일종의 인터페이스입니다.** 이는 HTML의 구조와 내용을 객체 모델로 바꿔 다양한 프로그램에서 쉽게 사용하기 위해 객체 기반으로 표현한 것입니다. 이 DOM은 node의 계층형 구조로 이루어져 있어, 각 node는 부모와 자식, 형제 node등을 가질 수 있습니다. (GUI 기반의 OS에서 폴더 생성 구조를 상상하면 이해가 쉽습니다.)

DOM node란 무엇인가에 대해 간략적으로 이해하고 나면, 위의 구조도 다르게 보일 것입니다. 
즉, 우리가 작성한 HTML 문서는 
**1. parsing되어 DOM node, 즉 객체 기반의 문서 노드(node = 트리계층 구성 요소)로 변환되고,**
**2. 이 변환된 요소들로 렌더 트리(render tree)를 구축한 후** 
**3. 올바른 자리에 각 node들이 자리잡도록 배치되며,** 
**4. 이들이 웹 페이지에 다시 그려진 것이 우리가 현재 보고 있는 웹 페이지 표출의 실체입니다.**

이러한 과정이 바로 렌더링 엔진에 의해 수행됩니다.

참고로, 크롬/사파리의 렌더링 엔진인 WebKit과 파이어폭스의 Gecko 엔진은 각 렌더링 엔진의 동작 과정을 다음과 같이 정의하고 있습니다. 이 두 가지 렌더링 엔진의 동작 과정은 다소 다르지만, 전반적인 흐름은 기본적으로 동일합니다.

![webkitmainflow](/uploads/webkitmainflow.png) *Figure: WebKit(Chrome/Safari) 렌더링 엔진 동작 과정*

![mozillamainflow](/uploads/mozilamainflow.jpg) *Figure: Gecko(Firefox) 렌더링 엔진 동작 과정*


위의 이미지를 보면, Gecko는 시각적으로 처리되는 render tree를 "frame tree"라고 부르며, 각 요소를 형상(frame)이라 명칭하는데에 반해, WebKit은 render tree를 render object(렌더 객체)로 구성된 render tree라는 용어를 사용합니다. WebKit은 요소를 배치하는 데에 layout이라는 용어를 사용하지만, Gecko는 layout 대신 reflow라 칭하고 있습니다. 또한, Attachment(첨부, 어테치먼트)는 WebKit이 render tree를 생성하기 위해 DOM node와 시각적인 정보를 연결하는 과정을 말합니다. Gecko는 HTML과 DOM tree 사이에 "content sink"라 부르는 과정을 별도로 두는 것으로 보이지만, 이는 DOM 요소를 생성하는 공정으로써, 렌더링 엔진 동작 과정에서 단지 명칭과 표현의 차이일 뿐 유의미한 차이는 아닙니다.

<br/>

# 2. Parsing and DOM tree construction
## 2-1. Parsing - general
parsing a document, 즉 문서 파싱은 코드를 이해하고 사용할 수 있는 구조로 변환하는 것을 의미합니다. parsing 결과는 보통 문서 구조를 나타내는 node tree인데, 이를 일반적으로는 parse tree(파싱 트리)또는 syntax tree(문법 트리)라 명칭합니다.
예를 들어 2+3-1과 같은 표현식은 다음과 같은 트리로 표현합니다.

![mathematicalexpression](/uploads/mathematicalexpression.png) *Figure: 수학 표현식을 파싱한 트리 노드*
parsing은 문서에 작성된 언어 또는 형식의 규칙을 따르는데, parsing 할 수 있는 모든 형식은 정해진 용어와 구문 규칙에 따라야합니다. 
이 것을 context free grammar = BNF에 따른 CFG(컨텍스트 자유 문법/문맥 자유 문법)이라 부릅니다.

![chomski](/uploads/chomski.png) *Figure: context free grammar(CFG)가 포함된 촘스키의 언어위계론*

## 2-2. Parser-Lexer Combination
parsing은 어휘 분석(lexical analysis)와 문법 분석(syntax analysis) 두 가지 과정으로 구분할 수 있습니다. 
먼저, **lexical analysis는 input을 토큰으로 분해하는 과정을 말합니다.** 토큰은 유효하게 구성된 단어의 집합체로, 일종의 단어장이라고도 할 수 있는데, 인간의 언어로 말 하자면 해당 언어의 사전에 등장하는 모든 단어들에 해당됩니다. (*Lexical analysis is the process of breaking the input into tokens. Tokens are the language vocabulary: the collection of valid building blocks. In human language it will consist of all the words that appear in the dictionary for the language.*) 반면에 Syntax analysis(문법 분석)의 경우, 해당 언어의 문법 규칙을 따릅니다.

**parser는 일반적으로 두 구성 요소 사이에서 업무를 분담하는데: the lexer(= tokenizer), input을 유효한 토큰으로 분해하고, parser는 언어 문법 규칙에 따라 문서 구조를 분석하는 parse tree를 구성합니다.** the lexer(tokenizer, 어휘)는 공백과 줄바꿈과 같이 무관한 문자들을 벗겨냅니다.

이러한 parsing 과정은 반복됩니다. parser는 통상 lexer에게 새로운 토큰과 문법 규칙 중 하나의 토큰의 대조를 요청합니다. 규칙과 대응된다면, node와 일치하는 토큰은 parse tree에 추가되며 parser는 또 다른 토큰을 요청합니다. 만약 규칙과 일치하지 않는다면, parser는 토큰을 내부적으로 저장하고, 내부에 저장된 모든 token이 그와 일치하는 규칙에 대응될 때까지 요청합니다. 그 어떤 규칙도 발견되지 않는다면, parser는 exception을 일으킵니다. 즉, 문서는 값을 가지지 않고 문법 오류를 포함하게 됩니다.