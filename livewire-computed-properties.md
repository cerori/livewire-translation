# 계산된 속성

계산된 속성은 Livewire에서 "파생된" 속성을 생성하는 방법입니다. Eloquent 모델의 접근자와 마같이, 계산된 속성을 사용하면 값에 접근하고 요청 중에 향후 접근을 위해 캐시할 수 있습니다.

계산된 속성은 특히 컴포넌트의 공개 속성과 함께 사용할 때 유용합니다.

## 기본 사용법

계산된 속성을 생성하려면 Livewire 컴포넌트의 메서드 위에 `#[Computed]` 속성을 추가할 수 있습니다. 속성이 메서드에 추가되면 다른 속성과 마찬가지로 접근할 수 있습니다.

주의: 속성 클래스를 반드시 가져와야 합니다

예를 들어, `use Livewire\Attributes\Computed;`와 같이 `#[Computed]` 속성을 사용하려면 해당 import를 반드시 추가해야 합니다.

예를 들어, 다음은 `$userId` 속성을 기반으로 `User` Eloquent 모델에 접근하기 위해 `user()`라는 계산된 속성을 사용하는 `ShowUser` 컴포넌트입니다:

```php
<?php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\User;

class ShowUser extends Component
{
    public $userId;

    #[Computed]
    public function user()
    {
        return User::find($this->userId);
    }

    public function follow()
    {
        Auth::user()->follow($this->user);
    }

    public function render()
    {
        return view('livewire.show-user');
    }
}
```

```blade
<div>
    <h1>{{ $this->user->name }}</h1>

    <span>{{ $this->user->email }}</span>

    <button wire:click="follow">Follow</button>
</div>
```

`user()` 메서드에 `#[Computed]` 속성이 추가되었기 때문에, 컴포넌트의 다른 메서드와 Blade 템플릿 내에서 값에 접근할 수 있습니다.

주의: 템플릿에서 `$this`를 사용해야 합니다

일반 속성과 달리, 계산된 속성은 컴포넌트의 템플릿 내에서 직접 사용할 수 없습니다. 대신 `$this` 객체에서 접근해야 합니다. 예를 들어, `posts()`라는 계산된 속성은 템플릿 내에서 `$this->posts`로 접근해야 합니다.

주의: 계산된 속성은 `Livewire\Form` 객체에서 지원되지 않습니다.

