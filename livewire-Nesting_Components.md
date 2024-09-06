# 컴포넌트 중첩

Livewire를 사용하면 부모 컴포넌트 내에 추가 Livewire 컴포넌트를 중첩할 수 있습니다. 이 기능은 매우 강력하며, 애플리케이션 전체에서 공유되는 Livewire 컴포넌트 내의 동작을 재사용하고 캡슐화할 수 있게 해줍니다.

> 주의: Livewire 컴포넌트가 필요하지 않을 수 있습니다
>
> 템플릿의 일부를 중첩된 Livewire 컴포넌트로 추출하기 전에 스스로에게 물어보세요: 이 컴포넌트의 내용이 "라이브"여야 하나요? 그렇지 않다면, 간단한 Blade 컴포넌트를 대신 만드는 것을 추천합니다. Livewire의 동적 특성이나 직접적인 성능 이점이 있을 때만 Livewire 컴포넌트를 만드세요.

Livewire 컴포넌트 중첩의 성능, 사용 영향 및 제약 사항에 대한 자세한 정보는 [중첩에 대한 심층 기술 검토](/docs/understanding-nesting) 문서를 참조하세요.

## 컴포넌트 중첩하기

부모 컴포넌트 내에 Livewire 컴포넌트를 중첩하려면 부모 컴포넌트의 Blade 뷰에 단순히 포함시키면 됩니다. 아래는 중첩된 `TodoList` 컴포넌트를 포함하는 `Dashboard` 부모 컴포넌트의 예시입니다:

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class Dashboard extends Component
{
    public function render()
    {
        return view('livewire.dashboard');
    }
}
```

```blade
<div>
    <h1>Dashboard</h1>

    <livewire:todo-list />
</div>
```

이 페이지의 초기 렌더링에서 `Dashboard` 컴포넌트는 `<livewire:todo-list />`를 만나면 그 자리에 렌더링합니다. `Dashboard`에 대한 후속 네트워크 요청에서는 중첩된 `todo-list` 컴포넌트가 이제 페이지에서 독립적인 컴포넌트이기 때문에 렌더링을 건너뜁니다. 중첩 및 렌더링 뒤의 기술적 개념에 대한 자세한 정보는 [중첩된 컴포넌트가 "아일랜드"인 이유](/docs/understanding-nesting#every-component-is-an-island)에 대한 문서를 참조하세요.

컴포넌트 렌더링 구문에 대한 자세한 정보는 [컴포넌트 렌더링](/docs/components#rendering-components) 문서를 참조하세요.

## 자식 컴포넌트에 props 전달하기

부모 컴포넌트에서 자식 컴포넌트로 데이터를 전달하는 것은 간단합니다. 사실, 일반적인 [Blade 컴포넌트](https://laravel.com/docs/blade#components)에 props를 전달하는 것과 매우 유사합니다.

예를 들어, `$todos` 컬렉션을 `TodoCount`라는 자식 컴포넌트에 전달하는 `TodoList` 컴포넌트를 살펴보겠습니다:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;

class TodoList extends Component
{
    public function render()
    {
        return view('livewire.todo-list', [
            'todos' => Auth::user()->todos,
        ]);
    }
}
```

```blade
<div>
    <livewire:todo-count :todos="$todos" />

    <!-- ... -->
</div>
```

보시다시피, `:todos="$todos"` 구문을 사용하여 `$todos`를 `todo-count`에 전달하고 있습니다.

이제 `$todos`가 자식 컴포넌트로 전달되었으므로, 자식 컴포넌트의 `mount()` 메서드를 통해 해당 데이터를 받을 수 있습니다:

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Todo;

class TodoCount extends Component
{
    public $todos;

    public function mount($todos)
    {
        $this->todos = $todos;
    }

