# Actions

Livewire 액션은 버튼 클릭이나 폼 제출과 같은 프론트엔드 상호작용에 의해 트리거될 수 있는 컴포넌트의 메서드입니다. 이를 통해 개발자는 브라우저에서 직접 PHP 메서드를 호출하는 것과 같은 경험을 할 수 있어, 애플리케이션의 프론트엔드와 백엔드를 연결하는 상용구 코드 작성에 시간을 낭비하지 않고 애플리케이션의 로직에 집중할 수 있습니다.

`CreatePost` 컴포넌트에서 `save` 액션을 호출하는 기본적인 예제를 살펴보겠습니다:

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class CreatePost extends Component
{
    public $title = '';
    public $content = '';

    public function save()
    {
        Post::create([
            'title' => $this->title,
            'content' => $this->content,
        ]);

        return redirect()->to('/posts');
    }

    public function render()
    {
        return view('livewire.create-post');
    }
}
```

```blade
<form wire:submit="save">
    <input type="text" wire:model="title">
    <textarea wire:model="content"></textarea>
    <button type="submit">Save</button>
</form>
```

위의 예제에서 사용자가 "Save" 버튼을 클릭하여 폼을 제출하면, `wire:submit`이 `submit` 이벤트를 가로채고 서버의 `save()` 액션을 호출합니다.

본질적으로 액션은 사용자 상호작용을 서버 측 기능에 쉽게 매핑할 수 있게 해주며, AJAX 요청을 수동으로 제출하고 처리하는 번거로움 없이 사용할 수 있습니다.

## 컴포넌트 새로고침

때때로 컴포넌트의 단순한 "새로고침"을 트리거하고 싶을 수 있습니다. 예를 들어, 데이터베이스의 어떤 것의 상태를 확인하는 컴포넌트가 있다면, 사용자에게 표시된 결과를 새로고침할 수 있는 버튼을 보여주고 싶을 수 있습니다.

이는 Livewire의 간단한 `$refresh` 액션을 사용하여 일반적으로 컴포넌트 메서드를 참조하는 곳 어디에서나 할 수 있습니다:

```blade
<button type="button" wire:click="$refresh">...</button>
```

`$refresh` 액션이 트리거되면, Livewire는 서버 왕복을 하고 어떤 메서드도 호출하지 않고 컴포넌트를 다시 렌더링합니다.

컴포넌트의 보류 중인 데이터 업데이트(예: `wire:model` 바인딩)는 컴포넌트가 새로고침될 때 서버에 적용된다는 점에 주의하는 것이 중요합니다.

내부적으로 Livewire는 Livewire 컴포넌트가 서버에서 업데이트될 때마다 "commit"이라는 용어를 사용합니다. 이 용어를 선호한다면 `$refresh` 대신 `$commit` 헬퍼를 사용할 수 있습니다. 둘은 동일합니다.

```blade
<button type="button" wire:click="$commit">...</button>
```

Livewire 컴포넌트 내에서 AlpineJS를 사용하여 컴포넌트 새로고침을 트리거할 수도 있습니다:

```blade
<button type="button" x-on:click="$wire.$refresh()">...</button>
```

자세한 내용은 [Livewire 내에서 Alpine 사용 문서](/docs/alpine)를 참조하세요.

## 액션 확인하기

사용자가 위험한 작업을 수행할 수 있도록 허용할 때 - 예를 들어 데이터베이스에서 게시물을 삭제하는 경우 - 해당 작업을 수행하려는 것인지 확인하는 알림을 표시하고 싶을 수 있습니다.

Livewire는 `wire:confirm`이라는 간단한 지시문을 제공하여 이를 쉽게 만들 수 있습니다:

```blade
<button
    type="button"
    wire:click="delete"
    wire:confirm="Are you sure you want to delete this post?"
>
    Delete post
