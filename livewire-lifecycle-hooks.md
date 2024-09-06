# 라이프사이클 훅

Livewire는 컴포넌트의 라이프사이클 동안 특정 시점에 코드를 실행할 수 있게 해주는 다양한 라이프사이클 훅을 제공합니다. 이러한 훅들을 사용하면 컴포넌트 초기화, 속성 업데이트, 템플릿 렌더링 등과 같은 특정 이벤트 전후에 작업을 수행할 수 있습니다.

다음은 사용 가능한 모든 컴포넌트 라이프사이클 훅의 목록입니다:

  훅 메서드                        설명
  --------------------------------- ---------------------------------------------------------------------------------
  `mount()`                         컴포넌트가 생성될 때 호출됨
  `hydrate()`                       후속 요청의 시작 시 컴포넌트가 재수화될 때 호출됨
  `boot()`                          모든 요청의 시작 시 호출됨. 초기 요청과 후속 요청 모두에서 호출됨
  `updating()`                      컴포넌트 속성을 업데이트하기 전에 호출됨
  `updated()`                       속성을 업데이트한 후에 호출됨
  `rendering()`                     `render()` 호출 전에 호출됨
  `rendered()`                      `render()` 호출 후에 호출됨
  `dehydrate()`                     모든 컴포넌트 요청의 끝에 호출됨
  `exception($e, $stopProgation)`   예외가 발생했을 때 호출됨

## Mount

일반적인 PHP 클래스에서는 생성자(`__construct()`)가 외부 매개변수를 받아 객체의 상태를 초기화합니다. 그러나 Livewire에서는 `mount()` 메서드를 사용하여 매개변수를 받고 컴포넌트의 상태를 초기화합니다.

Livewire 컴포넌트는 `__construct()`를 사용하지 않습니다. 왜냐하면 Livewire 컴포넌트는 후속 네트워크 요청에서 *재구성*되기 때문이며, 우리는 컴포넌트가 처음 생성될 때 한 번만 초기화하기를 원하기 때문입니다.

다음은 `mount()` 메서드를 사용하여 `UpdateProfile` 컴포넌트의 `name`과 `email` 속성을 초기화하는 예제입니다:

```php
use Illuminate\Support\Facades\Auth;
use Livewire\Component;

class UpdateProfile extends Component
{
    public $name;
    public $email;

    public function mount()
    {
        $this->name = Auth::user()->name;
        $this->email = Auth::user()->email;
    }

    // ...
}
```

앞서 언급했듯이, `mount()` 메서드는 컴포넌트로 전달된 데이터를 메서드 매개변수로 받습니다:

```php
use Livewire\Component;
use App\Models\Post;

class UpdatePost extends Component
{
    public $title;
    public $content;

    public function mount(Post $post)
    {
        $this->title = $post->title;
        $this->content = $post->content;
    }

    // ...
}
```

모든 훅 메서드에서 의존성 주입을 사용할 수 있습니다