    public function render()
    {
        return view('livewire.todo-count', [
            'count' => $this->todos->count(),
        ]);
    }
}
```

> 팁: `mount()`를 생략하여 더 짧게 작성하기
>
> 위의 예시에서 `mount()` 메서드가 중복된 보일러플레이트 코드로 느껴진다면, 속성 이름과 매개변수 이름이 일치하는 한 이를 생략할 수 있습니다:
>
> ```php
> public $todos;
> ```

### 정적 props 전달하기

이전 예시에서는 Livewire의 동적 prop 구문을 사용하여 자식 컴포넌트에 props를 전달했습니다. 이 구문은 다음과 같은 PHP 표현식을 지원합니다:

```blade
<livewire:todo-count :todos="$todos" />
```

그러나 때로는 문자열과 같은 단순한 정적 값을 컴포넌트에 전달하고 싶을 수 있습니다. 이런 경우에는 문장 시작 부분의 콜론을 생략할 수 있습니다:

```blade
<livewire:todo-count :todos="$todos" label="Todo Count:" />
```

불리언 값은 키만 지정하여 컴포넌트에 제공할 수 있습니다. 예를 들어, `$inline` 변수의 값을 `true`로 컴포넌트에 전달하려면 다음과 같이 컴포넌트 태그에 `inline`만 배치하면 됩니다:

```blade
<livewire:todo-count :todos="$todos" inline />
```

### 축약된 속성 구문

PHP 변수를 컴포넌트에 전달할 때, 변수 이름과 prop 이름이 종종 동일합니다. 이름을 두 번 작성하는 것을 피하기 위해 Livewire는 변수 앞에 콜론만 붙이는 것을 허용합니다:

```blade
<livewire:todo-count :$todos />
```

## 반복문에서 자식 컴포넌트 렌더링하기

반복문 내에서 자식 컴포넌트를 렌더링할 때는 각 반복에 대해 고유한 `key` 값을 포함해야 합니다.

컴포넌트 키는 Livewire가 후속 렌더링에서 각 컴포넌트를 추적하는 방법입니다. 특히 컴포넌트가 이미 렌더링되었거나 페이지에서 여러 컴포넌트가 재배열된 경우에 중요합니다.

자식 컴포넌트에 `:key` prop을 지정하여 컴포넌트의 키를 지정할 수 있습니다:

```blade
<div>
    <h1>Todos</h1>

    @foreach ($todos as $todo)
        <livewire:todo-item :$todo :key="$todo->id" />
    @endforeach
</div>
```

보시다시피, 각 자식 컴포넌트는 각 `$todo`의 ID로 설정된 고유한 키를 가집니다. 이는 todos가 재정렬되더라도 키가 고유하고 추적될 수 있도록 합니다.

> 경고: 키는 선택사항이 아닙니다
>
> Vue나 Alpine과 같은 프론트엔드 프레임워크를 사용해 본 경험이 있다면, 반복문에서 중첩된 요소에 키를 추가하는 것에 익숙할 것입니다. 하지만 이러한 프레임워크에서는 키가 *필수*가 아니어서, 키 없이도 항목들이 렌더링되지만 재정렬이 제대로 추적되지 않을 수 있습니다. 그러나 Livewire는 키에 더 크게 의존하며 키 없이는 제대로 작동하지 않습니다.

## 반응형 props

Livewire를 처음 접하는 개발자들은 props가 기본적으로 "반응형"일 것이라고 예상합니다. 다시 말해, 부모가 자식 컴포넌트로 전달되는 prop의 값을 변경할 때 자식 컴포넌트가 자동으로 업데이트될 것이라고 기대합니다. 하지만 기본적으로 Livewire props는 반응형이 아닙니다.

Livewire를 사용할 때, [모든 컴포넌트는 아일랜드입니다](/docs/understanding-nesting#every-component-is-an-island). 이는 부모에서 업데이트가 트리거되고 네트워크 요청이 발송될 때, 자식 컴포넌트의 상태가 아닌 부모 컴포넌트의 상태만 서버로 전송되어 다시 렌더링된다는 의미입니다. 이 동작의 의도는 서버와 클라이언트 간에 최소한의 데이터만 주고받아 업데이트를 최대한 성능적으로 만드는 것입니다.

하지만 prop을 반응형으로 만들고 싶거나 필요하다면, `#[Reactive]` 속성 매개변수를 사용하여 이 동작을 쉽게 활성화할 수 있습니다.