</button>
```

`wire:confirm`이 Livewire 액션을 포함하는 요소에 추가되면, 사용자가 해당 액션을 트리거하려고 할 때 제공된 메시지가 포함된 확인 대화 상자가 표시됩니다. 사용자는 "OK"를 눌러 액션을 확인하거나 "Cancel"을 누르거나 Escape 키를 눌러 취소할 수 있습니다.

자세한 내용은 [`wire:confirm` 문서 페이지](/docs/wire-confirm)를 참조하세요.

## 이벤트 리스너

Livewire는 다양한 유형의 사용자 상호작용에 응답할 수 있는 다양한 이벤트 리스너를 지원합니다:

| 리스너 | 설명 |
|-----------------|------------------------------------------------------------------------------|
| `wire:click` | 요소가 클릭되었을 때 트리거됨 |
| `wire:submit` | 폼이 제출되었을 때 트리거됨 |
| `wire:keydown` | 키가 눌렸을 때 트리거됨 |
| `wire:keyup` | 키가 놓였을 때 트리거됨 |
| `wire:mouseenter` | 마우스가 요소에 들어갔을 때 트리거됨 |
| `wire:*` | `wire:` 다음에 오는 텍스트가 리스너의 이벤트 이름으로 사용됨 |

`wire:` 다음의 이벤트 이름은 무엇이든 될 수 있기 때문에, Livewire는 리스닝해야 할 모든 브라우저 이벤트를 지원합니다. 예를 들어, `transitionend`를 리스닝하려면 `wire:transitionend`를 사용할 수 있습니다.

### 특정 키 리스닝하기

Livewire의 편리한 별칭 중 하나를 사용하여 키 누름 이벤트 리스너를 특정 키나 키 조합으로 좁힐 수 있습니다.

예를 들어, 사용자가 검색 상자에 입력한 후 `Enter`를 눌렀을 때 검색을 수행하려면 `wire:keydown.enter`를 사용할 수 있습니다:

```blade
<input wire:model="query" wire:keydown.enter="searchPosts">
```

첫 번째 키 별칭 뒤에 더 많은 키 별칭을 체인으로 연결하여 키 조합을 리스닝할 수 있습니다. 예를 들어, `Shift` 키가 눌린 상태에서만 `Enter` 키를 리스닝하려면 다음과 같이 작성할 수 있습니다:

```blade
<input wire:keydown.shift.enter="...">
```

다음은 사용 가능한 모든 키 수정자 목록입니다:

| 수정자 | 키 |
|--------------|-------------------------------------|
| `.shift` | Shift |
| `.enter` | Enter |
| `.space` | Space |
| `.ctrl` | Ctrl |
| `.cmd` | Cmd |
| `.meta` | Mac에서는 Cmd, Windows에서는 Windows 키 |
| `.alt` | Alt |
| `.up` | 위쪽 화살표 |
| `.down` | 아래쪽 화살표 |
| `.left` | 왼쪽 화살표 |
| `.right` | 오른쪽 화살표 |
| `.escape` | Escape |
| `.tab` | Tab |
| `.caps-lock` | Caps Lock |
| `.equal` | Equal, `=` |
| `.period` | Period, `.` |
| `.slash` | Forward Slash, `/` |

### 이벤트 핸들러 수정자

Livewire는 또한 일반적인 이벤트 처리 작업을 간단하게 만들어주는 유용한 수정자를 포함하고 있습니다.

예를 들어, 이벤트 리스너 내에서 `event.preventDefault()`를 호출해야 하는 경우, 이벤트 이름 뒤에 `.prevent`를 추가할 수 있습니다:

```blade
<input wire:keydown.prevent="...">
```

다음은 사용 가능한 모든 이벤트 리스너 수정자와 그 기능의 전체 목록입니다:

| 수정자 | 키 |
|-------------------|--------------------------------------------------------------------------------|
| `.prevent` | `.preventDefault()` 호출과 동등 |
| `.stop` | `.stopPropagation()` 호출과 동등 |
| `.window` | `window` 객체에서 이벤트 리스닝 |
| `.outside` | 요소 "외부"의 클릭만 리스닝 |
| `.document` | `document` 객체에서 이벤트 리스닝 |
| `.once` | 리스너가 한 번만 호출되도록 보장 |
| `.debounce` | 기본적으로 250ms로 핸들러 디바운스 |
| `.debounce.100ms` | 특정 시간 동안 핸들러 디바운스 |
| `.throttle` | 최소 250ms마다 핸들러가 호출되도록 스로틀 |
| `.throttle.100ms` | 사용자 지정 기간으로 핸들러 스로틀 |
| `.self` | 이벤트가 이 요소에서 시작된 경우에만 리스너 호출, 자식에서는 호출하지 않음 |
| `.camel` | 이벤트 이름을 카멜 케이스로 변환 (`wire:custom-event` -> "customEvent") |
| `.dot` | 이벤트 이름을 점 표기법으로 변환 (`wire:custom-event` -> "custom.event") |
| `.passive` | `wire:touchstart.passive`는 스크롤 성능을 차단하지 않음 |
| `.capture` | "캡처링" 단계에서 이벤트 리스닝 |

`wire:`는 내부적으로 Alpine의 `x-on` 지시문을 사용하기 때문에, 이러한 수정자들은 Alpine에 의해 제공됩니다. 이러한 수정자를 언제 사용해야 하는지에 대한 자세한 내용은 [Alpine 이벤트 문서](https://alpinejs.dev/essentials/events)를 참조하세요.

### 써드파티 이벤트 처리하기

Livewire는 또한 써드파티 라이브러리에서 발생하는 사용자 정의 이벤트를 리스닝할 수 있습니다.

예를 들어, 프로젝트에서 [Trix](https://trix-editor.org/) 리치 텍스트 에디터를 사용하고 있고 `trix-change` 이벤트를 리스닝하여 에디터의 내용을 캡처하고 싶다고 가정해 봅시다. 이는 `wire:trix-change` 지시문을 사용하여 수행할 수 있습니다:

```blade
<form wire:submit="save">
    <!-- ... -->

    <trix-editor
        wire:trix-change="setPostContent($event.target.value)"
    ></trix-editor>

    <!-- ... -->
