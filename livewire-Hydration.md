# Hydration (수화)

Livewire를 사용하면 서버 측 PHP 클래스를 웹 브라우저에 직접 연결하는 것처럼 느껴집니다. 버튼 클릭으로 서버 측 함수를 직접 호출하는 등의 기능이 이러한 환상을 지원합니다. 하지만 실제로는 단지 환상일 뿐입니다.

배경에서 Livewire는 실제로 표준 웹 애플리케이션과 매우 유사하게 동작합니다. 정적 HTML을 브라우저에 렌더링하고, 브라우저 이벤트를 수신한 다음, AJAX 요청을 통해 서버 측 코드를 호출합니다.

Livewire가 서버에 보내는 각 AJAX 요청은 "상태가 없기" 때문에 (즉, 컴포넌트의 상태를 유지하는 장기 실행 백엔드 프로세스가 없음), Livewire는 업데이트를 하기 전에 컴포넌트의 마지막으로 알려진 상태를 재생성해야 합니다.

이를 위해 각 서버 측 업데이트 후 PHP 컴포넌트의 "스냅샷"을 찍어 다음 요청에서 컴포넌트를 재생성하거나 *재개*할 수 있도록 합니다.

이 문서 전반에 걸쳐 스냅샷을 찍는 과정을 "탈수(dehydration)"라고 하고, 스냅샷에서 컴포넌트를 재생성하는 과정을 "수화(hydration)"라고 부를 것입니다.

## 탈수

Livewire가 서버 측 컴포넌트를 *탈수*할 때, 두 가지 작업을 수행합니다:

- 컴포넌트의 템플릿을 HTML로 렌더링
- 컴포넌트의 JSON 스냅샷 생성

### HTML 렌더링

컴포넌트가 마운트되거나 업데이트가 이루어진 후, Livewire는 컴포넌트의 `render()` 메서드를 호출하여 Blade 템플릿을 원시 HTML로 변환합니다.

예를 들어 다음과 같은 `Counter` 컴포넌트를 살펴보겠습니다:

```php
class Counter extends Component
{
    public $count = 1;

    public function increment()
    {
        $this->count++;
    }

    public function render()
    {
        return <<<'HTML'
        <div>
            Count: {{ $count }}

            <button wire:click="increment">+</button>
        </div>
        HTML;
    }
}
```

각 마운트나 업데이트 후, Livewire는 위의 `Counter` 컴포넌트를 다음과 같은 HTML로 렌더링합니다:

```html
<div>
  Count: 1

  <button wire:click="increment">+</button>
</div>
```

### 스냅샷

다음 요청 동안 서버에서 `Counter` 컴포넌트를 재생성하기 위해, 가능한 한 많은 컴포넌트 상태를 캡처하려고 시도하는 JSON 스냅샷이 생성됩니다:

```js
{
    state: {
        count: 1,
    },

    memo: {
        name: 'counter',
        id: '1526456',
    },
}
```

스냅샷의 두 가지 다른 부분인 `memo`와 `state`를 주목하세요.

`memo` 부분은 컴포넌트를 식별하고 재생성하는 데 필요한 정보를 저장하는 데 사용되며, `state` 부분은 모든 컴포넌트의 공개 속성 값을 저장합니다.

