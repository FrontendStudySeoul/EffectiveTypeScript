## item 39: any를 구체적으로 변형해서 사용하기

any는 자바스크립트에서 표현할 수 있는 모든 값을 아우르는 매우 큰 범위의 타입입니다. 
![image](https://github.com/FrontendStudySeoul/TypeScript/assets/59603529/9134cfc7-10ad-4349-9e03-773382798a48)
any는 모든 타입이 포함되지만 반대로 말하면 any보다 구체적으로 표현할 수 있는 타입이 존재할 가능성이 높습니다. 어쩔 수 없이 any를 사용하더라도 보다 더 구체적으로 any를 사용할 수 있도록 해야합니다. 예를 들어

```tsx
function getLengthBad(arr: any): {
	return arr.length;
}

function getLength(arr: any[]): {
	return arr.length;
}
```

배열 메소드에 접근하기 위해서는 any를 사용하기 보단 any[]를 사용해 배열임을 알려주는게 더 좋은 방식입니다. 이러면 arr.length 타입이 체크가 되고 getLength의 리턴 타입이 number로 추론됩니다. getLength의 매개변수가 배열인지 체크됩니다.

객체의 경우엔 `{ [key: string]: any }` 처럼 인덱스 시그니처를 이용할 수 있습니다. 

```tsx
function hasTwelveLetterKey(o: {[key: string]: any}) {
	for (const key in o) {
	  if (key.length === 12) {
			return true
		}
	}
}
```

함수의 경우에도 any를 사용하기 보단 `() => any`를 사용해 구체적으로 타입을 명시해줄 수 있습니다.
