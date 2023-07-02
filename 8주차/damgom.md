## 콜백에서 this에 대한 타입 제공하기

> 이 장은 주로 자바스크립트의 this에 대해 다룹니다.

- this가 가리키는 값은 함수 호출 방식에 따라 동적으로 결정된다


## this를 사용하는 콜백함수가 있으면 매개변수에 this를 추가한다. 
1. this 바인딩이 체크되어 실수를 방지할 수 있다.
2.  사용자의 콜백함수에서 this를 안전하게 사용할 수 있다.

``` ts
function addKeyListener (
  el: HTMLElement,
  fn: (this: HTMLElement, e:KeyboardEvent) => void
) {
  el.addEventListener('keydown', e => {
    fn.call(el, e);
  })
}
```

콜백 함수의 첫번째 매개변수에 있는 this는 특별하게 처리된다.
- call로 this 바인딩 없이 매개변수를 2개 넘기면 1개의 인수만을 원한다는 오류가 발생한다.



![image](https://github.com/FrontendStudySeoul/TypeScript/assets/80238096/a2d1b702-7fe9-4706-835e-e5ed93dac4fd)

- this 바인딩을 체크해주기 때문에 실수를 방지할 수 있다.

![image](https://github.com/FrontendStudySeoul/TypeScript/assets/80238096/3565d3e1-1c76-488b-830a-07d66d49146a)


화살표 함수

![image](https://github.com/FrontendStudySeoul/TypeScript/assets/80238096/ad58794a-e78a-4c32-87e8-f3dbde746987)



![image](https://github.com/FrontendStudySeoul/TypeScript/assets/80238096/599bad3a-ae97-4273-8d79-9aaa0776569b)


→ 콜백 함수에서 this를 사용해야 한다면, 타입 정보를 명시하자!

