# Alpine

AlpineJS는 웹 페이지에 클라이언트 측 상호작용을 쉽게 추가할 수 있게 해주는 경량 JavaScript 라이브러리입니다. 원래 Livewire와 같은 도구를 보완하기 위해 만들어졌으며, JavaScript 중심의 유틸리티가 앱에 상호작용을 추가하는 데 도움이 됩니다.

Livewire는 Alpine을 기본적으로 제공하므로 프로젝트에 별도로 설치할 필요가 없습니다.

AlpineJS 사용에 대해 배우기 가장 좋은 곳은 [Alpine 문서](https://alpinejs.dev)입니다.

## 기본적인 Alpine 컴포넌트

이 문서의 나머지 부분을 위한 기초를 다지기 위해, 가장 간단하고 유익한 Alpine 컴포넌트 예제 중 하나를 소개합니다. 페이지에 숫자를 표시하고 사용자가 버튼을 클릭하여 그 숫자를 증가시킬 수 있는 작은 "카운터"입니다:

```html
<!-- JavaScript 데이터 객체 선언... -->
<div x-data="{ count: 0 }">
  <!-- 현재 "count" 값을 요소 안에 렌더링... -->
  <h2 x-text="count"></h2>

  <!-- 클릭 이벤트가 발생하면 "count" 값을 "1" 증가... -->
  <button x-on:click="count++">+</button>
</div>
```

위의 Alpine 컴포넌트는 애플리케이션의 모든 Livewire 컴포넌트 내에서 문제 없이 사용할 수 있습니다. Livewire는 Livewire 컴포넌트 업데이트 전반에 걸쳐 Alpine의 상태를 유지합니다. 본질적으로, Livewire가 아닌 다른 컨텍스트에서 Alpine을 사용하는 것처럼 Livewire 내에서 Alpine 컴포넌트를 자유롭게 사용해야 합니다.

## Livewire 내에서 Alpine 사용하기

Livewire 컴포넌트 내에서 Alpine 컴포넌트를 사용하는 더 실제적인 예를 살펴보겠습니다.

아래는 데이터베이스의 게시물 모델 세부 정보를 보여주는 간단한 Livewire 컴포넌트입니다. 기본적으로 게시물의 제목만 표시됩니다:

```html
<div>
  <h1>{{ $post->title }}</h1>

  <div x-data="{ expanded: false }">
    <button type="button" x-on:click="expanded = ! expanded">
      <span x-show="! expanded">게시물 내용 보기...</span>
      <span x-show="expanded">게시물 내용 숨기기...</span>
    </button>

    <div x-show="expanded">{{ $post->content }}</div>
  </div>
</div>
```

Alpine을 사용하여 사용자가 "게시물 내용 보기..." 버튼을 누를 때까지 게시물의 내용을 숨길 수 있습니다. 그 시점에서 Alpine의 `expanded` 속성이 `true`로 설정되고, `x-show="expanded"`가 게시물 내용의 가시성을 Alpine이 제어하도록 사용되기 때문에 내용이 페이지에 표시됩니다.

이는 Alpine이 빛을 발하는 예시입니다: Livewire 서버 라운드트립을 트리거하지 않고 애플리케이션에 상호작용을 추가합니다.

## `$wire`를 사용하여 Alpine에서 Livewire 제어하기

Livewire 개발자로서 사용할 수 있는 가장 강력한 기능 중 하나는 `$wire`입니다. `$wire` 객체는 Livewire 내에서 사용되는 모든 Alpine 컴포넌트에서 사용할 수 있는 마법 객체입니다.

`$wire`를 JavaScript에서 PHP로 가는 게이트웨이로 생각할 수 있습니다. Livewire 컴포넌트 속성에 접근하고 수정하며, Livewire 컴포넌트 메서드를 호출하고, AlpineJS 내에서 더 많은 작업을 할 수 있게 해줍니다.

### Livewire 속성 접근하기

다음은 게시물 작성을 위한 폼에서 간단한 "문자 수" 유틸리티의 예입니다. 사용자가 입력할 때 게시물 내용에 포함된 문자 수를 즉시 보여줍니다:

```html
<form wire:submit="save">
  <!-- ... -->

  <input wire:model="content" type="text" />

  <small> 문자 수: <span x-text="$wire.content.length"></span> </small>

  <button type="submit">저장</button>
</form>
```

위 예제에서 볼 수 있듯이 `x-text`는 Alpine이 `<span>` 요소의 텍스트 내용을 제어할 수 있게 합니다. `x-text`는 내부에 모든 JavaScript 표현식을 받아들이고 의존성이 업데이트될 때마다 자동으로 반응합니다. `$wire.content`를 사용하여 `$content` 값에 접근하기 때문에, Alpine은 Livewire에서 `$wire.content`가 업데이트될 때마다 자동으로 텍스트 내용을 업데이트합니다. 이 경우에는 `wire:model="content"`에 의해 업데이트됩니다.

### Livewire 속성 변경하기

다음은 게시물 작성 폼의 "제목" 필드를 지우기 위해 Alpine 내에서 `$wire`를 사용하는 예입니다.

```html
<form wire:submit="save">
  <input wire:model="title" type="text" />

  <button type="button" x-on:click="$wire.title = ''">지우기</button>

  <!-- ... -->

  <button type="submit">저장</button>
</form>
```

사용자가 위의 Livewire 폼을 작성하는 동안 "지우기"를 누르면 Livewire에서 네트워크 요청을 보내지 않고 제목 필드가 지워집니다. 상호작용은 "즉각적"일 것입니다.

이것이 가능한 이유에 대한 간단한 설명은 다음과 같습니다:

- `x-on:click`은 Alpine에게 버튼 요소의 클릭을 수신하도록 지시합니다
- 클릭되면 Alpine은 제공된 JS 표현식을 실행합니다: `$wire.title = ''`
- `$wire`는 Livewire 컴포넌트를 나타내는 마법 객체이기 때문에, 컴포넌트의 모든 속성에 JavaScript에서 직접 접근하거나 변경할 수 있습니다
- `$wire.title = ''`는 Livewire 컴포넌트의 `$title` 값을 빈 문자열로 설정합니다
- `wire:model`과 같은 모든 Livewire 유틸리티는 서버 라운드트립을 보내지 않고 이 변경에 즉시 반응할 것입니다
- 다음 Livewire 네트워크 요청에서 `$title` 속성은 백엔드에서 빈 문자열로 업데이트될 것입니다

### Livewire 메서드 호출하기

Alpine은 `$wire`에서 직접 호출하여 모든 Livewire 메서드/액션을 쉽게 호출할 수 있습니다.

다음은 Alpine을 사용하여 입력의 "blur" 이벤트를 수신하고 폼 저장을 트리거하는 예입니다. "blur" 이벤트는 사용자가 현재 요소에서 포커스를 제거하고 페이지의 다음 요소로 포커스를 이동하기 위해 "탭"을 누를 때 브라우저에 의해 발송됩니다:

```html
<form wire:submit="save">
  <input wire:model="title" type="text" x-on:blur="$wire.save()" />

  <!-- ... -->

  <button type="submit">저장</button>
</form>
```

일반적으로 이 상황에서는 `wire:model.blur="title"`을 사용하겠지만, Alpine을 사용하여 이를 어떻게 달성할 수 있는지 보여주는 데 도움이 됩니다.

#### 매개변수 전달하기

`$wire` 메서드 호출에 매개변수를 단순히 전달하여 Livewire 메서드에 매개변수를 전달할 수도 있습니다.

다음과 같은 `deletePost()` 메서드가 있는 컴포넌트를 고려해 봅시다:

```php
public function deletePost($postId)
{
    $post = Post::find($postId);

    // 사용자가 삭제할 수 있는지 권한 확인...
    auth()->user()->can('update', $post);

    $post->delete();
}
```

이제 Alpine에서 `deletePost()` 메서드에 `$postId` 매개변수를 다음과 같이 전달할 수 있습니다:

```html
<button type="button" x-on:click="$wire.deletePost(1)"></button>
```

일반적으로 `$postId`와 같은 것은 Blade에서 생성됩니다. 다음은 Blade를 사용하여 Alpine이 `deletePost()`에 전달하는 `$postId`를 결정하는 예입니다:

```html
@foreach ($posts as $post)
<button type="button" x-on:click="$wire.deletePost({{ $post->id }})">
  "{{ $post->title }}" 삭제
</button>
@endforeach
```

페이지에 세 개의 게시물이 있다면, 위의 Blade 템플릿은 브라우저에서 다음과 같이 렌더링될 것입니다:

```html
<button type="button" x-on:click="$wire.deletePost(1)">"걷기의 힘" 삭제</button>

<button type="button" x-on:click="$wire.deletePost(2)">
  "노래 녹음 방법" 삭제
</button>

<button type="button" x-on:click="$wire.deletePost(3)">
  "배운 것을 가르치기" 삭제
</button>
```

보시다시피, Blade를 사용하여 Alpine `x-on:click` 표현식에 다른 게시물 ID를 렌더링했습니다.

#### Blade 매개변수 주의사항

이는 매우 강력한 기술이지만, Blade 템플릿을 읽을 때 혼란스러울 수 있습니다. 처음에는 어느 부분이 Blade이고 어느 부분이 Alpine인지 구분하기 어려울 수 있습니다. 따라서 페이지에 렌더링된 HTML을 검사하여 예상한 대로 렌더링되었는지 확인하는 것이 도움이 됩니다.

다음은 사람들을 흔히 혼란스럽게 하는 예입니다:

Post 모델이 인덱스로 ID 대신 UUID를 사용한다고 가정해 봅시다(ID는 정수이고, UUID는 긴 문자열입니다).

ID를 사용했던 것처럼 다음과 같이 렌더링하면 문제가 발생합니다:

```html
<!-- 경고: 이는 문제가 있는 코드의 예시입니다... -->
<button type="button" x-on:click="$wire.deletePost({{ $post->uuid }})"></button>
```

위의 Blade 템플릿은 HTML에 다음과 같이 렌더링됩니다:

```html
<!-- 경고: 이는 문제가 있는 코드의 예시입니다... -->
<button
  type="button"
  x-on:click="$wire.deletePost(93c7b04c-c9a4-4524-aa7d-39196011b81a)"
></button>
```

UUID 문자열 주변에 따옴표가 없다는 것을 주목하세요. Alpine이 이 표현식을 평가하려고 할 때, JavaScript는 오류를 던질 것입니다: "Uncaught SyntaxError: Invalid or unexpected token".

이를 수정하려면 Blade 표현식 주변에 따옴표를 추가해야 합니다:

```html
<button
  type="button"
  x-on:click="$wire.deletePost('{{ $post->uuid }}')"
></button>
```

이제 위의 템플릿이 제대로 렌더링되고네, 모든 것이 예상대로 작동할 것입니다:

```html
<button
  type="button"
  x-on:click="$wire.deletePost('93c7b04c-c9a4-4524-aa7d-39196011b81a')"
></button>
```

### 컴포넌트 새로고침하기

`$wire.$refresh()`를 사용하여 Livewire 컴포넌트를 쉽게 새로고침할 수 있습니다(컴포넌트의 Blade 뷰를 다시 렌더링하기 위한 네트워크 라운드트립 트리거):

```html
<button type="button" x-on:click="$wire.$refresh()"></button>
```

## `$wire.entangle`을 사용하여 상태 공유하기

대부분의 경우 Alpine에서 Livewire 상태와 상호 작용하기 위해 `$wire`만 있으면 충분합니다. 그러나 Livewire는 Livewire의 값을 Alpine의 값과 동기화 상태로 유지하는 데 사용할 수 있는 추가적인 `$wire.entangle()` 유틸리티를 제공합니다.

이를 보여주기 위해 `$wire.entangle()`을 사용하여 Livewire와 Alpine 사이에 `showDropdown` 속성이 얽혀 있는 이 드롭다운 예제를 고려해 보십시오. 얽힘을 사용함으로써 이제 Alpine과 Livewire 모두에서 드롭다운의 상태를 제어할 수 있습니다:

```php
use Livewire\Component;

class PostDropdown extends Component
{
    public $showDropdown = false;

    public function archive()
    {
        // ...

        $this->showDropdown = false;
    }

    public function delete()
    {
        // ...

        $this->showDropdown = false;
    }
}
```

```blade
<div x-data="{ open: $wire.entangle('showDropdown') }">
    <button x-on:click="open = true">더 보기...</button>

    <ul x-show="open" x-on:click.outside="open = false">
        <li><button wire:click="archive">보관</button></li>

        <li><button wire:click="delete">삭제</button></li>
    </ul>
</div>
```

이제 사용자는 Alpine으로 드롭다운을 즉시 토글할 수 있지만, "보관"과 같은 Livewire 액션을 클릭하면 Livewire에서 드롭다운을 닫으라고 지시합니다. Alpine과 Livewire 모두 각자의 속성을 조작할 수 있으며, 다른 쪽은 자동으로 업데이트됩니다.

기본적으로 상태 업데이트는 지연됩니다(클라이언트에서는 변경되지만 서버에서는 즉시 변경되지 않음). 다음 Livewire 요청까지 지연됩니다. 사용자가 클릭하는 즉시 서버 측 상태를 업데이트해야 하는 경우, `.live` 수식어를 다음과 같이 연결하십시오:

```blade
<div x-data="{ open: $wire.entangle('showDropdown').live }">
    ...
</div>
```

> **`$wire.entangle`이 필요하지 않을 수 있습니다**
>
> 대부분의 경우 Alpine에서 Livewire 속성에 직접 접근하기 위해 `$wire`를 사용하는 것으로 원하는 바를 달성할 수 있습니다. 두 속성을 얽히게 하는 것보다 하나에 의존하는 것이 예측 가능성과 성능 문제를 야기할 수 있습니다. 특히 자주 변경되는 깊게 중첩된 객체를 사용할 때 그렇습니다. 이러한 이유로 Livewire 버전 3부터 `$wire.entangle`은 문서에서 강조되지 않았습니다.

> **@entangle 디렉티브 사용을 자제하세요**
>
> Livewire 버전 2에서는 Blade의 `@entangle` 디렉티브를 사용하는 것이 권장되었습니다. 그러나 v3에서는 더 이상 그렇지 않습니다. `$wire.entangle()`가 더 강력한 유틸리티이며 [DOM 요소 제거 시 특정 문제](https://github.com/livewire/livewire/pull/6833#issuecomment-1902260844)를 피할 수 있기 때문에 선호됩니다.

## JavaScript 빌드에 Alpine 수동으로 번들링하기

기본적으로 Livewire와 Alpine의 JavaScript는 각 Livewire 페이지에 자동으로 주입됩니다.

이는 간단한 설정에 이상적이지만, 프로젝트에 자체 Alpine 컴포넌트, 스토어, 플러그인을 포함하고 싶을 수 있습니다.

페이지에 자체 JavaScript 번들을 통해 Livewire와 Alpine을 포함하는 것은 간단합니다.

먼저 레이아웃 파일에 `@livewireScriptConfig` 디렉티브를 다음과 같이 포함해야 합니다:

```blade
<html>
<head>
    <!-- ... -->
    @livewireStyles
    @vite(['resources/js/app.js'])
</head>
<body>
    {{ $slot }}

    @livewireScriptConfig
</body>
</html>
```

이를 통해 Livewire는 앱이 제대로 실행되는 데 필요한 특정 구성을 번들에 제공할 수 있습니다.

이제 `resources/js/app.js` 파일에서 Livewire와 Alpine을 다음과 같이 가져올 수 있습니다:

```js
import {
  Livewire,
  Alpine,
} from "../../vendor/livewire/livewire/dist/livewire.esm";

// 여기에 Alpine 디렉티브, 컴포넌트 또는 플러그인을 등록하세요...

Livewire.start();
```

다음은 애플리케이션에 "x-clipboard"라는 사용자 정의 Alpine 디렉티브를 등록하는 예입니다:

```js
import {
  Livewire,
  Alpine,
} from "../../vendor/livewire/livewire/dist/livewire.esm";

Alpine.directive("clipboard", (el) => {
  let text = el.textContent;

  el.addEventListener("click", () => {
    navigator.clipboard.writeText(text);
  });
});

Livewire.start();
```

이제 `x-clipboard` 디렉티브는 Livewire 애플리케이션의 모든 Alpine 컴포넌트에서 사용할 수 있습니다.
