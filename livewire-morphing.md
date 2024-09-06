# Morphing

Livewire 컴포넌트가 브라우저의 DOM을 업데이트할 때, "morphing"이라고 부르는 지능적인 방식으로 수행합니다. *morph*라는 용어는 *replace*(대체)와 대조됩니다.

컴포넌트가 업데이트될 때마다 새로 렌더링된 HTML로 컴포넌트의 HTML을 *대체*하는 대신, Livewire는 현재 HTML과 새 HTML을 동적으로 비교하여 차이점을 식별하고 변경이 필요한 부분에만 외과적으로 HTML을 수정합니다.

이는 컴포넌트의 기존 변경되지 않은 요소들을 보존하는 이점이 있습니다. 예를 들어, 이벤트 리스너, 포커스 상태, 폼 입력 값들은 모두 Livewire 업데이트 간에 보존됩니다. 물론 morphing은 모든 업데이트마다 DOM을 지우고 다시 렌더링하는 것에 비해 성능도 향상됩니다.

## Morphing 작동 방식

Livewire가 요청 간에 어떤 요소를 업데이트할지 결정하는 방법을 이해하기 위해 다음과 같은 간단한 `Todos` 컴포넌트를 고려해 봅시다:

```php
class Todos extends Component
{
    public $todo = '';
 
    public $todos = [
        'first',
        'second',
    ];
 
    public function add()
    {
        $this->todos[] = $this->todo;
    }
}
```

```blade
<form wire:submit="add">
    <ul>
        @foreach ($todos as $item)
            <li>{{ $item }}</li>
        @endforeach
    </ul>
 
    <input wire:model="todo">
</form>
```

이 컴포넌트의 초기 렌더링은 다음과 같은 HTML을 출력할 것입니다:

```blade
<form wire:submit="add">
    <ul>
        <li>first</li>
 
        <li>second</li>
    </ul>
 
    <input wire:model="todo">
</form>
```

이제 입력 필드에 "third"를 입력하고 `[Enter]` 키를 눌렀다고 상상해 보세요. 새로 렌더링된 HTML은 다음과 같을 것입니다:

```blade
<form wire:submit="add">
    <ul>
        <li>first</li>
 
        <li>second</li>
 
        <li>third</li> 
    </ul>
 
    <input wire:model="todo">
</form>
```

Livewire가 컴포넌트 업데이트를 처리할 때, 원본 DOM을 새로 렌더링된 HTML로 *morphs*(변형)합니다. 다음 시각화는 이것이 어떻게 작동하는지 직관적으로 이해하는 데 도움이 될 것입니다:

[시각화 예시]

보시다시피, Livewire는 두 HTML 트리를 동시에 순회합니다. 두 트리의 각 요소를 만날 때마다 변경, 추가, 제거 여부를 비교합니다. 변화를 감지하면 적절한 변경을 외과적으로 수행합니다.

## Morphing의 한계

다음은 morphing 알고리즘이 HTML 트리의 변경을 올바르게 식별하지 못해 애플리케이션에 문제를 일으킬 수 있는 시나리오입니다.

### 중간 요소 삽입

가상의 `CreatePost` 컴포넌트에 대한 다음 Livewire Blade 템플릿을 고려해 보세요:

```blade
<form wire:submit="save">
    <div>
        <input wire:model="title">
    </div>
 
    @if ($errors->has('title'))
        <div>{{ $errors->first('title') }}</div>
    @endif
 
    <div>
        <button>Save</button>
    </div>
</form>
```

사용자가 폼을 제출하려고 하지만 유효성 검사 오류가 발생하면 다음과 같은 문제가 발생합니다:

[문제 시각화]

보시다시피, Livewire가 오류 메시지에 대한 새로운 `<div>`를 만났을 때, 기존 `<div>`를 제자리에서 변경해야 할지, 아니면 중간에 새 `<div>`를 삽입해야 할지 모릅니다.

이 현상을 더 명확히 설명하면 다음과 같습니다:

