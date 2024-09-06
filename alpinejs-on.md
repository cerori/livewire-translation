# x-on

`x-on`은 디스패치된 DOM 이벤트에 대해 쉽게 코드를 실행할 수 있게 해줍니다.

다음은 클릭 시 알림을 표시하는 간단한 버튼의 예입니다.

```alpine
<button x-on:click="alert('안녕하세요!')">인사하기</button>
```

> `x-on`은 HTML 속성이 대소문자를 구분하지 않기 때문에 소문자 이름의 이벤트만 수신할 수 있습니다. `x-on:CLICK`을 작성하면 'click'이라는 이름의 이벤트를 수신합니다. camelCase 이름의 사용자 정의 이벤트를 수신해야 하는 경우, 이 제한을 해결하기 위해 [`.camel` 헬퍼](#camel)를 사용할 수 있습니다. 또는 [`x-bind`](/directives/bind#bind-directives)를 사용하여 자바스크립트 코드에서 요소에 `x-on` 디렉티브를 연결할 수 있습니다(이 경우 대소문자가 보존됩니다).

## 축약 문법

`x-on:`이 너무 장황하다고 생각되면 축약 문법 `@`를 사용할 수 있습니다.

다음은 위와 동일한 컴포넌트이지만 축약 문법을 사용한 예입니다:

```alpine
<button @click="alert('안녕하세요!')">인사하기</button>
```

## 이벤트 객체

표현식에서 네이티브 JavaScript 이벤트 객체에 접근하려면 Alpine의 매직 `$event` 속성을 사용할 수 있습니다.

```alpine
<button @click="alert($event.target.getAttribute('message'))" message="안녕하세요">인사하기</button>
```

또한 Alpine은 괄호 없이 참조된 모든 메서드에 이벤트 객체를 전달합니다. 예를 들어:

```alpine
<button @click="handleClick">...</button>

<script>
    function handleClick(e) {
        // 이제 이벤트 객체(e)에 직접 접근할 수 있습니다
    }
</script>
```

## 키보드 이벤트

Alpine은 특정 키에 대한 `keydown`과 `keyup` 이벤트를 쉽게 수신할 수 있게 해줍니다.

다음은 입력 요소 내에서 `Enter` 키를 수신하는 예입니다.

```alpine
<input type="text" @keyup.enter="alert('제출되었습니다!')">
```

이러한 키 수식어를 연결하여 더 복잡한 리스너를 만들 수도 있습니다.

다음은 `Shift` 키가 눌린 상태에서 `Enter`가 눌렸을 때 실행되는 리스너의 예입니다 (`Enter`만 눌렸을 때는 실행되지 않습니다).

```alpine
<input type="text" @keyup.shift.enter="alert('제출되었습니다!')">
```

[`KeyboardEvent.key`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values)를 통해 노출되는 모든 유효한 키 이름을 kebab-case로 변환하여 수식어로 직접 사용할 수 있습니다.

```alpine
<input type="text" @keyup.page-down="alert('제출되었습니다!')">
```

참고를 위해, 다음은 자주 사용할 수 있는 키 목록입니다.

| 수식어 | 키보드 키 |
|--------|-----------|
| `.shift` | Shift |
| `.enter` | Enter |
| `.space` | Space |
| `.ctrl` | Ctrl |
| `.cmd` | Cmd |
| `.meta` | Mac에서는 Cmd, Windows에서는 Windows 키 |
| `.alt` | Alt |
| `.up` `.down` `.left` `.right` | 위/아래/왼쪽/오른쪽 화살표 |
| `.escape` | Escape |
| `.tab` | Tab |
| `.caps-lock` | Caps Lock |
| `.equal` | Equal, `=` |
| `.period` | Period, `.` |
| `.comma` | Comma, `,` |
| `.slash` | 슬래시, `/` |

## 마우스 이벤트

위의 키보드 이벤트와 마찬가지로, Alpine은 `click` 이벤트를 처리하기 위한 일부 키 수식어 사용을 허용합니다.

| 수식어 | 이벤트 키 |
|--------|-----------|
| `.shift` | shiftKey |
| `.ctrl` | ctrlKey |
| `.cmd` | metaKey |
| `.meta` | metaKey |
| `.alt` | altKey |

이는 `click`, `auxclick`, `context`, `dblclick` 이벤트에서 작동하며, `mouseover`, `mousemove`, `mouseenter`, `mouseleave`, `mouseout`, `mouseup`, `mousedown` 이벤트에서도 작동합니다.

다음은 `Shift` 키가 눌린 상태에서 동작이 변경되는 버튼의 예입니다.

```alpine
<button type="button"
    @click="message = '선택됨'"
    @click.shift="message = '선택에 추가됨'"
    @mousemove.shift="message = '선택에 추가'"
    @mouseout="message = '선택'"
    x-text="message"></button>
```

> 참고: 일부 수식어(예: `ctrl`)가 있는 일반 클릭 이벤트는 대부분의 브라우저에서 자동으로 `contextmenu` 이벤트가 됩니다. 마찬가지로, `right-click` 이벤트는 `contextmenu` 이벤트를 트리거하지만, `contextmenu` 이벤트가 방지된 경우 `auxclick` 이벤트도 트리거합니다.

## 사용자 정의 이벤트

Alpine 이벤트 리스너는 네이티브 DOM 이벤트 리스너의 래퍼입니다. 따라서 사용자 정의 이벤트를 포함한 모든 DOM 이벤트를 수신할 수 있습니다.

다음은 사용자 정의 DOM 이벤트를 디스패치하고 수신하는 컴포넌트의 예입니다.

```alpine
<div x-data @foo="alert('버튼이 클릭되었습니다!')">
    <button @click="$event.target.dispatchEvent(new CustomEvent('foo', { bubbles: true }))">...</button>
</div>
```

버튼이 클릭되면 `@foo` 리스너가 호출됩니다.

`.dispatchEvent` API가 장황하기 때문에, Alpine은 이를 단순화하기 위해 `$dispatch` 헬퍼를 제공합니다.

다음은 `$dispatch` 매직 속성을 사용하여 다시 작성된 동일한 컴포넌트입니다.

```alpine
<div x-data @foo="alert('버튼이 클릭되었습니다!')">
    <button @click="$dispatch('foo')">...</button>
</div>
```

[→ `$dispatch`에 대해 더 읽기](/magics/dispatch)

## 수식어

Alpine은 이벤트 리스너의 동작을 사용자 정의할 수 있는 여러 디렉티브 수식어를 제공합니다.

### .prevent

`.prevent`는 브라우저 이벤트 객체에서 `.preventDefault()`를 호출하는 것과 동일합니다.

```alpine
<form @submit.prevent="console.log('제출됨')" action="/foo">
    <button>제출</button>
</form>
```

위의 예에서 `.prevent`를 사용하면 버튼을 클릭해도 폼이 `/foo` 엔드포인트로 제출되지 않습니다. 대신 Alpine의 리스너가 이를 처리하고 이벤트가 더 이상 처리되는 것을 "방지"합니다.

### .stop

`.prevent`와 유사하게, `.stop`은 브라우저 이벤트 객체에서 `.stopPropagation()`을 호출하는 것과 동일합니다.

```alpine
<div @click="console.log('이것은 기록되지 않을 것입니다')">
    <button @click.stop>클릭하세요</button>
</div>
```

위의 예에서 버튼을 클릭해도 메시지가 기록되지 않습니다. 이는 이벤트의 전파를 즉시 중지하고 `<div>`의 `@click` 리스너로 "버블링"되는 것을 허용하지 않기 때문입니다.

### .outside

`.outside`는 연결된 요소 외부의 클릭을 수신하기 위한 편리한 헬퍼입니다. 다음은 이를 설명하기 위한 간단한 드롭다운 컴포넌트 예제입니다:

```alpine
<div x-data="{ open: false }">
    <button @click="open = ! open">토글</button>

    <div x-show="open" @click.outside="open = false">
        내용...
    </div>
</div>
```

위의 예에서, "토글" 버튼을 클릭하여 드롭다운 내용을 표시한 후, 내용 외부의 페이지 어디를 클릭해도 드롭다운을 닫을 수 있습니다.

이는 `.outside`가 등록된 요소에서 시작되지 않은 클릭을 수신하고 있기 때문입니다.

> 주목할 점은 `.outside` 표현식은 등록된 요소가 페이지에 표시될 때만 평가된다는 것입니다. 그렇지 않으면 보이지 않을 때 "토글" 버튼을 클릭하면 `@click.outside` 핸들러도 실행되는 불편한 경쟁 조건이 발생할 수 있습니다.

### .window

`.window` 수식어가 있으면 Alpine은 요소 자체가 아닌 페이지의 루트 `window` 객체에 이벤트 리스너를 등록합니다.

```alpine
<div @keyup.escape.window="...">...</div>
```

위의 스니펫은 페이지의 어디에서나 "escape" 키가 눌리는 것을 수신합니다.

리스너에 `.window`를 추가하는 것은 전체 페이지에서 발생하는 이벤트에 관심이 있는 마크업의 작은 부분에 대해 이런 종류의 경우에 매우 유용합니다.

### .document

`.document`는 `.window`와 유사하게 작동하지만 `window` 전역 대신 `document` 전역에 리스너를 등록합니다.

### .once

리스너에 `.once`를 추가하면 핸들러가 한 번만 호출되도록 보장합니다.

```alpine
<button @click.once="console.log('한 번만 기록됩니다')">...</button>
```

### .debounce

때로는 이벤트 핸들러를 "디바운스"하여 일정 기간의 비활성 상태(기본적으로 250밀리초) 후에만 호출되도록 하는 것이 유용할 수 있습니다.

예를 들어, 사용자가 입력할 때마다 네트워크 요청을 실행하는 검색 필드가 있다면, 디바운스를 추가하면 모든 키 입력마다 네트워크 요청이 실행되는 것을 방지할 수 있습니다.

```alpine
<input @input.debounce="fetchResults">
```

이제 모든 키 입력 후에 `fetchResults`를 호출하는 대신, 250밀리초 동안 키 입력이 없을 때만 `fetchResults`가 호출됩니다.

디바운스 시간을 늘리거나 줄이고 싶다면 다음과 같이 `.debounce` 수식어 뒤에 기간을 추가할 수 있습니다:

```alpine
<input @input.debounce.500ms="fetchResults">
```

이제 `fetchResults`는 500밀리초의 비활성 상태 후에만 호출됩니다.

### .throttle

`.throttle`은 `.debounce`와 유사하지만 무기한 연기하는 대신 250밀리초마다 핸들러 호출을 릴리스합니다.

이는 반복적이고 장기적인 이벤트 발생이 있고 `.debounce`를 사용할 수 없지만 여전히 때때로 이벤트를 처리하고 싶은 경우에 유용합니다.

예를 들어:

```alpine
<div @scroll.window.throttle="handleScroll">...</div>
```

위의 예는 스로틀링의 좋은 사용 사례입니다. `.throttle` 없이는 사용자가 페이지를 스크롤할 때 `handleScroll` 메서드가 수백 번 실행될 것입니다. 이는 사이트 속도를 크게 저하시킬 수 있습니다. `.throttle`을 추가함으로써 `handleScroll`이 250밀리초마다만 호출되도록 보장합니다.

> 흥미로운 사실: 이 정확한 전략이 이 문서 사이트에서 오른쪽 사이드바의 현재 강조된 섹션을 업데이트하는 데 사용됩니다.

`.debounce`와 마찬가지로 스로틀된 이벤트에 사용자 정의 기간을 추가할 수 있습니다:

```alpine
<div @scroll.window.throttle.750ms="handleScroll">...</div>
```

이제 `handleScroll`은 750밀리초마다만 호출됩니다.

네, 감사합니다. 계속해서 번역하겠습니다.

### .self

이벤트 리스너에 `.self`를 추가하면 이벤트가 선언된 요소에서 시작되었는지 확인하고, 자식 요소에서 시작되지 않았는지 확인합니다.

```alpine
<button @click.self="handleClick">
    클릭하세요

    <img src="...">
</button>
```

위의 예에서 우리는 `<button>` 태그 안에 `<img>` 태그가 있습니다. 일반적으로 `<button>` 요소 내부에서 시작된 모든 클릭(예: `<img>`에서의 클릭)은 버튼의 `@click` 리스너에 의해 감지됩니다.

그러나 이 경우에는 `.self`를 추가했기 때문에 버튼 자체를 클릭할 때만 `handleClick`이 호출됩니다. `<img>` 요소에서 시작된 클릭만 처리되지 않습니다.

### .camel

```alpine
<div @custom-event.camel="handleCustomEvent">
    ...
</div>
```

때로는 `customEvent`와 같은 camelCased 이벤트를 수신하고 싶을 수 있습니다. HTML 속성 내에서 camelCasing이 지원되지 않기 때문에 `.camel` 수식어를 추가하여 Alpine이 내부적으로 이벤트 이름을 camelCase로 변환하도록 해야 합니다.

위의 예에서 `.camel`을 추가함으로써 Alpine은 이제 `custom-event` 대신 `customEvent`를 수신합니다.

### .dot

```alpine
<div @custom-event.dot="handleCustomEvent">
    ...
</div>
```

`.camelCase` 수식어와 유사하게 이름에 점이 있는 이벤트(예: `custom.event`)를 수신하고 싶은 상황이 있을 수 있습니다. 이벤트 이름 내의 점은 Alpine에서 예약되어 있기 때문에 대시로 작성하고 `.dot` 수식어를 추가해야 합니다.

위의 코드 예제에서 `custom-event.dot`은 `custom.event` 이벤트 이름에 해당합니다.

### .passive

브라우저는 JavaScript가 페이지에서 실행 중일 때도 페이지 스크롤을 빠르고 부드럽게 최적화합니다. 그러나 제대로 구현되지 않은 터치 및 휠 리스너는 이 최적화를 차단하고 사이트 성능을 저하시킬 수 있습니다.

터치 이벤트를 수신하고 있다면, 스크롤 성능을 차단하지 않도록 리스너에 `.passive`를 추가하는 것이 중요합니다.

```alpine
<div @touchstart.passive="...">...</div>
```

[→ 패시브 리스너에 대해 더 읽기](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#improving_scrolling_performance_with_passive_listeners)

### .capture

이 수식어를 추가하면 이벤트의 캡처링 단계에서 이 리스너를 실행할 수 있습니다. 즉, 이벤트가 대상 요소에서 DOM을 통해 버블링되기 전에 실행됩니다.

```alpine
<div @click.capture="console.log('나는 먼저 로그될 것입니다')">
    <button @click="console.log('나는 두 번째로 로그될 것입니다')"></button>
</div>
```

[→ 이벤트의 캡처링 및 버블링 단계에 대해 더 읽기](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#usecapture)
