# transform 비교 코드

- [code sandbox](https://codesandbox.io/s/vigilant-grass-rtnqce?file=/index.html)

# position 비교 코드

- [code sandbox](https://codesandbox.io/s/recursing-northcutt-deidds?file=/index.html)

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
- 또한, 하나의 요소가 reflow가 발생하면 상.하위 Layer내부에 존재하는 요소들도 reflow가 발생한다. <br/>
  (즉, 대부분의 reflow는 페이지 전체의 렌더링을 발생한다고 한다. )
- 스레드 큐에 자바스크립트 실행이 끝나야 reflow를 진행한다.

---

아래 두 영상을 보자,

https://user-images.githubusercontent.com/70205497/184067980-26d54aba-728b-45e9-9a28-fb972f07402c.mov

https://user-images.githubusercontent.com/70205497/184067983-817af8d9-a9b3-4123-bea3-3673eccb8a0b.mov

겉보기에는 별반 차이가 없다. <br/>
하지만, 브라우저 렌더링 과정을 확인해보면 차이가 분명하다! <br/>
<br/>
개발자 도구의 성능을 사용하여 비교해보았다. <br/>

---

# Performance 비교

우선 Reflow가 발생하는 작업부터 알아보면,

<img width="1440" alt="스크린샷 2022-08-11 오후 2 10 42" src="https://user-images.githubusercontent.com/70205497/184068418-e687098d-710d-4c26-9cda-7d913b31761e.png">

작업이 빽뺵하게 진행 중인 것을 알 수 있고, <br/>
이 작업들을 자세히 들여다 보면,

<img width="1440" alt="스크린샷 2022-08-11 오후 2 10 57" src="https://user-images.githubusercontent.com/70205497/184068469-5e8d2586-9ed4-4452-8294-9f34ac013b82.png">

위와 같이 스타일 계산과 레이아웃, 페인트 과정이 진행되는 것을 알 수 있다. <br/>
<br/>
이렇게 되면 문제가 발생하게 되는데, <br/>
레이아웃은 위에 설명한 것처럼, 스레드 큐에 존재하는 자바스크립트 실행이 끝나야 실행이 된다. <br/>
혹여나 reflow가 진행이 되어야 하는데, 자바스크립트 로직이 진행중이라면 사용자들은 애니메이션이 느리다고 판단할 것이다. <br/>

<br />

아래 영상을 보아 확인해보자 <br/>

https://user-images.githubusercontent.com/70205497/184068769-d3cff74d-a381-4271-8065-d2bf73f3ca3f.mov

alert가 발동하고 나니, alert창으로 인해 스크립트가 동작하지 못하여 reflow 또한 진행이 되질 않는다. <br/>
따라서, 애니메이션은 동작을 중단하게 된다. <br/>

---

그럼, Reflow가 진행되지 않는 animation을 보면,

<img width="1439" alt="스크린샷 2022-08-11 오후 2 19 52" src="https://user-images.githubusercontent.com/70205497/184069053-40b6a088-caed-49f0-bdb7-1e00d6767dc7.png">

스타일 계산, 레이아웃 , 페인트 아무런 동작을 진행하지 않는다.

<img width="1440" alt="스크린샷 2022-08-11 오후 2 12 01" src="https://user-images.githubusercontent.com/70205497/184069131-77ad24f8-ee50-40f4-acb8-cafa5c13b5f4.png">
<img width="1440" alt="스크린샷 2022-08-11 오후 2 20 28" src="https://user-images.githubusercontent.com/70205497/184069136-50ff16b3-ae38-4a2d-b6d5-9ac97d84fe6a.png">

위 두 이미지를 통해, 렌더링 시간과 페인팅 시간 소요에 대한 차이를 명확하게 확인할 수 있다. <br/>
<br/>
그렇다면, 과연 alert창이 나타날때 애니메이션은 동작할까? <br/>

https://user-images.githubusercontent.com/70205497/184069257-798c03b0-30ad-4c04-8824-b25dfc6e278c.mov

당연히, reflow가 발생하지 않으므로 애니메이션은 계속 동작하게 된다.

---

# 추가

## Position : fixed

```jsx
<div id="container">
  <div id="circle"></div>
</div>
```

일반 상황의 경우에서는 circle이 reflow발동이 되면 container 또한 동시에 같이 reflow가 발생한다. <br/>
<img width="1440" alt="스크린샷 2022-08-11 오후 2 50 30" src="https://user-images.githubusercontent.com/70205497/184072409-5569adef-9f30-42bd-b5ed-45a9d64bbe8f.png">

하지만 circle에 fixed를 넣어준다면, 아래와 같이 Layer가 분리됨을 확인할 수 있으며 <br/>

<img width="1439" alt="스크린샷 2022-08-11 오후 2 54 51" src="https://user-images.githubusercontent.com/70205497/184072459-4f0de889-7832-4ebd-90aa-fffbf3b0c989.png">

따라서, circle의 reflow로 인한 container요소의 reflow를 막을 수 있다. <br/>

아래 이미지로 성능을 확인할 수 있다. <br/>

<img width="1439" alt="스크린샷 2022-08-11 오후 2 52 23" src="https://user-images.githubusercontent.com/70205497/184072520-96ad0620-2c83-46c2-89ea-06c7c258a187.png">

<img width="1440" alt="fixed" src="https://user-images.githubusercontent.com/70205497/184072526-40f63457-c8de-4257-8d46-3b56073c0d95.png">

---

# 결론

동일해 보이는 Animation일지라도, 내부를 비교해보면 reflow의 여부에 따른 엄청난 성능차이가 발생하고, 그 차이로 인해 향후 동작에 영향으 끼치게 된다. <br/>
결국, reflow를 발생하지 않도록 해야 한다. <br/>
<br/>
reflow를 발생하지 않게 하기 위해서는, width와 height같은 layout 계산이 필요한 값을 변경하지 않게 해야한다. <br/>
만약, reflow가 발생하게된다면, 자식 요소를 동일한 layer에 두지 않는 것이 중요하므로, Position의 fixed를 사용하여 layer 계층을 분리하여 부모 또는 자식 요소에 대하여 reflow를 줄여야 한다. <br/>
<br />
추가로, 다른 글을 보면 reflow가 발동하면 자식 요소와 부모 요소가 reflow가 발동한다고 한다. <br />
하지만, 나는 동일한 Layer에 존재하는 요소가 reflow 진행된다고 말하는게 더 명확한 표현이라고 생각한다.
