---
description: 어우~ 유용한 것
---

# 👍 IntelliJ Tip

* 서버 재시작 없이 html의 변경 사항을 반영하는 방법
  * 아래 라이브러리 추가 후
  * Menu --&gt; Build --&gt; Recompile

```
implementation 'org.springframework.boot:spring-boot-devtools'
```

* Preferences 열기
  * command + ,
* Project Structure 열기
  * command + ;
* Test file 만들기
  * 테스트 대상의 클래스에 커서를 위치
  * command + shift + T
    * 만들어진 테스트 혹은 원본 소스로 이동할 수 있기도 하다
* Live template
  * Preferences --&gt; Editor --&gt; Live Templates
  * Custom 선택 후 오른쪽 + 버튼 클릭
  * Live template 선택
  * Abbreviation : 템플릿을 자동으로 완성시키기 위한 진입 키워드
  * Template text : 양식에 맞게 원하는 템플릿을 작성
  * 아래는 Test 함수용 템플릿 샘플
  * 오른쪽 옵션에 Reformat according to style 선택
  * 오른쪽 옵션에 Shorten FQ names 선택

    ```text
    @Test
    public void $NAME$() throws Exception {
        // given
        $END$
        // when
    
        // then
    
    }
    ```
* 자동 변수 추출
  * 메서드를 사용한 위치에서
  * command + option + V
* 경로명 줄이기
  * option + enter
  * `add on-demand static import` 선택\(enter\)
* Extract method \(메소드로 빼내기\)
  * 메소드로 빼낼 코드를 선택
  * option + enter
  * Extract method 선택
  * 함수 내 변수를 매개변수로 만들고 싶다면
    * command + option + P
* 코드 요약하기
  * 요약을 원하는 변수에 커서를 두고 command + option + N

```text
// BEFORE
List<Member> members = memberService.findMembers();
model.addAttribute("members", members);

// AFTER
model.addAttribute("members", memberService.findMembers());
```





