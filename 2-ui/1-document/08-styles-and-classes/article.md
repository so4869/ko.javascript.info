# 스타일과 클래스

자바스크립트로 스타일과 클래스를 다루는 방법을 배우기 전에 알아야 할 중요한 것이 있습니다. 충분히 알고있을것이라 생각하지만, 한번 짚고 넘어갑시다.

HTML요소에 스타일을 적용시키는데에는 두가지 방법이 있습니다.

1. CSS에서 클래스에 스타일을 준 뒤 HTML요소에 클래스를 적용합니다. `<div class="...">`
2. HTML요소에 직접 스타일속성을 입력합니다. `<div style="...">`.

자바스크립트는 클래스, 스타일 다 건드릴 수 있습니다.


우리는 style 방식보다는 CSS class 방식을 이용하도록 해야합니다. style 방식은 CSS class 방식으로 해결할 수 없는 경우에만 사용해야합니다.

예를들어 자바스크립트에서 좌표를 계산하고 HTML요소를 동적으로 움직이게 하고 싶을때 사용합니다.
이렇게요:

```js
let top = /* complex calculations */;
let left = /* complex calculations */;

elem.style.left = left; // e.g '123px', calculated at run-time
elem.style.top = top; // e.g '456px'
```

글씨를 빨간색으로 만들거나, 백그라운드 아이콘을 추가하는 경우에는 CSS에서 선언하고 class속성을 추가하는 편이 좀더 유지보수하기 좋습니다.

## className과 classList

클래스를 변경하는것은 자바스크립트에서 가장 많이 쓰이는 것중 하나입니다.

아주 먼 옛날 호랑이 담배피던 시절에는, 오브젝트의 속성이름에 `"class"`라는 단어를 사용할 수 없었습니다. 지금은 `elem.class`처럼 사용할 수 있지만 그시절엔 안됐었습니다.

그래서 비슷한 `"className"`이 나왔습니다. `"elem.className"`은 `"class"`속성과 일치합니다.

ex):

```html run
<body class="main page">
  <script>
    alert(document.body.className); // main page
  </script>
</body>
```

`elem.className`에 무언가를 할당하면 클래스속성의 전체 문자열이 바뀝니다. 어쩔때는 전체 문자열이 바뀌는 것을 원할 때도 있죠. 그렇지만 대개 우리는 클래스 하나하나를 직접 추가/삭제 하고싶어합니다.

이를 위한 다른 속성이 있습니다. 바로 `elem.classList` 이죠.

`elem.classList`는 추가/삭제/토글 메소드가 있는 특별한 객체입니다. (원문: special object)

ex):

```html run
<body class="main page">
  <script>
*!*
    // add a class
    document.body.classList.add('article');
*/!*

    alert(document.body.className); // main page article
  </script>
</body>
```

이런식으로 우리는 `className`속성을 이용해서 클래스 전체 문자열을 직접 조작할 수 있고, `classList`속성을 이용하여 각각의 클래스들을 따로 조작할 수도 있습니다. 알맞는 상황에 골라서 쓰면 됩니다.

`classList`의 메소드들:

- `elem.classList.add("class")` -- 클래스 추가
- `elem.classList.remove("class")` -- 클래스 제거
- `elem.classList.toggle("class")` -- 있으면 제거 없으면 추가
- `elem.classList.contains("class")` -- 있는지 없는지 검사 반환값:`true/false`.

게다가 `classList`는 `"iterable"`이기 때문에 `for..of`문으로 돌릴 수 있습니다.

ex)

```html run
<body class="main page">
  <script>
    for (let name of document.body.classList) {
      alert(name); // main, and then page
    }
  </script>
</body>
```

## Element style

`elem.style`속성은 HTML 요소의 `"style"`속성에 적용되는 객체입니다. `elem.style.width = "100px"`을 하면 HTML요소의 style 속성에 인라인으로  `width: 100px;`이 들어갑니다.

속성에 `-`가 들어가는 CSS속성들은 camelCase로 씁니다.