예를 들어, 아래는 부모 `TodoList` 컴포넌트의 템플릿입니다. 내부에서 `TodoCount` 컴포넌트를 렌더링하고 현재 todo 리스트를 전달하고 있습니다:

```blade
<div>
    <h1>Todos:</h1>

    <livewire:todo-count :$todos />

    <!-- ... -->
</div>
```

이제 `TodoCount` 컴포넌트의 `$todos` prop에 `#[Reactive]`를 추가해 보겠습니다. 이렇게 하면 부모 컴포넌트 내에서 todos가 추가되거나 제거될 때 `TodoCount` 컴포넌트 내에서 자동으로 업데이트가 트리거됩니다:

```php
<?php

namespace App\Livewire;

use Livewire\Attributes\Reactive;
use Livewire\Component;
use App\Models\Todo;

class TodoCount extends Component
{
    #[Reactive]
    public $todos;

    public function render()
    {
        return view('livewire.todo-count', [
            'count' => $this->todos->count(),
        ]);
    }
}
```

반응형 속성은 매우 강력한 기능으로, Livewire를 Vue나 React와 같은 프론트엔드 컴포넌트 라이브러리와 더 유사하게 만듭니다. 하지만 이 기능의 성능 영향을 이해하고 특정 시나리오에 적합할 때만 `#[Reactive]`를 추가하는 것이 중요합니다.

## `wire:model`을 사용하여 자식 데이터에 바인딩하기

부모와 자식 컴포넌트 간에 상태를 공유하는 또 다른 강력한 패턴은 Livewire의 `Modelable` 기능을 통해 자식 컴포넌트에 직접 `wire:model`을 사용하는 것입니다.

이 동작은 입력 요소를 전용 Livewire 컴포넌트로 추출하면서도 부모 컴포넌트에서 그 상태에 접근해야 할 때 매우 자주 필요합니다.

아래는 사용자가 추가하려는 현재 todo를 추적하는 `$todo` 속성을 포함하는 부모 `TodoList` 컴포넌트의 예시입니다:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Todo;

class TodoList extends Component
{
    public $todo = '';

    public function add()
    {
        Todo::create([
            'content' => $this->pull('todo'),
        ]);
    }

    public function render()
    {
        return view('livewire.todo-list', [
            'todos' => Auth::user()->todos,
        ]);
    }
}
```

`TodoList` 템플릿에서 볼 수 있듯이, `wire:model`을 사용하여 `$todo` 속성을 중첩된 `TodoInput` 컴포넌트에 직접 바인딩하고 있습니다:

```blade
<div>
    <h1>Todos</h1>

    <livewire:todo-input wire:model="todo" />

    <button wire:click="add">Add Todo</button>

    <div>
        @foreach ($todos as $todo)
            <livewire:todo-item :$todo :key="$todo->id" />
        @endforeach
    </div>
</div>
```

Livewire는 `#[Modelable]` 속성을 제공하며, 이를 자식 컴포넌트의 속성에 추가하여 부모 컴포넌트에서 `wire:model`로 바인딩할 수 있게 만들 수 있습니다.

아래는 `#[Modelable]` 속성이 `$value` 속성 위에 추가된 `TodoInput` 컴포넌트입니다. 이는 Livewire에게 부모가 이 컴포넌트에 `wire:model`을 선언하면 이 속성에 바인딩해야 한다고 알려줍니다:

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use Livewire\Attributes\Modelable;

class TodoInput extends Component
{
    #[Modelable]
    public $value = '';

    public function render()
    {
        return view('livewire.todo-input');
    }
}
```

```blade
<div>
    <input type="text" wire:model="value">
