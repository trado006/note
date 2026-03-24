### Livewire là gì?

+ Laravel Livewire là một framework giúp bạn xây dựng UI động (interactive UI) chỉ bằng PHP, mà gần như không cần viết JavaScript.
+ Livewire cho phép viết logic ở server (PHP) → tự động sync lên UI → UI update realtime
+ Server-side reactive system over stateless transport

### Cơ chế cốt lõi

Livewire dùng:
+ AJAX request (ngầm)
+ DOM diff (morph DOM)
+ Serialize state

Ưu điểm:
+ Không reload page
+ Không cần viết API
+ Không cần Vue/React

### Lifecycle

#### 1. Mount (khởi tạo lần đầu)
+ Chỉ chạy 1 lần duy nhất khi component được load lần đầu
+ Tương đương constructor nhưng dùng cho Livewire

```php
public function mount($id)
{
    $this->post = Post::find($id);
}
```

Dùng để:
+ Inject dữ liệu ban đầu
+ Setup state

#### 2. Hydrate (mỗi request sau đó)
+ Chạy mỗi lần request AJAX
+ Rebuild lại state từ frontend gửi lên

```php
public function hydrate()
{
    // chạy mỗi request
}
```

=> Hiểu đơn giản:
+ Livewire không giữ object thật giữa các request
+ Nó serialize → gửi lên client → gửi lại → rebuild

#### 3. Updating / Updated (hook khi data thay đổi)
Trước khi update:
```
public function updating($name, $value)
```
Sau khi update:
```
public function updated($name, $value)
```
Hoặc cụ thể field:
```
public function updatingTitle()
public function updatedTitle()
```

=> Dùng để:
+ Validate realtime
+ Chặn update

#### 4. Action (user trigger)

Khi user gọi:
```js
<button wire:click="save">
```

=> Livewire sẽ gọi:

```php
public function save()
```

=> Đây là nơi xử lý business logic chính

#### 5. Render (quan trọng nhất)
```php
public function render()
{
    return view('livewire.post');
}
```

Chạy:
+ Lần đầu
+ Sau mỗi update/action

=> Trả về Blade view

#### 6. Dehydrate (trước khi trả response)
Serialize state → gửi về frontend
```php
public function dehydrate()
```

👉 Đây là bước: Convert PHP → JSON

#### 7. DOM Diff (frontend)
+ Livewire dùng morphing (giống virtual DOM nhẹ)
+ Chỉ update phần thay đổi

👉 Không reload page

### Flow thực tế (rất quan trọng)
Khi load lần đầu:
```
mount → render → HTML
```

Khi user click / input:
```
hydrate → updating → action → updated → render → dehydrate → DOM update
```

### Một số insight quan trọng
#### 1. Livewire KHÔNG giữ state thật
+ Mỗi request là stateless (giống API)
+ State được serialize

👉 Giống concept của:
React nhưng chạy server-side

#### 2. Không nên query nặng trong render

Sai ❌:
```php
public function render()
{
    return view('...', [
        'posts' => Post::all()
    ]);
}
```

Đúng ✅:

Cache hoặc load ở mount

#### 3. Hook hydrate/dehydrate rất mạnh

Dùng để:
+ Custom serialize
+ Debug lifecycle

#### 4. Updating vs Updated
+ updating: chặn input
+ updated: xử lý sau khi thay đổi