Livewire는 [Laravel의 서비스 컨테이너](https://laravel.com/docs/container#automatic-injection)에서 라이프사이클 훅의 메서드 매개변수에 타입 힌트를 지정하여 의존성을 해결할 수 있도록 합니다.

`mount()` 메서드는 Livewire 사용에 있어 중요한 부분입니다. 다음 문서에서는 `mount()` 메서드를 사용하여 일반적인 작업을 수행하는 추가 예제를 제공합니다:

- [속성 초기화](/docs/properties#initializing-properties)
- [부모 컴포넌트로부터 데이터 받기](/docs/nesting#passing-props-to-children)
- [라우트 매개변수 접근하기](/docs/components#accessing-route-parameters)

## Boot

`mount()`가 유용하지만, 이는 컴포넌트 라이프사이클에서 한 번만 실행됩니다. 주어진 컴포넌트에 대한 모든 서버 요청의 시작에 로직을 실행하고 싶을 수 있습니다.

이런 경우를 위해 Livewire는 `boot()` 메서드를 제공합니다. 여기에 컴포넌트 클래스가 부팅될 때마다 실행하려는 컴포넌트 설정 코드를 작성할 수 있습니다: 초기화 시와 후속 요청 모두에서 실행됩니다.

`boot()` 메서드는 요청 간에 유지되지 않는 protected 속성을 초기화하는 데 유용할 수 있습니다. 아래는 protected 속성을 Eloquent 모델로 초기화하는 예제입니다:

```php
use Livewire\Attributes\Locked;
use Livewire\Component;
use App\Models\Post;

class ShowPost extends Component
{
    #[Locked]
    public $postId = 1;

    protected Post $post;

    public function boot() 
    {
        $this->post = Post::find($this->postId);
    }

    // ...
}
```

이 기술을 사용하여 Livewire 컴포넌트에서 컴포넌트 속성 초기화를 완전히 제어할 수 있습니다.

대부분의 경우 computed 속성을 대신 사용할 수 있습니다

위에서 사용된 기술은 강력하지만, 이런 사용 사례를 해결하기 위해 [Livewire의 computed 속성](/docs/computed-properties)을 사용하는 것이 종종 더 좋습니다.

항상 민감한 public 속성을 잠그세요

위에서 볼 수 있듯이, `$postId` 속성에 `#[Locked]` 속성을 사용하고 있습니다. 위와 같은 시나리오에서 클라이언트 측에서 사용자가 `$postId` 속성을 변조하지 못하도록 하려면, 사용하기 전에 속성의 값을 인증하거나 속성에 `#[Locked]`를 추가하여 절대 변경되지 않도록 하는 것이 중요합니다.

자세한 내용은 [Locked 속성에 대한 문서](/docs/locked)를 확인하세요.

## Update

클라이언트 측 사용자는 여러 가지 방법으로 public 속성을 업데이트할 수 있으며, 가장 일반적으로는 `wire:model`이 있는 입력을 수정하는 것입니다.

Livewire는 public 속성의 업데이트를 가로채는 편리한 훅을 제공하여 값이 설정되기 전에 유효성을 검사하거나 인증하거나, 주어진 형식으로 속성이 설정되도록 할 수 있습니다.

아래는 `updating`을 사용하여 `$postId` 속성의 수정을 방지하는 예제입니다.

이 특정 예제의 경우, 실제 애플리케이션에서는 위의 예제와 같이 [`#[Locked]` 속성](/docs/locked)을 대신 사용해야 한다는 점에 주목할 가치가 있습니다.

```php
use Exception;
use Livewire\Component;

class ShowPost extends Component
{
    public $postId = 1;

    public function updating($property, $value)
    {
        // $property: 현재 업데이트되고 있는 속성의 이름
        // $value: 속성에 설정될 값

        if ($property === 'postId') {
            throw new Exception;
        }
    }

    // ...
}
```

위의 `updating()` 메서드는 속성이 업데이트되기 전에 실행되어 잘못된 입력을 잡아내고 속성이 업데이트되는 것을 방지할 수 있습니다. 아래는 `updated()` 메서드를 사용하여 속성의 값이 일관되게 유지되도록 하는 예제입니다:

```php
use Livewire\Component;

class CreateUser extends Component
{
    public $username = '';
    public $email = '';

    public function updated($property)
    {
        // $property: 업데이트된 현재 속성의 이름

        if ($property === 'username') {
            $this->username = strtolower($this->username);
        }
    }

    // ...
}
```

이제 클라이언트 측에서 `$username` 속성이 업데이트될 때마다 값이 항상 소문자가 되도록 보장할 수 있습니다.

update 훅을 사용할 때 종종 특정 속성을 대상으로 하기 때문에, Livewire는 메서드 이름의 일부로 속성 이름을 직접 지정할 수 있도록 합니다. 다음은 위의 예제를 이 기술을 사용하여 다시 작성한 것입니다:

```php
use Livewire\Component;

class CreateUser extends Component
{
    public $username = '';
    public $email = '';

    public function updatedUsername()
    {
        $this->username = strtolower($this->username);
    }

    // ...
}
```

물론 이 기술을 `updating` 훅에도 적용할 수 있습니다.

### 배열

배열 속성은 이 함수들에 변경되는 요소를 지정하기 위한 추가적인 `$key` 인자가 전달됩니다.

배열 자체가 업데이트되고 특정 키가 아닌 경우 `$key` 인자가 null임에 주의하세요.

```php
use Livewire\Component;

class UpdatePreferences extends Component
{
    public $preferences = [];

    public function updatedPreferences($value, $key)
    {
        // $value = 'dark'
        // $key   = 'theme'
    }

    // ...
}
```

## Hydrate & Dehydrate

Hydrate와 dehydrate는 잘 알려지지 않고 덜 사용되는 훅입니다. 그러나 특정 시나리오에서는 강력할 수 있습니다.

"dehydrate"와 "hydrate"라는 용어는 Livewire 컴포넌트가 클라이언트 측을 위해 JSON으로 직렬화되고 후속 요청에서 다시 PHP 객체로 역직렬화되는 과정을 나타냅니다.

우리는 Livewire의 코드베이스와 문서 전체에서 이 과정을 나타내기 위해 "hydrate"와 "dehydrate"라는 용어를 자주 사용합니다. 이 용어들에 대해 더 명확히 알고 싶다면, [우리의 hydration 문서를 참조](/docs/hydration)하여 자세히 알아볼 수 있습니다.

`mount()`, `hydrate()`, `dehydrate()`를 모두 함께 사용하여 Eloquent 모델 대신 사용자 정의 [데이터 전송 객체(DTO)](https://en.wikipedia.org/wiki/Data_transfer_object)를 사용하여 컴포넌트에 게시물 데이터를 저장하는 예제를 살펴보겠습니다:

```php
use Livewire\Component;

class ShowPost extends Component
{
    public $post;

    public function mount($title, $content)
    {
        // 첫 번째 초기 요청의 시작에 실행됩니다...

        $this->post = new PostDto([
            'title' => $title,
            'content' => $content,
        ]);
    }

    public function hydrate()
    {
        // 모든 "후속" 요청의 시작에 실행됩니다...
        // 이는 초기 요청에서는 실행되지 않습니다("mount"가 실행됩니다)...

        $this->post = new PostDto($this->post);
    }

    public function dehydrate()
    {
        // 모든 단일 요청의 끝에 실행됩니다...

        $this->post = $this->post->toArray();
    }

    // ...
}
```

이제 컴포넌트 내의 액션과 다른 곳에서 기본 데이터 대신 `PostDto` 객체에 접근할 수 있습니다.

위의 예제는 주로 `hydrate()`와 `dehydrate()` 훅의 능력과 특성을 보여줍니다. 그러나 이를 달성하기 위해 [Wireables 또는 Synthesizers](/docs/properties#supporting-custom-types)를 사용하는 것을 권장합니다.

## Render

컴포넌트의 Blade 뷰를 렌더링하는 과정에 개입하고 싶다면, `rendering()`과 `rendered()` 훅을 사용할 수 있습니다:

```php
use Livewire\Component;
use App\Models\Post;

class ShowPosts extends Component
{
    public function render()
    {
        return view('livewire.show-posts', [
            'post' => Post::all(),
        ])
    }

    public function rendering($view, $data)
    {
        // 제공된 뷰가 렌더링되기 전에 실행됩니다...
        //
        // $view: 렌더링될 뷰
        // $data: 뷰에 제공된 데이터
    }

    public function rendered($view, $html)
    {
        // 제공된 뷰가 렌더링된 후에 실행됩니다...
        //
        // $view: 렌더링된 뷰
        // $html: 최종적으로 렌더링된 HTML
    }

    // ...
}
```

## Exception

때때로 오류를 가로채고 잡는 것이 도움이 될 수 있습니다. 예를 들어 오류 메시지를 사용자 정의하거나 특정 유형의 예외를 무시하기 위해서입니다. `exception()` 훅을 사용하면 이를 수행할 수 있습니다: `$error`에 대한 검사를 수행하고, `$stopPropagation` 매개변수를 사용하여 문제를 잡을 수 있습니다. 이는 또한 코드의 추가 실행을 중지하고 싶을 때(조기 반환) 강력한 패턴을 제공합니다. 이것이 `validate()` 같은 내부 메서드가 작동하는 방식입니다.

```php
use Livewire\Component;

class ShowPost extends Component
{
    public function mount() 
    {
        $this->post = Post::find($this->postId);
    }

    public function exception($e, $stopPropagation) {
        if($e instanceof NotFoundException) {
            $this->notify('Post is not found')
            $stopPropagation();
        }
    }

    // ...
}
```

## 트레이트 내에서 훅 사용하기

트레이트는 컴포넌트 간에 코드를 재사용하거나 단일 컴포넌트의 코드를 전용 파일로 추출하는 데 유용한 방법입니다.

라이프사이클 훅 메서드를 선언할 때 여러 트레이트가 서로 충돌하는 것을 피하기 위해, Livewire는 현재 선언하고 있는 트레이트의 *camelCased* 이름으로 훅 메서드 이름에 접두사를 붙이는 것을 지원합니다.

이렇게 하면 동일한 라이프사이클 훅을 사용하는 여러 트레이트를 가질 수 있고 충돌하는 메서드 정의를 피할 수 있습니다.

아래는 `HasPostForm`이라는 트레이트를 참조하는 컴포넌트의 예입니다:

```php
use Livewire\Component;

class CreatePost extends Component
{
    use HasPostForm;

    // ...
}
```

이제 사용 가능한 모든 접두사가 붙은 훅이 포함된 실제 `HasPostForm` 트레이트입니다:

```php
trait HasPostForm
{
    public $title = '';
    public $content = '';

    public function mountHasPostForm()
    {
        // ...
    }

    public function hydrateHasPostForm()
    {
        // ...
    }

    public function bootHasPostForm()
    {
        // ...
    }

    public function updatingHasPostForm()
    {
        // ...
    }

    public function updatedHasPostForm()
    {
        // ...
    }

    public function renderingHasPostForm()
    {
        // ...
    }

    public function renderedHasPostForm()
    {
        // ...
    }

    public function dehydrateHasPostForm()
    {
        // ...
    }

    // ...
}
```