</div>
```

이제 부모 `TodoList` 컴포넌트는 `TodoInput`을 다른 입력 요소처럼 취급하고 `wire:model`을 사용하여 그 값에 직접 바인딩할 수 있습니다.

> 주의: 현재 Livewire는 단일 `#[Modelable]` 속성만 지원하므로 첫 번째 것만 바인딩됩니다.

## 자식으로부터 이벤트 수신하기

부모-자식 컴포넌트 간 통신의 또 다른 강력한 기술은 Livewire의 이벤트 시스템입니다. 이를 통해 서버나 클라이언트에서 이벤트를 발송하고 다른 컴포넌트에서 이를 가로챌 수 있습니다.

[Livewire의 이벤트 시스템에 대한 전체 문서](/docs/events)에서 이벤트에 대한 더 자세한 정보를 제공하지만, 여기서는 이벤트를 사용하여 부모 컴포넌트에서 업데이트를 트리거하는 간단한 예시를 살펴보겠습니다.

todo를 보여주고 제거하는 기능이 있는 `TodoList` 컴포넌트를 고려해봅시다:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Todo;

class TodoList extends Component
{
    public function remove($todoId)
    {
        $todo = Todo::find($todoId);

        $this->authorize('delete', $todo);

        $todo->delete();
    }

    public function render()
    {
        return view('livewire.todo-list', [
            'todos' => Auth::user()->todos,
        ]);
    }
}
```

```blade
<div>
    @foreach ($todos as $todo)
        <livewire:todo-item :$todo :key="$todo->id" />
    @endforeach
</div>
```

자식 `TodoItem` 컴포넌트 내부에서 `remove()`를 호출하려면, `#[On]` 속성을 통해 `TodoList`에 이벤트 리스너를 추가할 수 있습니다:

```php
<?php

namespace App\Livewire;

use Illuminate\Support\Facades\Auth;
use Livewire\Component;
use App\Models\Todo;
use Livewire\Attributes\On;

class TodoList extends Component
{
    #[On('remove-todo')]
    public function remove($todoId)
    {
        $todo = Todo::find($todoId);

        $this->authorize('delete', $todo);

        $todo->delete();
    }

    public function render()
    {
        return view('livewire.todo-list', [
            'todos' => Auth::user()->todos,
        ]);
    }
}
```

액션에 속성을 추가했다면, `TodoList` 자식 컴포넌트에서 `remove-todo` 이벤트를 발송할 수 있습니다:

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Todo;

class TodoItem extends Component
{
    public Todo $todo;

    public function remove()
    {
        $this->dispatch('remove-todo', todoId: $this->todo->id);
    }

    public function render()
    {
        return view('livewire.todo-item');
    }
}
```

```blade
<div>
    <span>{{ $todo->content }}</span>

    <button wire:click="remove">Remove</button>
</div>
```

이제 `TodoItem` 내부의 "Remove" 버튼이 클릭되면, 부모 `TodoList` 컴포넌트가 발송된 이벤트를 가로채고 todo 제거를 수행합니다.

todo가 부모에서 제거된 후, 리스트가 다시 렌더링되고 `remove-todo` 이벤트를 발송한 자식은 페이지에서 제거됩니다.

### 클라이언트 측에서 발송하여 성능 향상하기

위의 예시는 작동하지만, 단일 액션을 수행하는 데 두 번의 네트워크 요청이 필요합니다:

1. `TodoItem` 컴포넌트에서의 첫 번째 네트워크 요청은 `remove` 액션을 트리거하고, `remove-todo` 이벤트를 발송합니다.
2. 두 번째 네트워크 요청은 `remove-todo` 이벤트가 클라이언트 측에서 발송되고 `TodoList`에 의해 가로채져 `remove` 액션을 호출한 후에 발생합니다.

클라이언트 측에서 직접 `remove-todo` 이벤트를 발송하여 첫 번째 요청을 완전히 피할 수 있습니다. 아래는 `remove-todo` 이벤트를 발송할 때 네트워크 요청을 트리거하지 않는 업데이트된 `TodoItem` 컴포넌트입니다:

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Todo;

class TodoItem extends Component
{
    public Todo $todo;

    public function render()
    {
        return view('livewire.todo-item');
    }
}
```

