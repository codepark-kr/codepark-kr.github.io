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

# 0. Reference

**Web links**
1. [How Browsers Work: Behind the scenes of modern web browsers](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/#The_render_tree_relation_to_the_DOM_tree)
2. [브라우저는 어떻게 동작하는가? Naver D2](https://d2.naver.com/helloworld/59361)
3. [BNF in WikiDic](https://ko.wikipedia.org/wiki/%EB%B0%B0%EC%BB%A4%EC%8A%A4-%EB%82%98%EC%9A%B0%EB%A5%B4_%ED%91%9C%EA%B8%B0%EB%B2%95)
4. [문맥 자유 문법 in wikiDic](https://ko.wikipedia.org/wiki/%EB%AC%B8%EB%A7%A5_%EC%9E%90%EC%9C%A0_%EB%AC%B8%EB%B2%95)
5. [문맥 자유 문법과 푸시다운 오토마타 - 컴파일러의 이해](http://nlp.chonbuk.ac.kr/CD/ch05.pdf)
6. [URI vs URL vs URN](https://mygumi.tistory.com/139)
7. [DOM 트리 - javascript.info](https://ko.javascript.info/dom-nodes)
8. [W3C Standards](https://www.w3.org/standards/)
9. [[컴파일러] - 구문 분석 (Syntax Analysis) III](https://untitledtblog.tistory.com/95)
10. [Linux - Lex and Yacc](https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=oidoman&logNo=220930978312)
11. [~Flex와 Bison의 사용 방법~](https://heaeat.github.io/flex-bison/)
12. [GNU in wikiDic](https://ko.wikipedia.org/wiki/GNU)
13. [Lex와 yacc](https://gammabeta.tistory.com/811)
14. [(Web) DTD(Document Type Definition)란?](https://medium.com/@yeon22/web-dtd-document-type-definition-%EB%9E%80-1a1413771189)
15. [[용어 정리] XHTML, XML, SGML 이란?](http://itnovice1.blogspot.com/2018/12/xhtml-xml-sgml.html)
16. [HTML parsing algorithm](https://html.spec.whatwg.org/multipage/parsing.html)


<br/>

# 1. Introduction

이 글은 이스라엘 개발자 탈리 가르시엘이 [html5rocks.com](html5rocks.com)에 게시한 [https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/](https://www.html5rocks.com/en/tutorials/internals/howbrowserswork/)을 원문으로, [https://d2.naver.com/helloworld/59361](https://d2.naver.com/helloworld/59361)을 참고하여 핵심 내용을 요약한 글입니다. 이는 우리가 작성한 웹 페이지가 어떤 과정을 거쳐서 렌더링되고, 또 동작되는지 그 구조와 과정을 상세하게 설명하고 있습니다. 글의 본문 중 중요하지 않은 부분은 없지만, 이 요약본에서는 전반적인 프로세스와 브라우저 별 렌더링 방법, 그리고 다소 생소할 수 있는 용어들에 대한 개념을 우선적으로 설명하겠습니다. 

더욱 자세한 내용을 알고 싶으신 분들은 해당 사이트의 원문과 Naver D2에서 번역한 글을 직접 읽어보시기를 추천합니다.

## 1-1. The browser
최근에는 인터넷 익스플로러, 파이어폭스, 사파리, 크롬, 오페라 이렇게 다섯 개의 브라우저를 많이 사용하지만, [StatCounter](https://gs.statcounter.com/) 브라우저 통계에 따르면 선두를 달리고 있는 크롬을 포함하여 오픈소스 브라우저가 시장의 상당 부분을 차지하고 있습니다. 이 글에서는, 같은 렌더링 엔진을 공유하고 있는(사파리/크롬(WebKit)과 파이어폭스(Gecko)) 브라우저를 묶어서 언급하겠습니다. 렌더링 엔진에 관한 부가적인 설명은 [1. 렌더링 엔진]() 항목에서 이어집니다.

## 1-2. The browsers main functionality, URI
브라우저의 주요 기능은 사용자가 선택한 resource를 서버에 요청하고 브라우저에 표시하는 것입니다. 자원은 보통 HTML 문서지만, PDF나 이미지, 또는 그 외의 형태일 수 있으며, 자원의 주소는 URI(Uniform Resource Identifier, 통합 자원 식별자)에 의해 정해집니다. 여기서 URI란, 하나의 리소스를 가리키는 문자열을 말합니다. 참고로 URI는 크게 URL(Uniform Resource Locator)과 URN(Uniform Resource Name)으로 분류됩니다. **URL은 웹 상에서의 위치로 리소스를 식별하는 것을 의미하며**, URN은 컨텐츠를 이루는 한 리소스에 대해, 위치에 영양 받지 않는 고유한 이름을 말합니다. 하지만 URN은 공식적으로 채택되지 않았으므로, 결론적으로 사용자가 선택한 자원은 URL에 의해 식별된다고 봐도 됩니다.

브라우저는 HTML과 CSS 명세에 따라 HTML 파일을 해석해서 표시하고, 이에 대한 명세는 웹 표준화 기구인 W3C(World Wide Web Consortium)에 의해 정해집니다.([https://www.w3.org/standards/](https://www.w3.org/standards/)) 현재는 대부분의 브라우저가 표준 명세를 따르고 있습니다.


## 1-3. The browser's high level structure
먼저, 브라우저의 공통적인 구성 요소는 다음과 같습니다: 

1. **The user interface :** 유저 인터페이스. 이는 주소 표시줄, 이전/뒤로가기 버튼, 북마크 메뉴, 기타 등등을 포함합니다. 요청한 페이지를 보여주는 창, 즉 content body를 제외한 모든 부분이라고 보면 됩니다. 
2. **The browser engine :** 브라우저 엔진. UI와 렌더링 엔진 사이의 동작을 최선방에서 제어합니다.
3. **The rendering engine :** 렌더링 엔진. 요청 내용을 표출하는 책임을 지고 있습니다. 예를 들어 요청된 내용이 HTML이라면, 렌더링 엔진은 HTML과 CSS를 파싱(parsing)하여 파싱된 내용을 스크린에 표출합니다.
4. **Networking :** 네트워킹. 플랫폼-독립적인 인터페이스 뒤로 각기 다른 플랫폼을 위해 상이한 구현체(implementations)를 이용하여 통신(HTTP request와 같은)을 호출합니다.
5. **UI backend :** 백엔드 유저 인터페이스. 콤보 박스와 창(windows)과 같은 기본적인 위젯들을 그리는 데에 사용됩니다. 이 백엔드는 플랫폼에 명시되지 않은 포괄적인 인터페이스를 표출합니다. 이는 OS의 유저 인터페이스 메소드들을 잠정적으로 사용합니다.
6. **JavaScript interpreter :** 자바스크립트 인터프리터, 즉 해석기. 자바스크립트 코드를 파싱하고 실행하는 데에 사용됩니다.
7. **Data storage :** 데이터 저장소. 이는 영속적인 레이어입니다. 브라우저는 로컬 데이터의 모든 정렬, 예를 들면 쿠키와 같은 것들의 저장이 필요할 것입니다. 또한 브라우저는 로컬 스토리지, 인덱싱된 데이터베이스, 웹 SQL, 그리고 파일 시스템과 같은 것들의 저장 메커니즘 또한 지원합니다.
<br/>
<br/>
![high-level](/uploads/layers%20(1).png) *Figure: 브라우저의 구성 요소*
<br/>
참고로, 크롬과 같은 브라우저들이 복수의 렌더링 엔진 인스턴스, 각 탭 별로 하나씩 분리된 프로세스를 실행한다는 점은 다소 괄목할 만 합니다.

<br/>

# 2. Rendering Engines

렌더링 엔진의 기본 동작 과정은 다음과 같습니다:
1. HTML 문서를 파싱하고, DOM node로 변환
2. render 트리 구축
3. render 트리 배치
4. render 트리 그리기
   
   ![renderingflow](/uploads/renderingflow.png) *Figure: 렌더링 엔진의 기본 동작 과정*
   

렌더링 엔진은 HTML 문서를 파싱하고 "contents tree = 본문 트리" 내부에서 각 태그를 DOM node로 변환합니다. 이후, 외부 CSS 파일과 함께 문서에 포함된 스타일 요소들을 parsing합니다. 스타일 정보와 HTML 표시 규칙은 "render tree = 렌더 트리"라 부르는 또 다른 계층을 생성합니다. 이러한 render tree의 생성이 끝나면, 각 node가 화면의 정확한 위치에 표시되도록 배치하며, ui backend에서 render tree의 각 node를 통해 형상을 만들어 내는 그리기 작업을 수행합니다.

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

# 3. Parsing and DOM tree construction
## 3-1. Parsing - general
parsing a document, 즉 문서 파싱은 코드를 이해하고 사용할 수 있는 구조로 변환하는 것을 의미합니다. parsing 결과는 보통 문서 구조를 나타내는 node tree인데, 이를 일반적으로는 parse tree(파싱 트리)또는 syntax tree(문법 트리)라 명칭합니다.
예를 들어 2+3-1과 같은 표현식은 다음과 같은 트리로 표현합니다.

![mathematicalexpression](/uploads/mathematicalexpression.png) *Figure: 수학 표현식을 파싱한 트리 노드*
parsing은 문서에 작성된 언어 또는 형식의 규칙을 따르는데, parsing 할 수 있는 모든 형식은 정해진 용어와 구문 규칙에 따라야합니다. 이 것을 context free grammar = BNF에 따른 CFG(컨텍스트 자유 문법/문맥 자유 문법)이라 부릅니다.

![chomski](/uploads/chomski.png) *Figure: context free grammar(CFG)가 포함된 촘스키의 언어위계론*

## 3-2. Parser-Lexer Combination
parsing은 어휘 분석(lexical analysis)와 문법 분석(syntax analysis) 두 가지 과정으로 구분할 수 있습니다. 
먼저, **lexical analysis는 input을 토큰으로 분해하는 과정을 말합니다.** 토큰은 유효하게 구성된 단어의 집합체로, 일종의 단어장이라고도 할 수 있는데, 인간의 언어로 말 하자면 해당 언어의 사전에 등장하는 모든 단어들에 해당됩니다. (*Lexical analysis is the process of breaking the input into tokens. Tokens are the language vocabulary: the collection of valid building blocks. In human language it will consist of all the words that appear in the dictionary for the language.*) 반면에 Syntax analysis(문법 분석)의 경우, 해당 언어의 문법 규칙을 따릅니다.

**parser는 일반적으로 두 구성 요소 사이에서 업무를 분담하는데: the lexer(= tokenizer), input을 유효한 토큰으로 분해하고, parser는 언어 문법 규칙에 따라 문서 구조를 분석하는 parse tree를 구성합니다.** the lexer(tokenizer, 어휘)는 공백과 줄바꿈과 같이 무관한 문자들을 벗겨냅니다.

이러한 parsing 과정은 반복됩니다. parser는 통상 lexer에게 새로운 토큰과 문법 규칙 중 하나의 토큰의 대조를 요청합니다. 규칙과 대응된다면, node와 일치하는 토큰은 parse tree에 추가되며 parser는 또 다른 토큰을 요청합니다. 만약 규칙과 일치하지 않는다면, parser는 토큰을 내부적으로 저장하고, 내부에 저장된 모든 token이 그와 일치하는 규칙에 대응될 때까지 요청합니다. 그 어떤 규칙도 발견되지 않는다면, parser는 exception을 일으킵니다. 즉, 문서는 값을 가지지 않고 문법 오류를 포함하게 됩니다.

## 3-4. Translation
많은 경우에, parse tree는 *최종* 결과물이 아닙니다. parsing은 주로 컴파일 등, input 문서를 또 다른 포맷으로 변환하는데에 사용됩니다. 소스 코드를 기계 코드로 컴파일하는 컴파일러는 먼저 이를 parse tree로 파싱하고, 기계 코드 문서로 변환합니다. 

![compilationflow](/uploads/compilationflow.png) 

즉, 컴파일 등 input을 또 다른 형태로 변환해야 하는 경우, 위의 이미지와 같이 소스 코드를 파싱해서 parse tree를 생성하고, 이를 해당 현태로 변환하여 목적한 최종 코드로 만들어 냅니다.

## 3-5. Types of parsers
parser는 두 가지 종류가 있습니다: 상향식(top-down) parser와 하향식(bottom-up) parser입니다. 직관적인 설명으로는, top-down(하향식) parser는 문법의 높은 수준의 규칙을 찾고 그와의 대응을 찾아내려 합니다. bottom-up(상향식) parser는 input에서부터 저수준 규칙이 고수준 규칙을 만날 때까지 문법 규칙 내부로 점진적인 변환을 시작합니다.

이 중, 상향식 파서는 입력 값이 규칙에 맞을 때까지 찾아서, 입력 값을 규칙으로 바꾸는데 이 과정은 입력 값의 끝(최우단-rightmost-)까지 진행됩니다. 또한, 부분적으로 일치하는 표현식은 parser stack에 쌓이게 됩니다. 따라서 상향식 파서는 입력값의 오른 쪽으로 이동하면서(입력 값의 처음을 가리키는 포인터가 오른쪽으로 이동하는 것을 상상해보면 이해가 됩니다.) 구문 규칙으로 갈수록 남는 것이 점차 감소하기 때문에 shift-reduce parser(이동-감소 파서)라고도 부릅니다.

## 3-6. Generating parsers automatically
parser를 생성하기 위한 도구들이 존재합니다. 이들은 언어의 어휘(vocabulary)나 문법 같은 것들을 부여하면 작동하는 parser를 생성 해 줍니다. 파서를 생성하기 위해서는 파싱에 대한 깊은 이해를 요구하며, 직접 최적화된 파서를 생성하는 것은 결코 쉬운 일이 아닙니다. 따라서, parser 생성기들은 매우 유용할 수 있습니다.

WebKit은 두 개의 잘 알려진 parser 생성기들을 사용합니다: lexer, 즉 **어휘를 생성하기 위한 Flex와 parser를 생성하기 위한 Bison입니다.** Flex는 lex의 기능을 개선한 소프트웨어로, Flex의 input은 토큰의 정규표현식 정의를 포함한 파일입니다. Bison은 yacc의 기능을 개선한 GNU parser를 생성하며, BNF 포맷의 언어 문법 규칙을 입력받습니다.

Lex와 Yacc에 대한 부가 설명을 붙여보자면 프로그래밍 언어를 위한 컴파일러 또는 인터프리터(해석기)는 2가지로 분류되곤 하는데, 이 중 소스 프로그램을 읽고 그 구조를 발견하기 위한 parser 생성기가 Lex와 Yacc입니다. 소스 구조를 발견하는 작업은 다시 2가지의 하위 작업으로 나눠지며, 소스 파일을 토큰으로 분해하는 것이 Lex, 프로그램의 계층 구조를 찾는 역할을 하는 것이 바로 Yacc입니다. Flex는 Lex의 오픈 소스 버전이고, Bison은 Yacc의 GNU버전 입니다.


<br/>

# 4. HTML parser
HTML 파서는 HTML 마크업을 parse tree로 변환합니다.

## 4-1. Not a context free grammar
parser-general에서 상기한 것처럼, 프로그래밍 문법은 BNF와 같은 형식을 사용해서 공식적으로 정의할 수 있습니다. 하지만 모든 관습적인 parser는 HTML에 적용할 수 없습니다. HTML은 parser가 필요한 것처럼 문맥 자유 문법에 의해 쉽게 정의될 수 없기 때문입니다. HTML-DTD(Document Type Definition = 문서 형식 정의)를 정의하기 위한 공식적인 포맷이 존재하나, 이는 문맥 자유 문법은 아닙니다. 

재미있는 점은, HTML은 XML과 유사하며, XML은 사용할 수 있는 parser가 많다는 점입니다. 하지만 HTML이 문맥 자유 문법에 기반한 parser로 파싱될 수 없는 이유는, HTML이 더욱 너그럽다는 점입니다. 만약 몇 태그들을 생략하거나, 때때로 시작과 끝 태그를 생략하더라도 이를 지속되도록 내버려둡니다. HTML은 유연한 문법이기 때문에. 

이렇게 작아보이는 세부 사항이 아주 큰 차이를 만들어냅니다. 한 편으로는 이 것이 왜 HTML이 매우 인기있었는지에 대한 주된 이유이기도 합니다. 이는 웹 저자들의 실수를 용납하고 편히 만들었기 때문입니다. 종합하면, HTML은 문맥 자유 문법이 아닌 이상 전통적인 parser들에 의해 parsing 될 수 없고, XML parser로도 parsing이 불가능합니다.

## 4-2. HTML DTD
HTML 정의는 DTD(Document Type Definition) 형식입니다. 이 형식은 SGML(Standard Generalized Markup Language) 계열의 언어들을 정의하는데에 사용됩니다. 참고로 SGML이란 다른 markup 언어를 기술하기 위한 markup 언어입니다. 이 형식은 모든 허용되는 요소들, 그들의 속성들 그리고 계층에 대한 정의를 포함하고 있습니다. DTD에는 몇 가지 변종들이 있습니다. 엄격한 형식은 명세만을 따르지만 또 다른 모드에서는 과거의 브라우저들에 의해 사용된 markup을 위한 지원도 포함하고 있습니다. 

## 4-3. DOM
결과물 트리 (the "parse tree")는 DOM 요소들과 속성 노드들의 tree입니다. DOM은 Document Object Model(문서 객체 모델)의 준말로, HTML 문서와 HTML 인터메이스 요소들을 JavaScript와 같이 외부 세계로 표출하는 객체입니다. tree의 최상위 객체는 "Document" 입니다. 

DOM은 markup과 1:1의 관계를 맺습니다. 예를 들어:

{% highlight html %}
{% raw %}
<html>
    <body>
    <p>
    Hello World
    </p>
    </body>
    <div><img src="example.png"></div>
</html>
{% endraw %}
{% endhighlight %}

위의 마크업은 다음과 같은 DOM tree로 변환됩니다:

![dommarkup](/uploads/dommarkup.png) *Figure: 예시 마크업의 DOM tree** 

트리가 DOM node를 생성한다고 말 한 것은, DOM 인터페이스들 중 하나를 상속하는 요소들이 tree를 구축하고 있다는 의미입니다. 브라우저는 브라우저 내부적으로 다른 속성들을 구체적인 구현체로 사용합니다.

## 4-4. The parsing algorithm
이 전의 섹션에서 상기한 바와 같이, HTMl은 일반작인 상향식 또는 하향식 파서로는 parsing 될 수 없습니다.
그 이유는 다음과 같습니다:

1. 언어의 너그러운 속성
2. 브라우저들이 가진 유효하지 않은 HTML에 대한 관습적인 에러 허용
3. **파싱 과정이 재입가능하기 때문에.** 다른 언어들의 소스는 파싱 과정에서 변화하지 않으나, **HTML의 동적인 코드**(`document.write()`를 포함하고 있는 script element(요소) 등)**는 토큰들을 추가할 수 있고 파싱 프로세스는 실제로 input을 수정하게 된다.**

기존의 parsing 기술들이 사용 불가하기 때문에, 브라우저는 HTML을 파싱하기 위해 custom parsers를 생성합니다.
[파싱 알고리즘은 HTML5 명세 중 세부사항에 기술되어 있습니다.](https://html.spec.whatwg.org/multipage/parsing.html). 
알고리즘은 두 단계로 구성됩니다: Tokenization(토큰화), Tree Construction(트리 구축).

토큰화는 어휘 분석(Lexical Analysis)으로, input을 시작 태그, 종료 태그, 속성명, 그리고 속성값 중 하나의 토큰으로 parsing합니다. 토큰화는 이를 트리 구축과 다음 토큰을 인지하기 위한 다음의 문자로, 입력값이 끝날 때까지 토큰을 인지합니다. 

![htmlparsingflow](/uploads/htmlparsingflow.png) *Figure: HTML 파싱 과정(HTML5 spec에서 발췌)*


## 4-5. The tokenization algorithm
알고리즘의 결과물은 HTML 토큰입니다. 알고리즘은 상태 기계와 같이 표출됩니다. (The algorithm is expressed as a state machine. -주: 상태를 의미하는 state를 주기적으로, 또는 많은 양을 만들어낸다는 의미에서-) 각 상태는 하나 이상의 연속된 문자를 입력받아 이 문자에 따라 다음 state, 즉 상태를 갱신합니다. 그러나 그 결과는 현재의 토큰화 상태와 트리 구축 상태의 영향을 받는데, 이 것은 같은 문자를 읽어 들여도 현재의 상태에 따라 다음 상태의 결과가 다르게 나온다는 것을 의미합니다. 

다음의 HTML은 토큰화에 관한 기본적인 예제입니다:

{% highlight html %}
{% raw %}
<html>
    <body>
    Hello World
    </body>
</html>
{% endraw %}
{% endhighlight %}

초기의 상태는 "Data State", 즉 (가공되지 않은)데이터 상태입니다. &lt; 문자를 만나면, 상태는 "태그 열림 상태 Tag open state"로 변경됩니다. a부터 z까지의 문자를 만나면 "시작 토큰 Start tag token" 상태를 생성하고, "태그 이름 상태 Tag name state" 상태로 변경됩니다. &gt; 문자를 만나기 전까지 이 상태에 머무릅니다. 각 문자들은 새로운 token 이름에 덧붙여집니다. 위의 경우에서는 `<html>` 토큰에 해당됩니다.

&gt; 태그에 이르면, 현재의 토큰(`<html>`)은 방출되고 상태는 다시 "데이터 상태 Data state"로 돌아옵니다. `<body>` 태그가 같은 과정을 밟게 될 것입니다. 지금까지 html과 body 태그들이 발행되었습니다. 우리는 다시 "데이터 상태"로 돌아옵니다. `Hello World`의 `H` 문자를 만나면 문자 토큰의 생성과 방출을 일으킬 것이고, 이는 `</body>` 에 도달할 때까지 계속됩니다. 이제 우리는 `Hello World`의 문자 토큰들을 방출할 것입니다.

우리는 이제 다시 "태그 열림 상태"로 돌아옵니다. `/`문자는 종료 태그 토큰을 생성하고 "태그 이름 상태 end tag state"로 변경될 것입니다. 이 상태는 `>` 문자를 만날 때까지 유지되며, 새로운 태그 토큰이 발행되고 다시 `데이터 상태`가 됩니다. 이는 이전의 경우들과 동일하게 처리됩니다.

![tokenizingthe](/uploads/tokenizingthe.png) *Figure: 예시 input의 토큰화*


## 4-6. Tree construction algorithm

parser가 생성되면 Document 객체가 생성됩니다. tree가 구축되는 동안 문서 최상단에서는 DOM 트리가 수정되고, 요소가 추가됩니다. 토큰화에 의해 발행된 각 노드는 트리 생성자에 의해 처리됩니다. 각 토근에 대한 specification(명세)는 이와 관련된 DOM 요소에 정의되어 있고, 이는 토큰에 의해 생성됩니다. DOM 트리에 요소를 추가하는 것이 아니라면 열린 요소는 stack(임시 버퍼 저장소)에 추가됩니다. 이 stack은 부정확한 중첩 및, 종료되지 않은 태그들을 교정합니다. 이 알고리즘 역시 일종의 state machine(상태 기계)로 기술됩니다. 이 상태는 "Insertion modes(삽입 모드)"로 불립니다.

예시의 input을 위한 트리 구축 프로세스는 이러합니다:

{% highlight html %}
{% raw %}
<html>
    <body>
    Hello World
    </body>
</html>
{% endraw %}
{% endhighlight %}

트리 구축 단계의 입력값은 토큰화 단계에서 만들어지는 일련의 토큰을 말합니다. 받은 html 토큰은 **"초기 모드 Initial mode"**가 됩니다. "html" 토큰을 받게 되면 이는 **"before html -즉 html 이전-"** 모드가 되고, 토큰은 이 모드에서 처리됩니다. 이는 `HTMLhtmlElement`요소를 생성하고 문서 객체의 최상단에 추가되게 됩니다.

상태는 **"before head -즉 head 이전-"** 모드로 변경될 것입니다. 그 다음, "body" 토큰을 받게 됩니다. "head" 토큰이 없더라도 `HTMLHeadElement`는 묵시적으로 생성되어 트리에 추가됩니다.

문자 토큰인 `Hello World` 문자열을 아직 받지 않았습니다. 첫 번째 문자를 받게 되면 **"문자 노드 Text node"**를 생성하고 삽입하게 되며, 다른 문자들도 그 노드에 덧붙여지게 됩니다.

body 종료 토큰을 받으면 **"after body -즉 body 이후-"** 모드가 됩니다. 마지막 파일 토큰을 받으면 parsing은 끝이 납니다.

![treeconstruction](/uploads/treeconstruction.jpg) *Figure: 예제 html의 tree 구축*

## 4-7. Actions when the parsing is finished
이 단계에서 브라우저는 문서와 상호작용할 수 있게 되고, 문서 파싱 이후에 실행되어야 하는 **"deferred mode 지연 모드"** 내의 스크립트를 파싱하기 시작합니다. 문서의 상태는 "complete 완료"가 되고, "load" 이벤트가 발생됩니다. 보다 자세한 내용은 [HTML5의 토큰화 알고리즘과 트리 구축](https://html.spec.whatwg.org/multipage/parsing.html#html-parser)에서 확인이 가능합니다.


# 5. CSS parsing
상기했듯, CSS는 문맥 자유 문법(Context free Grammar)이기 때문에 HTML과는 다르게 기존의 parser로 파싱이 가능합니다.

## 5-1. WebKit CSS Parser
브라우저별로 상이한 렌더링 엔진을 차용하고 있다고 설명했습니다. 크롬/사파리는 WebKit, 파이어폭스는 Gecko라는 렌더링 엔진을 사용하고 있다고 설명한 바 있는데, 이 중 WebKit CSS parser에 대해서 먼저 설명하겠습니다.

WebKit은 CSS 문법 파일로부터 자동으로 파서를 생성하기 위해 Flex(= Lexer = Tokenizer), GNU Bison parser generator를 사용합니다. 이 중 Bison은 상향식 이동 감소 파서를 생성하며, 파이어폭스는 직접 작성한 하향식 파서를 사용합니다. 두 경우 모두, 각 CSS 파일은 스타일 시트 객체로 파싱되고 각 객체는 CSS 규칙을 포함하고 있습니다. CSS 규칙 객체(CSS rules object)는 선택자와 선언 객체, 그리고 CSS 문법과 일치하는 다른 객체를 포함합니다. 
이 CSS 파싱 구조는 다음과 같습니다:

![parserCSS](/uploads/parserCSS.png) *Figure: CSS 파싱*

<br/>

# 6. The order of processing scripts and style sheets
## 6-1. Script
**웹은 parsing과 동시에 수행이 되는 동기화 모델로, parsing 중 script 태그를 만나게 되면 parsing을 멈추고 스크립트를 실행하게 됩니다. html을 작성할 때 script 태그를 body 태그의 내부에, 그러나 최하단에 작성하는 이유가 바로 이것입니다.** 스크립트가 외부에 있는 경우, 우선 네트워크로부터 자원을 가져와야 하는데 이 또한 실시간으로 처리되고, 해당 자원을 받을 때까지 parsing은 중단됩니다. 참고로 script에는 **defer**, 즉 지연 옵션이 존재하는데, 해당 스크립트를 defer 옵션으로 설정하면 문서 파싱 과정에서 `<script>`를 만난다고 하더라도 문서 파싱은 중단되지 않으며 문서 파싱이 완료된 이후에 스크립트가 실행되게 됩니다. 또한 HTML5는 스크립트를 비동기로 처리하는 속성을 추가하였는데, 이 때문에 스크립트는 별도의 맥락의 의해 파싱되고 실행될 수 있습니다.

## 6-2. Stylesheet
한 편, 스타일 시트는 다른 모델을 사용합니다. 이론적으로 스타일 시트는 DOM 트리를 변경하지 않기 때문에, 문서 파싱을 기다리거나 중단할 이유가 없습니다. 그러나 스크립트가 문서를 파싱하는 동안 스타일 정보를 요청하는 경우라면 문제가 됩니다. 스타일이 파싱되지 않은 상태라면 스크립트는 잘못된 결과를 내놓기 때문에 많은 문제를 야기합니다. 이 문제는 생각보다 빈번하게 발생하는 것으로, Gecko(파이어 폭스의 렌더링 엔진)은 아직 로드 중이거나 파싱 중인 스타일 시트가 있는 경우 모든 스크립트의 실행을 중단합니다. 반면에, WebKit(크롬/사파리)은 로드되지 않은 스타일 시트 가운데 문제가 될 만한 속성이 있는 경우에만 스크립트를 중단합니다.

<br/>

# 7. Render tree construction
DOM tree가 구축되면, 브라우저는 또 다른 트리, render tree를 구축합니다. 이 트리는 표시해야 할 순서와 문서의 시각적인 구성 요소로써 올바른 순서로 내용을 그려낼 수 있게 하기 위함입니다. Gecko는 이 render tree를 "frame(형상)"이라 명칭하고,  WebKit은 renderer(렌더러), 또는 render object(렌더 객체)라 칭하고 있습니다.

## 7-1. The render tree relation to the DOM tree
renderer는 DOM element에 부합하지만 1:1로 대응하는 관계는 아닙니다. 예를 들어, "head" 요소와 같은 비시각적 DOM 요소는 render tree에 추가되지 않습니다. 또한, display 속성에 "none" 값이 할당된 요소는 트리에 나타나지 않습니다. 단, visibility 요소에 hidden 값이 할당된 요스는 트리에 나타납니다. 간단하게, 시각적으로 표출되는 요소의 경우 render tree에 추가되는 것입니다. 

이와 반면에, 여러 개의 시각적인 객체와 대응하는 DOM 요소도 있는데, 예를 들면 select(표시 영역, 드롭다운 목록, 버튼 표시를 위한 3개의 renderer), 또한 하나의 라인에 충분히 표시할 수 없는 문자가 여러 라인으로 변경될 때 - 새로운 라인이 별도의 renderer로 추가됩니다. 여러 개의 renderer와 대응하는 또 다른 예는 broken HTML(깨진 HTML)입니다. CSS specification에 의하면, inline 박스는 block 박스만 포함하거나, inline 박스만을 포함해야 하는데, inline box와 block box가 혼재되어 있는 경우 이 inline box를 감싸기 위한 익명의 block renderer를 생성합니다.

![therendertree](/uploads/therendertree.png) *Figure: render tree와 DOM tree의 대응 (3.1). The "Viewport"는 최초의 containing block입니다. 이는 WebKit에서는 "RenderView" 객체가 이 역할을 수행합니다.*

## 7-2. The flow of constructing tree
WebKit에서는 스타일을 결정하고 renderer tree root를 구