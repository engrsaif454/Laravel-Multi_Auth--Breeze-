

- **Laravel Breeze** ব্যবহার করে Role wise ইউজার-কে আলাদা আলাদা **Dashboard** এ পাঠাতে হলে নির্দিষ্ট ৭টি ধাপ অনুসরণ করতে হবে। সেগুলো হলোঃ

## Step-1: 

- User Table এ নিচের কলামটি থাকতে হবেঃ

```php
$table->enum('role', ['superadmin', 'admin', 'user'])->default('user');
```
____

## Step-2:

- User Model -এ কলামের নাম যুক্ত করাঃ

```php 
protected $fillable = [
    'name',
    'email',
    'password',
    'role', // add this
];
```
_______

## Step-3:

- `AuthenticatedSessionController.php` তে নিচের দেয়া কোডটি লিখতে হবেঃ

1. পদ্ধতি-১ঃ যদি Remember Token use করতে চাইঃ 
```php
public function store(Request $request): RedirectResponse
    {
        $request->validate([
            'email' => 'required|email',
            'password' => 'required',
        ]);

        if (!Auth::attempt($request->only('email', 'password'), $request->boolean('remember'))) {
            return back()->withErrors([
                'email' => 'The provided credentials do not match our records.',
            ]);
        }

        $request->session()->regenerate();

        $user = Auth::user();

        switch ($user->role) {
            case 'superadmin':
                return redirect()->route('superadmin.dashboard');
            case 'admin':
                return redirect()->route('admin.dashboard');
            default:
                return redirect()->route('user.dashboard');
        }
    }
```


2. পদ্ধতি-১ঃ  **(Simple Way)** যদি Remember Token use না করলেও হয়ঃ 
   
```php
public function store(LoginRequest $request): RedirectResponse
    {
        $request->authenticate();

        $request->session()->regenerate();

        $user = Auth::user(); // লগিন হওয়া ইউজার

        if ($user->role == 'superadmin') {
            return redirect('/superadmin/dashboard');
        } elseif ($user->role == 'admin') {
            return redirect('/admin/dashboard');
        } elseif ($user->role == 'user') {
            return redirect('/user/dashboard');
        } else {
            return redirect('/');
        }
    }
```
_______

## Step-4:

1. Middleware Create করাঃ
```bash
php artisan make:middleware RoleMiddleware
```

2. Middleware -এ কন্ডিশন দেওয়াঃ
```php
public function handle(Request $request, Closure $next, string $role): Response
    {
        if (auth()->check() && auth()->user()->role === $role) {
            return $next($request);
        }

        abort(403); // যদি role match না করে, তাহলে 403 Forbidden দেখাবে
    }
```
_______

## Step-5:

- Middleware রেজিস্ট্রেশনঃ

`kernel.php`:
```php
protected $middlewareAliases = [
        // Other Middlewares...........
        'role' => \App\Http\Middleware\RoleMiddleware::class
    ];
```
______

## Step-6:

- `routes/web.php` ফাইলে Route গুলো যুক্ত করো:

```php
/*
|--------------------------------------------------------------------------
| Super Admin Routes.
|--------------------------------------------------------------------------
*/
Route::middleware(['auth', 'verified', 'role:superadmin'])->group(function () {
    Route::get('/superadmin/dashboard', function () {
        return view('super-admin.dashboard');
    })->name('superadmin.dashboard');
});
/*
|--------------------------------------------------------------------------
| Admin Routes.
|--------------------------------------------------------------------------
*/
Route::middleware(['auth', 'verified', 'role:admin'])->group(function () {
    Route::get('/admin/dashboard', function () {
        return view('admin.dashboard');
    })->name('admin.dashboard');
});
/*
|--------------------------------------------------------------------------
| User Routes.
|--------------------------------------------------------------------------
*/
Route::middleware(['auth', 'verified', 'role:user'])->group(function () {
    Route::get('/user/dashboard', function () {
        return view('user.dashboard');
    })->name('user.dashboard');
});
```
____

## Step-7:

- এবার আলাদা আলাদা User এর জন্য আলাদা **Dashboard** Create করলেই হবে। 