```blade
<div>
    <span>{{ $todo->content }}</span>

    <button wire:click="$dispatch('remove-todo', { todoId: {{ $todo->id }} })">Remove</button>
</div>
```

경험상, 가능한 경우 항상 클라이언트 측에서 발송하는 것이 좋습니다.

## 자식에서 부모에 직접 접근하기

이벤트 통신은 간접적인 계층을 추가합니다. 부모는 자식에서 절대 발송되지 않는 이벤트를 수신할 수 있고, 자식은 부모가 절대 가로채지 않는 이벤트를 발송할 수 있습니다.

이러한 간접성이 때로는 바람직할 수 있지만, 다른 경우에는 자식 컴포넌트에서 부모 컴포넌트에 직접 접근하는 것이 더 선호될 수 있습니다.

Livewire는 이를 위해 Blade 템플릿에 매직 `$parent` 변수를 제공합니다. 이를 사용하여 자식에서 직접 부모의 액션과 속성에 접근할 수 있습니다. 다음은 매직 `$parent` 변수를 통해 부모의 `remove()` 액션을 직접 호출하도록 다시 작성한 위의 `TodoItem` 템플릿입니다:

```blade
<div>
    <span>{{ $todo->content }}</span>

    <button wire:click="$parent.remove({{ $todo->id }})">Remove</button>
</div>
```

이벤트와 직접적인 부모 통신은 부모와 자식 컴포넌트 간에 통신하는 몇 가지 방법입니다. 이들의 트레이드오프를 이해하면 특정 시나리오에서 어떤 패턴을 사용할지에 대해 더 정보에 기반한 결정을 내릴 수 있습니다.

## 동적 자식 컴포넌트

때로는 어떤 자식 컴포넌트가 페이지에 렌더링되어야 하는지 런타임까지 알 수 없을 수 있습니다. 따라서 Livewire는 `<livewire:dynamic-component ...>`를 통해 런타임에 자식 컴포넌트를 선택할 수 있게 해줍니다. 이는 `:is` prop을 받습니다:

```blade
<livewire:dynamic-component :is="$current" />
```

동적 자식 컴포넌트는 다양한 시나리오에서 유용하지만, 아래는 동적 컴포넌트를 사용하여 다단계 양식의 다른 단계를 렌더링하는 예시입니다:

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class Steps extends Component
{
    public $current = 'step-one';

    protected $steps = [
        'step-one',
        'step-two',
        'step-three',
    ];

    public function next()
    {
        $currentIndex = array_search($this->current, $this->steps);

        $this->current = $this->steps[$currentIndex + 1];
    }

    public function render()
    {
        return view('livewire.todo-list');
    }
}
```

```blade
<div>
    <livewire:dynamic-component :is="$current" :key="$current" />

    <button wire:click="next">Next</button>
</div>
```

이제 `Steps` 컴포넌트의 `$current` prop이 "step-one"으로 설정되어 있다면, Livewire는 다음과 같이 "step-one"이라는 이름의 컴포넌트를 렌더링합니다:

```php
<?php

namespace App\Livewire;

use Livewire\Component;

class StepOne extends Component
{
    public function render()
    {
        return view('livewire.step-one');
    }
}
```

원한다면 대체 구문을 사용할 수 있습니다:

```blade
<livewire:is :component="$current" :key="$current" />
```

> 주의: 각 자식 컴포넌트에 고유한 키를 할당하는 것을 잊지 마세요. Livewire는 `<livewire:dynamic-child />` 및 `<livewire:is />`에 대해 자동으로 키를 생성하지만, 그 동일한 키가 _모든_ 자식 컴포넌트에 적용되어 후속 렌더링이 건너뛰게 됩니다.
>
> 키가 컴포넌트 렌더링에 어떤 영향을 미치는지에 대한 더 깊은 이해를 위해 [자식 컴포넌트를 강제로 다시 렌더링하기](#forcing-a-child-component-to-re-render)를 참조하세요.

## 재귀적 컴포넌트

대부분의 애플리케이션에서는 거의 필요하지 않지만, Livewire 컴포넌트는 재귀적으로 중첩될 수 있습니다. 즉, 부모 컴포넌트가 자신을 자식으로 렌더링할 수 있습니다.

자신에게 하위 질문이 첨부될 수 있는 `SurveyQuestion` 컴포넌트를 포함하는 설문조사를 상상해보세요:

```php
<?php