</form>
```

이 예제에서 `trix-change` 이벤트가 트리거될 때마다 `setPostContent` 액션이 호출되어 Livewire 컴포넌트의 `content` 속성을 Trix 에디터의 현재 값으로 업데이트합니다.

> 팁: 이벤트 핸들러 내에서 `$event`를 통해 이벤트 객체에 접근할 수 있습니다. 이는 이벤트에 대한 정보를 참조할 때 유용합니다. 예를 들어, `$event.target`을 통해 이벤트를 트리거한 요소에 접근할 수 있습니다.

> 주의: 위의 Trix 데모 코드는 불완전하며 이벤트 리스너의 시연용으로만 유용합니다. 그대로 사용하면 모든 키 입력마다 네트워크 요청이 발생할 것입니다. 더 성능이 좋은 구현은 다음과 같을 것입니다:
> 
> ```blade
> <trix-editor x-on:trix-change="$wire.content = $event.target.value"></trix-editor>
> ```

### 디스패치된 커스텀 이벤트 리스닝하기

Alpine에서 커스텀 이벤트를 디스패치하는 경우, Livewire를 사용하여 이러한 이벤트를 리스닝할 수도 있습니다:

```blade
<div wire:custom-event="...">
    <!-- 이 컴포넌트 내부 깊숙이 중첩된 곳: -->
    <button x-on:click="$dispatch('custom-event')">...</button>
</div>
```

위의 예제에서 버튼이 클릭되면, `custom-event` 이벤트가 디스패치되고 Livewire 컴포넌트의 루트까지 버블링되어 `wire:custom-event`가 이를 캐치하고 주어진 액션을 호출합니다.

애플리케이션의 다른 곳에서 디스패치된 이벤트를 리스닝하려면, 이벤트가 `window` 객체까지 버블링되기를 기다려야 하며 거기서 이를 리스닝해야 합니다. 다행히도 Livewire는 모든 이벤트 리스너에 간단한 `.window` 수정자를 추가하여 이를 쉽게 만들어줍니다:

```blade
<div wire:custom-event.window="...">
    <!-- ... -->
</div>

<!-- 컴포넌트 외부의 페이지 어딘가에서 디스패치됨: -->
<button x-on:click="$dispatch('custom-event')">...</button>
```

### 폼이 제출되는 동안 입력 비활성화하기

이전에 논의한 `CreatePost` 예제를 고려해봅시다:

```blade
<form wire:submit="save">
    <input wire:model="title">
    <textarea wire:model="content"></textarea>
    <button type="submit">Save</button>
</form>
```

사용자가 "Save"를 클릭하면, Livewire 컴포넌트의 `save()` 액션을 호출하는 네트워크 요청이 서버로 전송됩니다.

하지만 사용자가 느린 인터넷 연결에서 이 폼을 작성하고 있다고 가정해 봅시다. 사용자가 "Save"를 클릭하고 네트워크 요청이 평소보다 오래 걸리기 때문에 처음에는 아무 일도 일어나지 않습니다. 사용자는 제출이 실패했다고 생각하고 첫 번째 요청이 아직 처리되고 있는 동안 "Save" 버튼을 다시 클릭하려고 할 수 있습니다.

이 경우, 동시에 동일한 액션에 대한 두 개의 요청이 처리되고 있을 것입니다.

이러한 시나리오를 방지하기 위해 Livewire는 `wire:submit` 액션이 처리되는 동안 자동으로 제출 버튼과 `<form>` 요소 내의 모든 폼 입력을 비활성화합니다. 이를 통해 폼이 실수로 두 번 제출되는 것을 방지할 수 있습니다.

더 느린 연결에서 사용자의 혼란을 더욱 줄이기 위해, 미묘한 배경색 변경이나 SVG 애니메이션과 같은 로딩 표시기를 보여주는 것이 종종 도움이 됩니다.

Livewire는 페이지의 어디에서나 로딩 표시기를 쉽게 보여주고 숨길 수 있게 해주는 `wire:loading` 지시문을 제공합니다. 다음은 "Save" 버튼 아래에 로딩 메시지를 보여주기 위해 `wire:loading`을 사용하는 간단한 예제입니다:

```blade
<form wire:submit="save">
    <textarea wire:model="content"></textarea>
    <button type="submit">Save</button>
    <span wire:loading>Saving...</span>
