### Keyof, Typeof



#### keyof

`keyof`은 객체 앞에 사용되어서, 객체의 키들의 리터럴 유니온을 나타낸다.

```typescript
type A = keyof { x: number; y: number }; // type A = "x" | "y"
type B = keyof { [n: number]: unknown }; // type B = number
```



#### Enums

TypeScript는 Enum을 지원한다. 근데 Enum을 사용하지 않고도 비슷한 작업을 할 수 있다. (Enum을 쓰지 않는게 좋다는 의견도 있다고 한다.)

```typescript
const ODirection = {
  Up: 0,
  Down: 1,
  Left: 2,
  Right: 3,
} as const;
type Direction = typeof ODirection[keyof typeof ODirection];
// type Direction = 0 | 1 | 2 | 3
```

근데 맨 마지막 줄의 코드가 무슨 말인지 도저히 이해가 안 됐다. 하나씩 뜯어보도록 하자..
<hr>


```typescript
let a = typeof ODirection; // a = "object"
type b = typeof ODirection; 
/* type b = {
    readonly Up: 0;
    readonly Down: 1;
    readonly Left: 2;
    readonly Right: 3;
} */
```

typeof를 let a로 변수로서 할당하니 "object"라는 문자열이 되었다. 이건 JS에서도 동일하다.

근데 type b로 타입으로서 할당하니 오브젝트 리터럴이 할당되었다. 오 신기..
<hr>


``` typescript
type c = keyof typeof ODirection; // type c = "Up" | "Down" | "Left" | "Right"
```

keyof은 타입의 프로퍼티들의 유니온을 반환한다. 위에서 선언한 `ODirection`의 key가 될 수 있는 문자열들이다.
<hr>


```typescript
type Direction = typeof ODirection[keyof typeof ODirection]; 
// type Direction = 0 | 1 | 2 | 3

function run(dir: Direction) { 
  //do something 
}
run(ODirection.Down); 
```

그리고 이걸 다시 객체에 key로 집어넣은 것의 타입을 구하면, 밸류들의 유니온이 나오게 된다. 이렇게 하면  enum을 파라미터로 쓰는 효과를 얻을 수 있다~~