namespace App\Livewire;

use Livewire\Component;
use App\Models\Question;

class SurveyQuestion extends Component
{
    public Question $question;

    public function render()
    {
        return view('livewire.survey-question', [
            'subQuestions' => $this->question->subQuestions,
        ]);
    }
}
```

```blade
<div>
    Question: {{ $question->content }}

    @foreach ($subQuestions as $subQuestion)
        <livewire:survey-question :question="$subQuestion" :key="$subQuestion->id" />
    @endforeach
</div>
```

> 주의: 물론, 재귀의 표준 규칙이 재귀적 컴포넌트에도 적용됩니다. 가장 중요한 것은 템플릿이 무한히 재귀하지 않도록 하는 로직이 템플릿에 있어야 한다는 것입니다. 위의 예시에서, `$subQuestion`이 자신의 `$subQuestion`으로 원래 질문을 포함하고 있다면 무한 루프가 발생할 것입니다.

## 자식 컴포넌트를 강제로 다시 렌더링하기

내부적으로 Livewire는 템플릿의 각 중첩된 Livewire 컴포넌트에 대해 키를 생성합니다.

예를 들어, 다음과 같은 중첩된 `todo-count` 컴포넌트를 고려해봅시다:

```blade
<div>
    <livewire:todo-count :$todos />
</div>
```

Livewire는 내부적으로 다음과 같이 컴포넌트에 임의의 문자열 키를 첨부합니다:

```blade
<div>
    <livewire:todo-count :$todos key="lska" />
</div>
```

부모 컴포넌트가 렌더링되고 위와 같은 자식 컴포넌트를 만나면, 부모에 첨부된 자식 목록에 키를 저장합니다:

```php
'children' => ['lska'],
```

Livewire는 이 목록을 후속 렌더링에서 참조하여 자식 컴포넌트가 이전 요청에서 이미 렌더링되었는지 감지합니다. 이미 렌더링되었다면 컴포넌트는 건너뛰어집니다. [중첩된 컴포넌트는 아일랜드](/docs/understanding-nesting#every-component-is-an-island)임을 기억하세요. 그러나 자식 키가 목록에 없다면, 즉 아직 렌더링되지 않았다면, Livewire는 컴포넌트의 새 인스턴스를 생성하고 그 자리에 렌더링합니다.

이러한 세부사항은 대부분의 사용자가 알 필요가 없는 백그라운드 동작입니다. 그러나 자식에 키를 설정하는 개념은 자식 렌더링을 제어하는 강력한 도구입니다.

이 지식을 사용하여 컴포넌트를 강제로 다시 렌더링하려면 단순히 키를 변경하면 됩니다.

아래는 컴포넌트에 전달되는 `$todos`가 변경되면 `todo-count` 컴포넌트를 파괴하고 다시 초기화하고 싶을 수 있는 예시입니다:

```blade
<div>
    <livewire:todo-count :todos="$todos" :key="$todos->pluck('id')->join('-')" />
</div>
```

위에서 볼 수 있듯이, `$todos`의 내용을 기반으로 동적 `:key` 문자열을 생성하고 있습니다. 이렇게 하면 `todo-count` 컴포넌트는 `$todos` 자체가 변경될 때까지 정상적으로 렌더링되고 존재합니다. 그 시점에서 컴포넌트는 완전히 처음부터 다시 초기화되고 이전 컴포넌트는 폐기됩니다.
