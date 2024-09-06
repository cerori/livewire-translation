네, 제공해주신 Livewire 문서를 한국어로 번역해 드리겠습니다. 프로그래밍 용어는 적절히 원문을 유지하면서 번역하겠습니다.

# 중첩

다른 많은 컴포넌트 기반 프레임워크와 마찬가지로, Livewire 컴포넌트도 중첩이 가능합니다 - 즉, 하나의 컴포넌트가 그 안에 여러 컴포넌트를 렌더링할 수 있습니다.

그러나 Livewire의 중첩 시스템은 다른 프레임워크와는 다르게 구축되어 있기 때문에, 알아두어야 할 특정한 영향과 제약사항들이 있습니다.

> 먼저 하이드레이션을 이해해야 합니다
>
> Livewire의 중첩 시스템에 대해 더 자세히 배우기 전에, Livewire가 컴포넌트를 어떻게 하이드레이션하는지 완전히 이해하는 것이 도움이 됩니다. [하이드레이션 문서](/docs/hydration)를 읽어보시면 더 자세히 알 수 있습니다.

## 모든 컴포넌트는 섬과 같습니다

Livewire에서는 페이지의 모든 컴포넌트가 다른 컴포넌트와 독립적으로 자신의 상태를 추적하고 업데이트합니다.

예를 들어, 다음과 같은 `Posts`와 중첩된 `ShowPost` 컴포넌트를 고려해 보겠습니다:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;

class Posts extends Component
{
    public $postLimit = 2;

    public function render()
    {
        return view('livewire.posts', [
            'posts' => Auth::user()->posts()
                ->limit($this->postLimit)->get(),
        ]);
    }
}
```

```blade
<div>
    Post Limit: <input type="number" wire:model.live="postLimit">

    @foreach ($posts as $post)
        <livewire:show-post :$post :key="$post->id">
    @endforeach
</div>
```

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Post;

class ShowPost extends Component
{
    public Post $post;

    public function render()
    {
        return view('livewire.show-post');
    }
}
```

```blade
<div>
    <h1>{{ $post->title }}</h1>

    <p>{{ $post->content }}</p>

    <button wire:click="$refresh">Refresh post</button>
</div>
```

초기 페이지 로드 시 전체 컴포넌트 트리의 HTML은 다음과 같을 수 있습니다:

```html
<div wire:id="123" wire:snapshot="...">
  Post Limit: <input type="number" wire:model.live="postLimit" />

  <div wire:id="456" wire:snapshot="...">
    <h1>The first post</h1>

    <p>Post content</p>

    <button wire:click="$refresh">Refresh post</button>
  </div>

  <div wire:id="789" wire:snapshot="...">
    <h1>The second post</h1>

    <p>Post content</p>

    <button wire:click="$refresh">Refresh post</button>
  </div>
</div>
```

부모 컴포넌트가 자신의 렌더링된 템플릿과 그 안에 중첩된 모든 컴포넌트의 렌더링된 템플릿을 포함하고 있음을 주목하세요.

각 컴포넌트가 독립적이기 때문에, Livewire의 JavaScript 코어가 추출하고 추적할 수 있도록 각각 고유한 ID와 스냅샷(`wire:id`와 `wire:snapshot`)이 HTML에 포함되어 있습니다.

중첩의 다양한 수준에서 Livewire가 어떻게 다르게 처리하는지 보기 위해 몇 가지 다른 업데이트 시나리오를 고려해 보겠습니다.

### 자식 업데이트하기

자식 `show-post` 컴포넌트 중 하나에서 "Refresh post" 버튼을 클릭하면, 서버로 다음과 같은 내용이 전송됩니다:

```js
{
    memo: { name: 'show-post', id: '456' },
    state: { ... },
}
```

다음은 반환되는 HTML입니다:

```html
<div wire:id="456">
  <h1>The first post</h1>

  <p>Post content</p>

  <button wire:click="$refresh">Refresh post</button>
</div>
```

여기서 중요한 점은 자식 컴포넌트에서 업데이트가 트리거되면, 해당 컴포넌트의 데이터만 서버로 전송되고 해당 컴포넌트만 다시 렌더링된다는 것입니다.

이제 덜 직관적인 시나리오인 부모 컴포넌트 업데이트를 살펴보겠습니다.

### 부모 업데이트하기

다시 한번 부모 `Posts` 컴포넌트의 Blade 템플릿을 상기해 보겠습니다:

