# Compare_Animation

Comparison of performance through Composite Layer using css Animation

---

# Browser 렌더링

브라우저의 렌더링 과정은

HTML Parsing & CSS Parsing => Dom Tree & CSSOM Tree 생성 <br/>
Attachment => Render Tree 생성 <br/>
Reflow => Layout 에 따른 요소의 크기와 위치 계산 <br/>
Repaint => Reflow에서 계산해주는 값을 기반하여 Paint <br/>
Composite => Layer 계층을 병합하여 화면에 나타냄 <br/>
<br/>
으로 이루어진다. <br/>

---

# reflow의 특징

- reflow는 브라우저 창 크기를 조절하여도 발생하게 된다.
- 또한, 하나의 요소가 reflow가 발생하면 하위 Layer내부에 존재하는 요소들도 reflow가 발생한다.
- 스레드 큐에 자바스크립트 실행이 끝나야 reflow를 진행한다.

---