> 💡 위의 스냅샷은 실제 Livewire의 스냅샷을 간소화한 버전입니다. 실제 애플리케이션에서 스냅샷에는 유효성 검사 오류, 자식 컴포넌트 목록, 로케일 등 훨씬 더 많은 정보가 포함됩니다. 스냅샷 객체에 대한 더 자세한 내용은 [스냅샷 스키마 문서](/docs/javascript#the-snapshot-object)를 참조하실 수 있습니다.

### HTML에 스냅샷 포함하기

컴포넌트가 처음 렌더링될 때, Livewire는 스냅샷을 `wire:snapshot`이라는 HTML 속성 안에 JSON으로 저장합니다. 이렇게 하면 Livewire의 JavaScript 코어가 JSON을 추출하여 런타임 객체로 변환할 수 있습니다:

```html
<div wire:id="..." wire:snapshot="{ state: {...}, memo: {...} }">
  Count: 1

  <button wire:click="increment">+</button>
</div>
```

## 수화

컴포넌트 업데이트가 트리거될 때, 예를 들어 `Counter` 컴포넌트에서 "+" 버튼이 눌렸을 때, 다음과 같은 페이로드가 서버로 전송됩니다:

```js
{
    calls: [
        { method: 'increment', params: [] },
    ],

    snapshot: {
        state: {
            count: 1,
        },

        memo: {
            name: 'counter',
            id: '1526456',
        },
    }
}
```

Livewire가 `increment` 메서드를 호출하기 전에, 먼저 새로운 `Counter` 인스턴스를 생성하고 스냅샷의 상태로 초기화해야 합니다.

다음은 이 결과를 달성하는 PHP 의사 코드입니다:

```php
$state = request('snapshot.state');
$memo = request('snapshot.memo');

$instance = Livewire::new($memo['name'], $memo['id']);

foreach ($state as $property => $value) {
    $instance[$property] = $value;
}
```

위 스크립트를 따라가면, `Counter` 객체를 생성한 후 스냅샷에서 제공된 상태를 기반으로 공개 속성이 설정되는 것을 볼 수 있습니다.

## 고급 수화

위의 `Counter` 예제는 수화의 개념을 잘 보여주지만, 정수(`1`)와 같은 단순한 값을 Livewire가 어떻게 처리하는지만 보여줍니다.

아시다시피, Livewire는 정수 이외에도 훨씬 더 복잡한 속성 유형을 지원합니다.

조금 더 복잡한 예제인 `Todos` 컴포넌트를 살펴보겠습니다:

```php
class Todos extends Component
{
    public $todos;

    public function mount() {
        $this->todos = collect([
            'first',
            'second',
            'third',
        ]);
    }
}
```

보시다시피, `$todos` 속성을 세 개의 문자열을 내용으로 하는 [Laravel 컬렉션](https://laravel.com/docs/collections#main-content)으로 설정하고 있습니다.

JSON만으로는 Laravel 컬렉션을 표현할 방법이 없기 때문에, Livewire는 스냅샷 내에서 순수 데이터와 메타데이터를 연결하는 자체 패턴을 만들었습니다.

다음은 이 `Todos` 컴포넌트의 스냅샷 상태 객체입니다:

```js
state: {
    todos: [
        [ 'first', 'second', 'third' ],
        { s: 'clctn', class: 'Illuminate\\Support\\Collection' },
    ],
},
```

이것이 다음과 같은 더 단순한 형태를 예상했다면 혼란스러울 수 있습니다:

```js
state: {
    todos: [ 'first', 'second', 'third' ],
},
```

하지만 Livewire가 이 데이터를 기반으로 컴포넌트를 수화한다면, 이것이 컬렉션인지 아니면 단순한 배열인지 알 수 있는 방법이 없을 것입니다.

따라서 Livewire는 튜플(두 개의 항목으로 이루어진 배열) 형태의 대체 상태 구문을 지원합니다:

```js
todos: [
    [ 'first', 'second', 'third' ],
    { s: 'clctn', class: 'Illuminate\\Support\\Collection' },
],
```

Livewire가 컴포넌트의 상태를 수화할 때 튜플을 만나면, 튜플의 두 번째 요소에 저장된 정보를 사용하여 첫 번째 요소에 저장된 상태를 더 지능적으로 수화합니다.

더 명확하게 설명하기 위해, 다음은 Livewire가 위의 스냅샷을 기반으로 컬렉션 속성을 어떻게 재생성할 수 있는지 보여주는 간단한 코드입니다:

```php
[ $state, $metadata ] = request('snapshot.state.todos');

$collection = new $metadata['class']($state);
```

보시다시피, Livewire는 상태와 연결된 메타데이터를 사용하여 전체 컬렉션 클래스를 유추합니다.

### 깊이 중첩된 튜플

이 접근 방식의 한 가지 뚜렷한 장점은 깊이 중첩된 속성을 탈수하고 수화할 수 있다는 것입니다.

예를 들어, 위의 `Todos` 예제에서 컬렉션의 세 번째 항목을 일반 문자열 대신 [Laravel Stringable](https://laravel.com/docs/helpers#method-str)로 변경해 보겠습니다:

```php
class Todos extends Component
{
    public $todos;

    public function mount() {
        $this->todos = collect([
            'first',
            'second',
            str('third'),
        ]);
    }
}
```

이 컴포넌트의 상태에 대한 탈수된 스냅샷은 이제 다음과 같이 보일 것입니다:

```js
todos: [
    [
        'first',
        'second',
        [ 'third', { s: 'str' } ],
    ],
    { s: 'clctn', class: 'Illuminate\\Support\\Collection' },
],
```

보시다시피, 컬렉션의 세 번째 항목이 메타데이터 튜플로 탈수되었습니다. 튜플의 첫 번째 요소는 일반 문자열 값이고, 두 번째는 이 문자열이 *stringable*임을 Livewire에 알리는 플래그입니다.

### 사용자 정의 속성 유형 지원

내부적으로 Livewire는 가장 일반적인 PHP와 Laravel 유형에 대한 수화 지원을 제공합니다. 그러나 지원되지 않는 유형을 지원하고 싶다면, [Synthesizers](/docs/synthesizers)를 사용할 수 있습니다 --- 이는 Livewire의 내부 메커니즘으로 비기본 속성 유형을 수화/탈수하는 데 사용됩니다.