```blade
<div>
    Post Limit: <input type="number" wire:model.live="postLimit">

    @foreach ($posts as $post)
        <livewire:show-post :$post :key="$post->id">
    @endforeach
</div>
```

사용자가 "Post Limit" 값을 `2`에서 `1`로 변경하면, 부모에 대해서만 업데이트가 트리거됩니다.

다음은 요청 페이로드의 예시입니다:

```js
{
    updates: { postLimit: 1 },
    snapshot: {
        memo: { name: 'posts', id: '123' },
        state: { postLimit: 2, ... },
    },
}
```

보시다시피, 부모 `Posts` 컴포넌트의 스냅샷만 서버로 전송됩니다.

여기서 중요한 질문은: 부모 컴포넌트가 다시 렌더링되고 자식 `show-post` 컴포넌트를 만났을 때 어떻게 되는가? 요청 페이로드에 자식 컴포넌트의 스냅샷이 포함되지 않았는데 어떻게 자식 컴포넌트를 다시 렌더링할 수 있을까?

답은: 다시 렌더링되지 않습니다.

Livewire가 `Posts` 컴포넌트를 렌더링할 때, 만나는 모든 자식 컴포넌트에 대해 플레이스홀더를 렌더링합니다.

위의 업데이트 후 `Posts` 컴포넌트의 렌더링된 HTML 예시는 다음과 같습니다:

```html
<div wire:id="123">
  Post Limit: <input type="number" wire:model.live="postLimit" />

  <div wire:id="456"></div>
</div>
```

보시다시피, `postLimit`가 `1`로 업데이트되었기 때문에 하나의 자식만 렌더링되었습니다. 그러나 전체 자식 컴포넌트 대신 일치하는 `wire:id` 속성을 가진 빈 `<div></div>`만 있다는 것을 알 수 있습니다.

이 HTML이 프론트엔드에서 수신되면, Livewire는 이 컴포넌트의 이전 HTML을 이 새로운 HTML로 *변형(morph)*시키지만, 모든 자식 컴포넌트 플레이스홀더는 지능적으로 건너뜁니다.

결과적으로, _변형_ 후 부모 `Posts` 컴포넌트의 최종 DOM 내용은 다음과 같습니다:

```html
<div wire:id="123">
  Post Limit: <input type="number" wire:model.live="postLimit" />

  <div wire:id="456">
    <h1>The first post</h1>

    <p>Post content</p>

    <button wire:click="$refresh">Refresh post</button>
  </div>
</div>
```

## 성능에 미치는 영향

Livewire의 "섬" 아키텍처는 애플리케이션에 긍정적 및 부정적 영향을 모두 미칠 수 있습니다.

이 아키텍처의 장점은 애플리케이션의 비용이 많이 드는 부분을 격리할 수 있다는 것입니다. 예를 들어, 느린 데이터베이스 쿼리를 독립적인 컴포넌트로 격리하면, 그 성능 오버헤드가 페이지의 나머지 부분에 영향을 미치지 않습니다.

그러나 이 접근 방식의 가장 큰 단점은 컴포넌트가 완전히 분리되어 있기 때문에 컴포넌트 간 통신/의존성이 더 어려워진다는 것입니다.

예를 들어, 위의 부모 `Posts` 컴포넌트에서 중첩된 `ShowPost` 컴포넌트로 전달된 속성이 있다면, 이는 "반응형"이 아닐 것입니다. 각 컴포넌트가 섬이기 때문에, 부모 컴포넌트에 대한 요청이 `ShowPost`로 전달되는 속성의 값을 변경하더라도 `ShowPost` 내부에서는 업데이트되지 않을 것입니다.

Livewire는 이러한 여러 장애물을 극복하고 이러한 시나리오를 위한 전용 API를 제공합니다. 예를 들어 [반응형 속성](/docs/nesting#reactive-props), [모델화 가능한 컴포넌트](/docs/nesting#binding-to-child-data-using-wiremodel), 그리고 [`$parent` 객체](/docs/nesting#directly-accessing-the-parent-from-the-child) 등이 있습니다.

중첩된 Livewire 컴포넌트가 어떻게 작동하는지에 대한 이 지식을 바탕으로, 애플리케이션 내에서 언제 어떻게 컴포넌트를 중첩할지에 대해 더 정보에 입각한 결정을 내릴 수 있을 것입니다.