[Form](https://livewire.laravel.com/docs/forms) 내에서 계산된 속성을 사용하려고 하면 블레이드에서 $form->property 구문을 사용하여 속성에 접근하려고 할 때 오류가 발생합니다.

## 성능 이점

계산된 속성을 사용해야 하는 이유가 궁금할 수 있습니다. 왜 메서드를 직접 호출하지 않고 계산된 속성을 사용할까요?

메서드를 계산된 속성으로 접근하면 메서드를 호출하는 것보다 성능 이점이 있습니다. 내부적으로 계산된 속성이 처음 실행될 때 Livewire는 반환된 값을 캐시합니다. 이렇게 하면 요청에서 이후의 모든 접근은 여러 번 실행하는 대신 캐시된 값을 반환합니다.

이를 통해 파생된 값에 자유롭게 접근할 수 있으며 성능에 대한 영향을 걱정할 필요가 없습니다.

주의: 계산된 속성은 단일 요청에 대해서만 캐시됩니다

Livewire가 계산된 속성을 페이지의 Livewire 컴포넌트 전체 수명 동안 캐시한다는 것은 일반적인 오해입니다. 그러나 이는 사실이 아닙니다. 대신 Livewire는 단일 컴포넌트 요청 기간 동안만 결과를 캐시합니다. 이는 계산된 속성 메서드에 비용이 많이 드는 데이터베이스 쿼리가 포함된 경우 Livewire 컴포넌트가 업데이트를 수행할 때마다 실행된다는 것을 의미합니다.

### 캐시 무효화하기

다음과 같은 문제가 있는 시나리오를 고려해보세요:

1. 특정 속성이나 데이터베이스 상태에 의존하는 계산된 속성에 접근합니다.
2. 기본 속성이나 데이터베이스 상태가 변경됩니다.
3. 속성에 대한 캐시된 값이 오래되어 다시 계산해야 합니다.

저장된 캐시를 지우거나 "무효화"하려면 PHP의 `unset()` 함수를 사용할 수 있습니다.

다음은 애플리케이션에 새 게시물을 생성함으로써 `posts()` 계산된 속성을 오래된 것으로 만드는 `createPost()` 작업의 예입니다. 이는 `posts()` 계산된 속성이 새로 추가된 게시물을 포함하도록 다시 계산되어야 함을 의미합니다:

```php
<?php

use Illuminate\Support\Facades\Auth;
use Livewire\Attributes\Computed;
use Livewire\Component;

class ShowPosts extends Component
{
    public function createPost()
    {
        if ($this->posts->count() > 10) {
            throw new \Exception('Maximum post count exceeded');
        }

        Auth::user()->posts()->create(...);

        unset($this->posts);
    }

    #[Computed]
    public function posts()
    {
        return Auth::user()->posts;
    }

    // ...
}
```

위의 컴포넌트에서 새 게시물이 생성되기 전에 계산된 속성이 캐시되는데, 이는 `createPost()` 메서드가 새 게시물이 생성되기 전에 `$this->posts`에 접근하기 때문입니다. 뷰 내에서 접근할 때 `$this->posts`가 가장 최신 내용을 포함하도록 하기 위해 `unset($this->posts)`를 사용하여 캐시를 무효화합니다.

### 요청 간 캐싱

때로는 계산된 속성의 값을 모든 요청 후에 지우는 대신 Livewire 컴포넌트의 수명 동안 캐시하고 싶을 수 있습니다. 이러한 경우에는 [Laravel의 캐싱 유틸리티](https://laravel.com/docs/cache#retrieve-store)를 사용할 수 있습니다.

다음은 `user()`라는 계산된 속성의 예입니다. 여기서는 Eloquent 쿼리를 직접 실행하는 대신 `Cache::remember()`로 쿼리를 감싸서 향후 요청에서 Laravel의 캐시에서 검색하도록 합니다:

```php
<?php

use Illuminate\Support\Facades\Cache;
use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\User;

class ShowUser extends Component
{
    public $userId;

    #[Computed]
    public function user()
    {
        $key = 'user'.$this->getId();
        $seconds = 3600; // 1 hour...

        return Cache::remember($key, $seconds, function () {
            return User::find($this->userId);
        });
    }

    // ...
}
```

각 Livewire 컴포넌트의 고유한 인스턴스에는 고유한 ID가 있기 때문에 `$this->getId()`를 사용하여 이 동일한 컴포넌트 인스턴스의 향후 요청에만 적용될 고유한 캐시 키를 생성할 수 있습니다.

하지만 눈치채셨겠지만, 이 코드의 대부분은 예측 가능하며 쉽게 추상화할 수 있습니다. 이 때문에 Livewire의 `#[Computed]` 속성은 유용한 `persist` 매개변수를 제공합니다. 메서드에 `#[Computed(persist: true)]`를 적용하면 추가 코드 없이 동일한 결과를 얻을 수 있습니다:

```php
use Livewire\Attributes\Computed;
use App\Models\User;

#[Computed(persist: true)]
public function user()
{
    return User::find($this->userId);
}
```

위의 예에서 컴포넌트에서 `$this->user`에 접근할 때, 페이지의 Livewire 컴포넌트 수명 동안 계속 캐시됩니다. 이는 실제 Eloquent 쿼리가 한 번만 실행됨을 의미합니다.

Livewire는 지속된 값을 3600초(1시간) 동안 캐시합니다. `#[Computed]` 속성에 추가 `seconds` 매개변수를 전달하여 이 기본값을 재정의할 수 있습니다:

```php
#[Computed(persist: true, seconds: 7200)]
```

팁: `unset()`을 호출하면 이 캐시가 무효화됩니다

이전에 논의한 대로 PHP의 `unset()` 메서드를 사용하여 계산된 속성의 캐시를 지울 수 있습니다. 이는 `persist: true` 매개변수를 사용하는 계산된 속성에도 적용됩니다. 캐시된 계산된 속성에 `unset()`을 호출하면 Livewire는 계산된 속성 캐시뿐만 아니라 Laravel의 캐시에 있는 기본 캐시된 값도 지웁니다.

## 모든 컴포넌트에서 캐싱

단일 컴포넌트의 수명 동안 계산된 속성의 값을 캐시하는 대신, `#[Computed]` 속성이 제공하는 `cache: true` 매개변수를 사용하여 애플리케이션의 모든 컴포넌트에서 계산된 값을 캐시할 수 있습니다:

```php
use Livewire\Attributes\Computed;
use App\Models\Post;

#[Computed(cache: true)]
public function posts()
{
    return Post::all();
}
```

위의 예에서 캐시가 만료되거나 무효화될 때까지 애플리케이션의 이 컴포넌트의 모든 인스턴스는 `$this->posts`에 대해 동일한 캐시된 값을 공유합니다.

계산된 속성의 캐시를 수동으로 지워야 하는 경우 `key` 매개변수를 사용하여 사용자 정의 캐시 키를 설정할 수 있습니다:

```php
use Livewire\Attributes\Computed;
use App\Models\Post;

#[Computed(cache: true, key: 'homepage-posts')]
public function posts()
{
    return Post::all();
}
```

## 계산된 속성을 언제 사용해야 할까요?

성능 이점을 제공하는 것 외에도 계산된 속성이 유용한 몇 가지 다른 시나리오가 있습니다.

특히 컴포넌트의 Blade 템플릿에 데이터를 전달할 때 계산된 속성이 더 나은 대안이 되는 몇 가지 경우가 있습니다. 다음은 Blade 템플릿에 `posts` 컬렉션을 전달하는 간단한 컴포넌트의 `render()` 메서드의 예입니다:

```php
public function render()
{
    return view('livewire.show-posts', [
        'posts' => Post::all(),
    ]);
}
```

```blade
<div>
    @foreach ($posts as $post)
        <!-- ... -->
    @endforeach
</div>
```

이는 많은 사용 사례에 충분하지만, 계산된 속성이 더 나은 대안이 되는 세 가지 시나리오가 있습니다:

### 조건부로 값에 접근하기

Blade 템플릿에서 계산 비용이 많이 드는 값에 조건부로 접근하는 경우 계산된 속성을 사용하여 성능 오버헤드를 줄일 수 있습니다.

계산된 속성 없이 다음템플릿을 고려해보세요:

```blade
<div>
    @if (Auth::user()->can_see_posts)
        @foreach ($posts as $post)
            <!-- ... -->
        @endforeach
    @endif
</div>
```

사용자가 게시물을 볼 수 없는 경우에도 게시물을 검색하기 위한 데이터베이스 쿼리가 이미 실행되었지만, 템플릿에서는 게시물이 전혀 사용되지 않습니다.

다음은 대신 계산된 속성을 사용한 위 시나리오의 버전입니다:

```php
use Livewire\Attributes\Computed;
use App\Models\Post;

#[Computed]
public function posts()
{
    return Post::all();
}

public function render()
{
    return view('livewire.show-posts');
}
```

```blade
<div>
    @if (Auth::user()->can_see_posts)
        @foreach ($this->posts as $post)
            <!-- ... -->
        @endforeach
    @endif
</div>
```

이제 계산된 속성을 사용하여 템플릿에 게시물을 제공하기 때문에 데이터가 필요할 때만 데이터베이스 쿼리를 실행합니다.

### 인라인 템플릿 사용하기

계산된 속성이 유용한 또 다른 시나리오는 컴포넌트에서 [인라인 템플릿](/docs/components#inline-components)을 사용하는 경우입니다.

다음은 `render()` 내에서 직접 템플릿 문자열을 반환하기 때문에 뷰에 데이터를 전달할 기회가 없는 인라인 컴포넌트의 예입니다:

```php
<?php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

class ShowPosts extends Component
{
    #[Computed]
    public function posts()
    {
        return Post::all();
    }

    public function render()
    {
        return <<<HTML
        <div>
            @foreach ($this->posts as $post)
                <!-- ... -->
            @endforeach
        </div>
        HTML;
    }
}
```

위의 예에서 계산된 속성 없이는 Blade 템플릿에 데이터를 명시적으로 전달할 방법이 없습니다.

### render 메서드 생략하기

Livewire에서 컴포넌트의 상용구를 줄이는 또 다른 방법은 `render()` 메서드를 완전히 생략하는 것입니다. 생략하면 Livewire는 규칙에 따라 해당하는 Blade 뷰를 반환하는 자체 `render()` 메서드를 사용합니다.

이 경우에는 당연히 Blade 뷰에 데이터를 전달할 수 있는 `render()` 메서드가 없습니다.

컴포넌트에 `render()` 메서드를 다시 도입하는 대신, 계산된 속성을 통해 뷰에 해당 데이터를 제공할 수 있습니다:

```php
<?php

use Livewire\Attributes\Computed;
use Livewire\Component;
use App\Models\Post;

class ShowPosts extends Component
{
    #[Computed]
    public function posts()
    {
        return Post::all();
    }
}
```

```blade
<div>
    @foreach ($this->posts as $post)
        <!-- ... -->
    @endforeach
</div>
```