</form>
```

`wire:loading`은 다양한 더 강력한 기능을 가진 강력한 기능입니다. [자세한 내용은 전체 로딩 문서를 확인하세요](/docs/wire-loading).

## 매개변수 전달하기

Livewire를 사용하면 Blade 템플릿에서 컴포넌트의 액션으로 매개변수를 전달할 수 있어, 액션이 호출될 때 프론트엔드에서 추가 데이터나 상태를 액션에 제공할 수 있습니다.

예를 들어, 사용자가 게시물을 삭제할 수 있는 `ShowPosts` 컴포넌트가 있다고 가정해 봅시다. 게시물의 ID를 Livewire 컴포넌트의 `delete()` 액션에 매개변수로 전달할 수 있습니다. 그런 다음 액션은 관련 게시물을 가져와 데이터베이스에서 삭제할 수 있습니다:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Post;

class ShowPosts extends Component
{
    public function delete($id)
    {
        $post = Post::findOrFail($id);

        $this->authorize('delete', $post);

        $post->delete();
    }

    public function render()
    {
        return view('livewire.show-posts', [
            'posts' => Auth::user()->posts,
        ]);
    }
}
```

```blade
<div>
    @foreach ($posts as $post)
        <div wire:key="{{ $post->id }}">
            <h1>{{ $post->title }}</h1>
            <span>{{ $post->content }}</span>

            <button wire:click="delete({{ $post->id }})">Delete</button>
        </div>
    @endforeach
</div>
```

ID가 2인 게시물의 경우, 위의 Blade 템플릿의 "Delete" 버튼은 브라우저에서 다음과 같이 렌더링됩니다:

```blade
<button wire:click="delete(2)">Delete</button>
```

이 버튼이 클릭되면, `delete()` 메서드가 호출되고 `$id`에 "2"라는 값이 전달됩니다.