camelCase: 두번째 단어의 앞을자를 대문자로하는 방식, 낙타의 혹과 비슷해서 이렇게 불림
snakeCase: 두번째 단어의 앞에 `_`를 넣는 방식, 뱀처럼 바닥을 기어다니는것 같아서 이렇게 불림

```js no-beautify
background-color  => elem.style.backgroundColor
z-index           => elem.style.zIndex
border-left-width => elem.style.borderLeftWidth
```

ex)

```js run
document.body.style.backgroundColor = prompt('background color?', 'green');
```

````smart header="Prefixed properties"
Browser-prefixed properties like `-moz-border-radius`, `-webkit-border-radius` also follow the same rule: a dash means upper case.

For instance:

```js
button.style.MozBorderRadius = '5px';
button.style.WebkitBorderRadius = '5px';
```
````

## 스타일 속성 재설정

가끔씩 스타일속성을 적용시켰다가 지우고싶을때도 있습니다.

예를들어 HTML요소를 숨기기 위하여 `elem.style.display = "none"`을 할 수도 있습니다.

그리고 후에 `style.display`속성을 처음부터 적용되어있지 않았던 것처럼 지워버리고 싶을 수 도 있습니다. `delete elem.style.display`를 하는 대신에 `elem.style.display = ""`와같이 빈 문자열을 집어넣습니다.

옮긴이: `elem.style.removeProperty("display");` 이렇게 js로 추가된 속성을 지워주는 메소드가 있습니다.

```js run
// if we run this code, the <body> will blink
document.body.style.display = "none"; // hide

setTimeout(() => document.body.style.display = "", 1000); // back to normal
```

`style.display`속성에 빈 문자열을 할당하게 되면 `style.display`속성이 아이에 없었던 것처럼 브라우저가 알아서 CSS를 적용합니다.

````smart header="Full rewrite with `style.cssText"
일반적으로, 우리는 각각의 style속성을 적용시키기 위해서 `style.*`을 사용합니다. `div.style="color: red; width: 100px"`과 같이 전체 스타일을 적용시킬 수 없습니다. 왜냐하면 `elem.style`은 읽기전용이기 때문입니다.

따라서 전체 style속성 텍스트를 변경하기 위해서 `style.cssText`라는 속성을 사용합니다.

```html run
<div id="div">Button</div>

<script>
  // we can set special style flags like "important" here
  div.style.cssText=`color: red !important;
    background-color: yellow;
    width: 100px;
    text-align: center;
  `;

  alert(div.style.cssText);
</script>
```

이 속성은 거의 쓰이지 않습니다. 왜냐하면 특정한 하나만 변경되거나 추가되는것이 아니라 전체텍스트가 대체되기 때문입니다. 따라서 필요한 속성도 같이 지워져버릴 수 있습니다. 그렇다고해서 꼭 사용하면 안된다는 것은 아닙니다. 새로운 HTML요소를 만들었을때 쓸 수 있습니다.

`div.setAttribute('style', 'color: red...')`로 같은 동작을 할 수 있습니다.
````

## 단위

적용하려는 값에 단위를 같이 써야합니다.

예를들어서 `elem.style.top`속성에 `10`을 넣으면 작동되지 않을 것입니다.

```html run height=100
<body>
  <script>
  *!*
    // doesn't work!
    document.body.style.margin = 20;
    alert(document.body.style.margin); // '' (empty string, the assignment is ignored)
  */!*

    // now add the CSS unit (px) - and it works
    document.body.style.margin = '20px';
    alert(document.body.style.margin); // 20px

    alert(document.body.style.marginTop); // 20px
    alert(document.body.style.marginLeft); // 20px
  </script>
</body>
````

유의: 브라우저가 마지막의 `style.margin`을 분석해서 `style.marginLeft`과 `style.marginTop`의 값을 넣습니다.

## 현재 적용되어 있는 속성: getComputedStyle

style속성을 변경하는것은 간한답니다. 읽어오는건 어떨까요?

예로, 우리가 HTML요소의 크기, 여백, 색상들을 알고싶어 합니다. 어떻게 할까요.

**JS의 `style`속성은 HTML요소에 인라인으로 적용된 style속성에 대해서만 작동합니다.**

그렇기 때문에 우리는 `elem.style`로 어떤것도 읽어올 수 없습니다.

이 예시에서는 `style`로 `margin`속성을 알 수 없습니다.

```html run height=60 no-beautify
<head>
  <style> body { color: red; margin: 5px } </style>