- Livewire는 두 트리에서 첫 번째 `<div>`를 만납니다. 둘이 같으므로 계속 진행합니다.
- Livewire는 두 트리에서 두 번째 `<div>`를 만나고 이들이 같은 `<div>`라고 생각하지만, 하나의 내용만 변경되었다고 봅니다. 따라서 오류 메시지를 새 요소로 삽입하는 대신 `<button>`을 오류 메시지로 변경합니다.
- 그런 다음 Livewire는 이전 요소를 잘못 수정한 후, 비교 끝에 추가 요소가 있음을 알아챕니다. 그래서 요소를 생성하여 이전 요소 뒤에 추가합니다.
- 따라서 단순히 이동되었어야 할 요소를 파괴한 다음 다시 생성합니다.

이 시나리오는 거의 모든 morph 관련 버그의 근본 원인입니다.

이러한 버그들의 구체적인 문제점은 다음과 같습니다:

- 업데이트 사이에 이벤트 리스너와 요소 상태가 손실됩니다.
- 이벤트 리스너와 상태가 잘못된 요소에 배치됩니다.
- Livewire 컴포넌트도 DOM 트리의 단순한 요소이므로 전체 Livewire 컴포넌트가 리셋되거나 중복될 수 있습니다.
- Alpine 컴포넌트와 상태가 손실되거나 잘못 배치될 수 있습니다.

다행히 Livewire는 다음과 같은 접근 방식을 사용하여 이러한 문제를 완화하기 위해 노력했습니다:

### 내부 선행 검사

Livewire는 요소를 변경하기 전에 후속 요소와 그 내용을 확인하는 추가 단계를 morphing 알고리즘에 포함하고 있습니다.

이는 많은 경우에 위의 시나리오가 발생하는 것을 방지합니다.

다음은 "선행 검사" 알고리즘이 작동하는 모습을 시각화한 것입니다:

[시각화 예시]

### Morph 마커 주입

백엔드에서 Livewire는 Blade 템플릿 내의 조건문을 자동으로 감지하고 이를 HTML 주석 마커로 감싸서 Livewire의 JavaScript가 morphing 시 가이드로 사용할 수 있도록 합니다.

다음은 이전 Blade 템플릿에 Livewire의 주입된 마커가 포함된 예입니다:

```blade
<form wire:submit="save">
    <div>
        <input wire:model="title">
    </div>
 
    <!--[if BLOCK]><![endif]-->
    @if ($errors->has('title'))
        <div>Error: {{ $errors->first('title') }}</div>
    @endif
    <!--[if ENDBLOCK]><![endif]-->
 
    <div>
        <button>Save</button>
    </div>
</form>
```

이러한 마커가 템플릿에 주입되면 Livewire는 이제 변경과 추가의 차이를 더 쉽게 감지할 수 있습니다.

이 기능은 Livewire 애플리케이션에 매우 유익하지만, 정규식을 통한 템플릿 파싱이 필요하기 때문에 때로는 조건문을 제대로 감지하지 못할 수 있습니다. 이 기능이 애플리케이션에 도움이 되는 것보다 방해가 된다면 애플리케이션의 `config/livewire.php` 파일에서 다음 설정으로 비활성화할 수 있습니다:

```php
'inject_morph_markers' => false,
```

#### 조건문 감싸기

위의 두 가지 해결책이 상황에 맞지 않는다면, morphing 문제를 피하는 가장 안정적인 방법은 조건문과 반복문을 항상 존재하는 자체 요소로 감싸는 것입니다.

예를 들어, 위의 Blade 템플릿을 감싸는 `<div>` 요소로 다시 작성하면 다음과 같습니다:

```blade
<form wire:submit="save">
    <div>
        <input wire:model="title">
    </div>
 
    <div> 
        @if ($errors->has('title'))
            <div>{{ $errors->first('title') }}</div>
        @endif
    </div> 
 
    <div>
        <button>Save</button>
    </div>
</form>
```

이제 조건문이 영속적인 요소로 감싸졌기 때문에 Livewire는 두 개의 다른 HTML 트리를 적절하게 morph할 것입니다.

#### Morphing 우회하기

요소에 대해 morphing을 완전히 우회해야 하는 경우, [wire:replace](/docs/wire-replace)를 사용하여 Livewire에게 기존 요소를 morph하려고 시도하는 대신 요소의 모든 자식을 대체하도록 지시할 수 있습니다.