> 주의: 액션 매개변수를 신뢰하지 마세요
> 
> 컨트롤러 요청 입력과 마찬가지로, 액션 매개변수는 임의의 사용자 입력이므로 이를 승인하는 것이 중요합니다. 데이터베이스에서 엔티티를 업데이트하기 전에 항상 해당 엔티티의 소유권을 승인해야 합니다.
> 
> 자세한 내용은 [보안 문제 및 모범 사례](/docs/actions#security-concerns)에 관한 문서를 참조하세요.

추가적인 편의를 위해, 액션에 매개변수로 제공된 해당 모델 ID에 의해 Eloquent 모델을 자동으로 해결할 수 있습니다. 이는 [라우트 모델 바인딩](/docs/components#using-route-model-binding)과 매우 유사합니다. 시작하려면 액션 매개변수에 모델 클래스를 타입 힌트로 지정하면 ID 대신 적절한 모델이 자동으로 데이터베이스에서 검색되어 액션에 전달됩니다:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Post;

class ShowPosts extends Component
{
    public function delete(Post $post)
    {
        $this->authorize('delete', $post);

        $post->delete();
    }

    public function render()
    {
        return view('livewire.show-posts', [
            'posts' => Auth::user()->posts,
        ]);
    }
}
```

## 의존성 주입

액션의 서명에 매개변수를 타입 힌트로 지정하여 [Laravel의 의존성 주입](https://laravel.com/docs/controllers#dependency-injection-and-controllers) 시스템을 활용할 수 있습니다. Livewire와 Laravel은 자동으로 컨테이너에서 액션의 의존성을 해결합니다:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Repositories\PostRepository;

class ShowPosts extends Component
{
    public function delete(PostRepository $posts, $postId)
    {
        $posts->deletePost($postId);
    }

    public function render()
    {
        return view('livewire.show-posts', [
            'posts' => Auth::user()->posts,
        ]);
    }
}
```

```blade
<div>
    @foreach ($posts as $post)
        <div wire:key="{{ $post->id }}">
            <h1>{{ $post->title }}</h1>
            <span>{{ $post->content }}</span>

            <button wire:click="delete({{ $post->id }})">Delete</button>
        </div>
    @endforeach
</div>
```

이 예제에서 `delete()` 메서드는 제공된 `$postId` 매개변수를 받기 전에 [Laravel의 서비스 컨테이너](https://laravel.com/docs/container#main-content)를 통해 해결된 `PostRepository`의 인스턴스를 받습니다.

## Alpine에서 액션 호출하기

Livewire는 [Alpine](https://alpinejs.dev/)과 원활하게 통합됩니다. 실제로 내부적으로 모든 Livewire 컴포넌트는 Alpine 컴포넌트이기도 합니다. 이는 컴포넌트 내에서 Alpine을 완전히 활용하여 JavaScript 기반의 클라이언트 측 상호작용을 추가할 수 있음을 의미합니다.

이 조합을 더욱 강력하게 만들기 위해, Livewire는 Alpine에 PHP 컴포넌트의 JavaScript 표현으로 취급될 수 있는 매직 `$wire` 객체를 노출합니다. [`$wire`를 통해 공개 속성에 접근하고 변경하는 것](/docs/properties#accessing-properties-from-javascript) 외에도, 액션을 호출할 수 있습니다. `$wire` 객체에서 액션이 호출되면, 백엔드 Livewire 컴포넌트에서 해당하는 PHP 메서드가 호출됩니다:

```blade
<button x-on:click="$wire.save()">Save Post</button>
```

또는 더 복잡한 예를 들어, Alpine의 [`x-intersect`](https://alpinejs.dev/plugins/intersect) 유틸리티를 사용하여 주어진 요소가 페이지에 보일 때 Livewire `incrementViewCount()` 액션을 트리거할 수 있습니다:

```blade
<div x-intersect="$wire.incrementViewCount()">...</div>
```

### 매개변수 전달하기

`$wire` 메서드에 전달하는 모든 매개변수도 PHP 클래스 메서드로 전달됩니다. 예를 들어, 다음과 같은 Livewire 액션을 고려해보세요:

```php
public function addTodo($todo)
{
    $this->todos[] = $todo;
}
```

컴포넌트의 Blade 템플릿 내에서 Alpine을 통해 이 액션을 호출할 수 있으며, 액션에 제공해야 할 매개변수를 제공할 수 있습니다:

```blade
<div x-data="{ todo: '' }">
    <input type="text" x-model="todo">

    <button x-on:click="$wire.addTodo(todo)">Add Todo</button>
</div>
```

사용자가 텍스트 입력에 "Take out the trash"를 입력하고 "Add Todo" 버튼을 눌렀다면, `addTodo()` 메서드가 트리거되고 `$todo` 매개변수 값은 "Take out the trash"가 될 것입니다.

### 반환 값 받기

더 강력한 기능을 위해, 호출된 `$wire` 액션은 네트워크 요청이 처리되는 동안 프로미스를 반환합니다. 서버 응답이 수신되면, 프로미스는 백엔드 액션이 반환한 값으로 해결됩니다.

예를 들어, 다음과 같은 액션이 있는 Livewire 컴포넌트를 고려해보세요:

```php
use App\Models\Post;

public function getPostCount()
{
    return Post::count();
}
```

`$wire`를 사용하여 액션을 호출하고 반환된 값을 해결할 수 있습니다:

```blade
<span x-init="$el.innerHTML = await $wire.getPostCount()"></span>
```

이 예제에서 `getPostCount()` 메서드가 "10"을 반환한다면, `<span>` 태그에도 "10"이 포함될 것입니다.

Livewire를 사용할 때 Alpine 지식이 필요한 것은 아닙니다. 그러나 Alpine은 매우 강력한 도구이며, Alpine을 알면 Livewire 경험과 생산성이 향상될 것입니다.

## Livewire의 "하이브리드" JavaScript 함수

때로는 서버와 통신할 필요가 없고 JavaScript만으로 더 효율적으로 작성할 수 있는 컴포넌트의 액션이 있을 수 있습니다.

이러한 경우, Blade 템플릿이나 다른 파일 내부에 액션을 작성하는 대신, 컴포넌트 액션이 JavaScript 함수를 문자열로 반환할 수 있습니다. 액션에 `#[Js]` 속성이 표시되어 있으면, 애플리케이션의 프론트엔드에서 호출할 수 있습니다:

예를 들어:

```php
<?php

namespace App\Livewire;

use Livewire\Attributes\Js;
use Livewire\Component;
use App\Models\Post;

class SearchPosts extends Component
{
    public $query = '';

    #[Js]
    public function resetQuery()
    {
        return <<<'JS'
            $wire.query = '';
        JS;
    }

    public function render()
    {
        return view('livewire.search-posts', [
            'posts' => Post::whereTitle($this->query)->get(),
        ]);
    }
}
```

```blade
<div>
    <input wire:model.live="query">

    <button wire:click="resetQuery">Reset Search</button>

    @foreach ($posts as $post)
        <!-- ... -->
    @endforeach
</div>
```

위의 예제에서, "Reset Search" 버튼을 누르면 서버에 어떤 요청도 보내지 않고 텍스트 입력이 지워집니다.

### 일회성 JavaScript 표현식 평가하기

전체 메서드를 JavaScript에서 평가하도록 지정하는 것 외에도, `js()` 메서드를 사용하여 더 작은 개별 표현식을 평가할 수 있습니다.

이는 일반적으로 서버 측 액션이 수행된 후 클라이언트 측 후속 작업을 수행하는 데 유용합니다.

예를 들어, 다음은 게시물이 데이터베이스에 저장된 후 클라이언트 측 알림 대화 상자를 트리거하는 `CreatePost` 컴포넌트의 예입니다:

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class CreatePost extends Component
{
    public $title = '';

    public function save()
    {
        // ...

        $this->js("alert('Post saved!')");
    }
}
```

JavaScript 표현식 `alert('Post saved!')`는 이제 서버에서 게시물이 데이터베이스에 저장된 후 클라이언트에서 실행됩니다.

`#[Js]` 메서드와 마찬가지로, 표현식 내에서 현재 컴포넌트의 `$wire` 객체에 접근할 수 있습니다.

## 매직 액션

Livewire는 사용자 정의 메서드를 정의하지 않고도 컴포넌트에서 일반적인 작업을 수행할 수 있는 "매직" 액션 세트를 제공합니다. 이러한 매직 액션은 Blade 템플릿에 정의된 이벤트 리스너 내에서 사용할 수 있습니다.

### `$parent`

`$parent` 매직 변수를 사용하면 자식 컴포넌트에서 부모 컴포넌트의 속성에 접근하고 부모 컴포넌트의 액션을 호출할 수 있습니다:

```blade
<button wire:click="$parent.removePost({{ $post->id }})">Remove</button>
```

위의 예제에서, 부모 컴포넌트에 `removePost()` 액션이 있다면, 자식은 `$parent.removePost()`를 사용하여 Blade 템플릿에서 직접 이를 호출할 수 있습니다.

### `$set`

`$set` 매직 액션을 사용하면 Blade 템플릿에서 직접 Livewire 컴포넌트의 속성을 업데이트할 수 있습니다. `$set`을 사용하려면 업데이트하려는 속성과 새 값을 인수로 제공하세요:

```blade
<button wire:click="$set('query', '')">Reset Search</button>
```

이 예제에서, 버튼을 클릭하면 컴포넌트의 `$query` 속성을 `''`로 설정하는 네트워크 요청이 디스패치됩니다.

### `$refresh`

`$refresh` 액션은 Livewire 컴포넌트의 재렌더링을 트리거합니다. 이는 어떤 속성 값도 변경하지 않고 컴포넌트의 뷰를 업데이트할 때 유용할 수 있습니다:

```blade
<button wire:click="$refresh">Refresh</button>
```

버튼을 클릭하면 컴포넌트가 재렌더링되어 뷰의 최신 변경 사항을 볼 수 있습니다.

### `$toggle`

`$toggle` 액션은 Livewire 컴포넌트의 불리언 속성 값을 토글하는 데 사용됩니다:

```blade
<button wire:click="$toggle('sortAsc')">
    Sort {{ $sortAsc ? 'Descending' : 'Ascending' }}
</button>
```

이 예제에서, 버튼을 클릭하면 컴포넌트의 `$sortAsc` 속성이 `true`와 `false` 사이에서 토글됩니다.

### `$dispatch`

`$dispatch` 액션을 사용하면 브라우저에서 직접 Livewire 이벤트를 디스패치할 수 있습니다. 다음은 클릭 시 `post-deleted` 이벤트를 디스패치하는 버튼의 예입니다:

```blade
<button type="submit" wire:click="$dispatch('post-deleted')">Delete Post</button>
```

### `$event`

`$event` 액션은 `wire:click`과 같은 이벤트 리스너 내에서 사용할 수 있습니다. 이 액션을 사용하면 트리거된 실제 JavaScript 이벤트에 접근할 수 있어 트리거한 요소와 기타 관련 정보를 참조할 수 있습니다:

```blade
<input type="text" wire:keydown.enter="search($event.target.value)">
```

위의 입력에서 사용자가 입력하는 동안 엔터 키를 누르면, 입력의 내용이 `search()` 액션의 매개변수로 전달됩니다.

### Alpine에서 매직 액션 사용하기

`$wire` 객체를 사용하여 Alpine에서 매직 액션을 호출할 수도 있습니다. 예를 들어, `$wire` 객체를 사용하여 `$refresh` 매직 액션을 호출할 수 있습니다:

```blade
<button x-on:click="$wire.$refresh()">Refresh</button>
```

## 재렌더링 건너뛰기

때로는 액션이 호출될 때 렌더링된 Blade 템플릿을 변경하는 부작용이 없는 컴포넌트의 액션이 있을 수 있습니다. 이런 경우, 액션 메서드 위에 `#[Renderless]` 속성을 추가하여 Livewire 라이프사이클의 `render` 부분을 건너뛸 수 있습니다.

예를 들어, 아래의 `ShowPost` 컴포넌트에서는 사용자가 게시물의 하단으로 스크롤했을 때 "조회수"가 기록됩니다:

```php
<?php

namespace App\Livewire;

use Livewire\Attributes\Renderless;
use Livewire\Component;
use App\Models\Post;

class ShowPost extends Component
{
    public Post $post;

    public function mount(Post $post)
    {
        $this->post = $post;
    }

    #[Renderless]
    public function incrementViewCount()
    {
        $this->post->incrementViewCount();
    }

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

    <div x-intersect="$wire.incrementViewCount()"></div>
</div>
```

위의 예제는 [`x-intersect`](https://alpinejs.dev/plugins/intersect)를 사용합니다. 이는 요소가 뷰포트에 들어올 때 표현식을 호출하는 Alpine 유틸리티입니다(일반적으로 사용자가 페이지 아래쪽의 요소로 스크롤할 때 감지하는 데 사용됩니다).

보시다시피, 사용자가 게시물 하단으로 스크롤하면 `incrementViewCount()`가 호출됩니다. `#[Renderless]`가 액션에 추가되었기 때문에, 조회수는 기록되지만 템플릿은 재렌더링되지 않으며 페이지의 어떤 부분도 영향을 받지 않습니다.

메서드 속성을 사용하지 않거나 조건부로 렌더링을 건너뛰어야 하는 경우, 컴포넌트 액션에서 `skipRender()` 메서드를 호출할 수 있습니다:

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class ShowPost extends Component
{
    public Post $post;

    public function mount(Post $post)
    {
        $this->post = $post;
    }

    public function incrementViewCount()
    {
        $this->post->incrementViewCount();

        $this->skipRender();
    }

    public function render()
    {
        return view('livewire.show-post');
    }
}
```

## 보안 고려사항

Livewire 컴포넌트의 모든 공개 메서드는 클라이언트 측에서 호출할 수 있으며, 이를 호출하는 `wire:click` 핸들러가 연결되어 있지 않더라도 마찬가지입니다. 이러한 시나리오에서 사용자는 여전히 브라우저의 개발자 도구에서 액션을 트리거할 수 있습니다.

다음은 Livewire 컴포넌트에서 쉽게 놓칠 수 있는 취약점의 세 가지 예입니다. 각 예제는 먼저 취약한 컴포넌트를 보여주고 그 다음에 안전한 컴포넌트를 보여줍니다. 연습 삼아 첫 번째 예제에서 취약점을 발견하기 전에 해결책을 보지 않고 시도해 보세요.

취약점을 발견하는 데 어려움을 겪고 있고 이로 인해 자신의 애플리케이션을 안전하게 유지하는 능력에 대해 걱정된다면, 이러한 모든 취약점이 요청과 컨트롤러를 사용하는 표준 웹 애플리케이션에도 적용된다는 점을 기억하세요. 컴포넌트 메서드를 컨트롤러 메서드의 프록시로, 그리고 그 매개변수를 요청 입력의 프록시로 사용한다면, 기존의 애플리케이션 보안 지식을 Livewire 코드에 적용할 수 있을 것입니다.

### 항상 액션 매개변수를 인증하세요

컨트롤러 요청 입력과 마찬가지로, 액션 매개변수는 임의의 사용자 입력이므로 이를 인증하는 것이 중요합니다.

아래는 사용자가 한 페이지에서 모든 게시물을 볼 수 있는 `ShowPosts` 컴포넌트입니다. 사용자는 각 게시물의 "Delete" 버튼을 사용하여 원하는 게시물을 삭제할 수 있습니다.

다음은 취약한 버전의 컴포넌트입니다:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Post;

class ShowPosts extends Component
{
    public function delete($id)
    {
        $post = Post::find($id);

        $post->delete();
    }

    public function render()
    {
        return view('livewire.show-posts', [
            'posts' => Auth::user()->posts,
        ]);
    }
}
```

```blade
<div>
    @foreach ($posts as $post)
        <div wire:key="{{ $post->id }}">
            <h1>{{ $post->title }}</h1>
            <span>{{ $post->content }}</span>

            <button wire:click="delete({{ $post->id }})">Delete</button>
        </div>
    @endforeach
</div>
```

악의적인 사용자는 JavaScript 콘솔에서 직접 `delete()`를 호출하여 액션에 원하는 매개변수를 전달할 수 있다는 점을 기억하세요. 이는 자신의 게시물을 보고 있는 사용자가 소유하지 않은 게시물의 ID를 `delete()`에 전달하여 다른 사용자의 게시물을 삭제할 수 있음을 의미합니다.

이를 방지하기 위해, 삭제하려는 게시물을 사용자가 소유하고 있는지 인증해야 합니다:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Post;

class ShowPosts extends Component
{
    public function delete($id)
    {
        $post = Post::find($id);

        $this->authorize('delete', $post);
        
        $post->delete();
    }

    public function render()
    {
        return view('livewire.show-posts', [
            'posts' => Auth::user()->posts,
        ]);
    }
}
```

### 항상 서버 측에서 인증하세요

표준 Laravel 컨트롤러와 마찬가지로, UI에 액션을 호출하는 방법이 없더라도 모든 사용자가 Livewire 액션을 호출할 수 있습니다.

다음의 `BrowsePosts` 컴포넌트를 고려해보세요. 여기서는 모든 사용자가 애플리케이션의 모든 게시물을 볼 수 있지만, 관리자만 게시물을 삭제할 수 있습니다:

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Post;

class BrowsePosts extends Component
{
    public function deletePost($id)
    {
        $post = Post::find($id);

        $post->delete();
    }

    public function render()
    {
        return view('livewire.browse-posts', [
            'posts' => Post::all(),
        ]);
    }
}
```

```blade
<div>
    @foreach ($posts as $post)
        <div wire:key="{{ $post->id }}">
            <h1>{{ $post->title }}</h1>
            <span>{{ $post->content }}</span>

            @if (Auth::user()->isAdmin())
                <button wire:click="deletePost({{ $post->id }})">Delete</button>
            @endif
        </div>
    @endforeach
</div>
```

보시다시피, 관리자만 "Delete" 버튼을 볼 수 있습니다. 그러나 모든 사용자는 브라우저의 개발자 도구에서 `deletePost()`를 호출할 수 있습니다.

이 취약점을 해결하기 위해, 서버에서 액션을 인증해야 합니다:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Post;

class BrowsePosts extends Component
{
    public function deletePost($id)
    {
        if (! Auth::user()->isAdmin) {
            abort(403);
        }

        $post = Post::find($id);

        $post->delete();
    }

    public function render()
    {
        return view('livewire.browse-posts', [
            'posts' => Post::all(),
        ]);
    }
}
```

이 변경으로, 이제 관리자만 이 컴포넌트에서 게시물을 삭제할 수 있습니다.

### 위험한 메서드를 protected 또는 private로 유지하세요

Livewire 컴포넌트 내의 모든 public 메서드는 클라이언트에서 호출할 수 있습니다. `wire:click` 핸들러 내에서 참조하지 않은 메서드도 마찬가지입니다. 클라이언트 측에서 호출하도록 의도되지 않은 메서드를 사용자가 호출하는 것을 방지하려면, 해당 메서드를 `protected` 또는 `private`로 표시해야 합니다. 이렇게 하면 그 민감한 메서드의 가시성을 컴포넌트의 클래스와 그 하위 클래스로 제한하여 클라이언트 측에서 호출할 수 없도록 합니다.

이전에 논의한 `BrowsePosts` 예제를 고려해보세요. 여기서 사용자는 애플리케이션의 모든 게시물을 볼 수 있지만 관리자만 게시물을 삭제할 수 있습니다. [항상 서버 측에서 인증하세요](/docs/actions#always-authorize-server-side) 섹션에서 서버 측 인증을 추가하여 액션을 안전하게 만들었습니다. 이제 코드를 단순화하기 위해 실제 게시물 삭제를 전용 메서드로 리팩토링했다고 상상해보세요:

```php
// 경고: 이 스니펫은 하지 말아야 할 것을 보여줍니다...
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Post;

class BrowsePosts extends Component
{
    public function deletePost($id)
    {
        if (! Auth::user()->isAdmin) {
            abort(403);
        }

        $this->delete($id);
    }

    public function delete($postId) 
    {
        $post = Post::find($postId);

        $post->delete();
    }

    public function render()
    {
        return view('livewire.browse-posts', [
            'posts' => Post::all(),
        ]);
    }
}
```

```blade
<div>
    @foreach ($posts as $post)
        <div wire:key="{{ $post->id }}">
            <h1>{{ $post->title }}</h1>
            <span>{{ $post->content }}</span>

            <button wire:click="deletePost({{ $post->id }})">Delete</button>
        </div>
    @endforeach
</div>
```

보시다시피, 게시물 삭제 로직을 `delete()`라는 전용 메서드로 리팩토링했습니다. 이 메서드가 템플릿 어디에서도 참조되지 않더라도, 사용자가 그 존재를 알게 되면 `public`이기 때문에 브라우저의 개발자 도구에서 호출할 수 있습니다.

이를 해결하기 위해 메서드를 `protected` 또는 `private`로 표시할 수 있습니다. 메서드가 `protected` 또는 `private`로 표시되면, 사용자가 이를 호출하려고 할 때 오류가 발생합니다:

```php
<?php
 
namespace App\Livewire;
 
use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Post;
 
class BrowsePosts extends Component
{
    public function deletePost($id)
    {
        if (! Auth::user()->isAdmin) {
            abort(403);
        }
 
        $this->delete($id);
    }
 
    protected function delete($postId) 
    {
        $post = Post::find($postId);
 
        $post->delete();
    }
 
    public function render()
    {
        return view('livewire.browse-posts', [
            'posts' => Post::all(),
        ]);
    }
}
```