</head>
<body>

  The red text
  <script>
*!*
    alert(document.body.style.color); // empty
    alert(document.body.style.marginTop); // empty
*/!*
  </script>
</body>
```

우리는 현재 적용되어있는 값을 알고싶은 것입니다.

이를 위한 다른 메소드가 있습니다. 바로 `getComputedStyle`입니다.

문법:

```js
getComputedStyle(element, [pseudo])
```

element: 값을 읽어올 요소

pseudo: `::before`와 같은 pseudo element. 빈 문자열이나 입력하지 않으면 element자신을 가리킵니다.

결과는 `elem.style`같은 style들의 object일것입니다. 하지만 이번엔 다른 CSS가 적용된 것일것입니다..

ex)

```html run height=100
<head>
  <style> body { color: red; margin: 5px } </style>
</head>
<body>

  <script>
    let computedStyle = getComputedStyle(document.body);

    // now we can read the margin and the color from it

    alert( computedStyle.marginTop ); // 5px
    alert( computedStyle.color ); // rgb(255, 0, 0)
  </script>

</body>
```

```smart header="Computed and resolved values"
There are two concepts in [CSS](https://drafts.csswg.org/cssom/#resolved-values):

1. A *computed* style value is the value after all CSS rules and CSS inheritance is applied, as the  result of the CSS cascade. It can look like `height:1em` or `font-size:125%`.
2. A *resolved* style value is the one finally applied to the element. Values like `1em` or `125%` are relative. The browser takes the computed value and makes all units fixed and absolute, for instance: `height:20px` or `font-size:16px`. For geometry properties resolved values may have a floating point, like `width:50.5px`.

A long time ago `getComputedStyle` was created to get computed values, but it turned out that resolved values are much more convenient, and the standard changed.

So nowadays `getComputedStyle` actually returns the resolved value of the property, usually in `px` for geometry.
```

````warn header="`getComputedStyle` requires the full property name"
We should always ask for the exact property that we want, like `paddingLeft` or `marginTop` or `borderTopWidth`. Otherwise the correct result is not guaranteed.

For instance, if there are properties `paddingLeft/paddingTop`, then what should we get for `getComputedStyle(elem).padding`? Nothing, or maybe a "generated" value from known paddings? There's no standard rule here.

There are other inconsistencies. As an example, some browsers (Chrome) show `10px` in the document below, and some of them (Firefox) --  do not:

```html run
<style>
  body {
    margin: 10px;
  }
</style>
<script>
  let style = getComputedStyle(document.body);
  alert(style.margin); // empty string in Firefox
</script>
```
````

```smart header="Styles applied to `:visited` links are hidden!"
Visited links may be colored using `:visited` CSS pseudoclass.

But `getComputedStyle` does not give access to that color, because otherwise an arbitrary page could find out whether the user visited a link by creating it on the page and checking the styles.

JavaScript may not see the styles applied by `:visited`. And also, there's a limitation in CSS that forbids to apply geometry-changing styles in `:visited`. That's to guarantee that there's no sideway for an evil page to test if a link was visited and hence to break the privacy.
```

## Summary

To manage classes, there are two DOM properties:

- `className` -- the string value, good to manage the whole set of classes.
- `classList` -- the object with methods `add/remove/toggle/contains`, good for individual classes.

To change the styles:

- The `style` property is an object with camelCased styles. Reading and writing to it has the same meaning as modifying individual properties in the `"style"` attribute. To see how to apply `important` and other rare stuff -- there's a list of methods at [MDN](mdn:api/CSSStyleDeclaration).

- The `style.cssText` property corresponds to the whole `"style"` attribute, the full string of styles.

To read the resolved styles (with respect to all classes, after all CSS is applied and final values are calculated):

- The `getComputedStyle(elem, [pseudo])` returns the style-like object with them. Read-only.
