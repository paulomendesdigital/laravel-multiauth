# Laravel 6 Multiauth
Step by step to create a multiauth system with Laravel 6

At the example below we will create a multiauth with login and admin area

In the terminal run the code below (this needs of composer)

```
composer require laravel/ui --dev
```

```
php artisan ui:auth
```

Create a folder at `app/Http/Controllers` with name `Admin`

Copy all files from `app/Http/Controllers/Auth` to `app/Http/Controllers/Admin`
  - Change the namespace of the copied files from `Auth` to `Admin`

Create a folder at `resources/view` with name `admin`

Copy all files and folders from `resources/views/auth` to `resources/views/admin`
  - Add `admin.` at the begin of routes

In the `routes/web.php` create the routes below

```
Route::GET('admin/home', 'AdminController@index');

Route::group(['namespace' => 'Admin'], function() {

    Route::GET('admin', 'LoginController@showLoginForm')->name('admin.login');
    Route::POST('admin', 'LoginController@login');
    Route::POST('admin-password/email', 'ForgotPasswordController@sendResetLinkEmail')->name('admin.email');
    Route::GET('admin-password/reset', 'ForgotPasswordController@showLinkRequestForm')->name('admin.password.request');
    Route::POST('admin-password/reset', 'ResetPasswordController@reset');
    Route::GET('admin-password/reset/{token}', 'ResetPasswordController@showResetForm')->name('admin.password.reset');
});
```

Create a file at `app/Http/Controllers` with name `AdminController.php`

Copy the content of `HomeController.php` file at `app/Http/Controllers` to `AdminController.php`

Change the class name from `HomeController` to `AdminController` in the `AdminController.php` file

Comment the line `$this->middleware('auth');` at `AdminController.php` like example below

```
public function __construct()
{
    // $this->middleware('auth');
}
```

Add the line below at `config/auth.php` in the `guards` array, after `web` array

```
'admin' => [
    'driver' => 'session',
    'provider' => 'admins',
],
```

Add the line below at `config/auth.php` in the `providers` array, after `users` array

```
'admins' => [
    'driver' => 'eloquent',
    'model' => App\Admin::class,
],
```

Add the line below at `config/auth.php` in the `passwords` array, after `users` array

```
'admins' => [
    'provider' => 'admins',
    'table' => 'password_resets',
    'expire' => 60,
],
```

Create a file at `app` with name `Admin.php`

Copy all content from `app/User.php` to `app/Admin.php`

Change the class name from `User` to `Admin` in the `app/Admin.php` file

---
At the `app/Http/Controllers/Admin/ForgotController.php` change de below lines

this:
```
$this->middleware('guest');
```
to:
```
$this->middleware('guest:admin');
```
---

At the `app/Http/Controllers/Admin/LoginController.php` change de below lines

this:
```
protected $redirectTo = '/home';
```
to :
```
protected $redirectTo = 'admin/home';
```
---
and this:
```
$this->middleware('guest')->except('logout');
```
to:
```
$this->middleware('guest:admin')->except('logout');
```
---

At the `app/Http/Controllers/Admin/RegisterController.php` change de below lines

this:
```
protected $redirectTo = '/home';
```
to :
```
protected $redirectTo = 'admin/home';
```
---
and this:
```
$this->middleware('guest');
```
to:
```
$this->middleware('guest:admin');
```
---

At the `app/Http/Controllers/Admin/ResetController.php` change de below lines

this:
```
protected $redirectTo = '/home';
```
to :
```
protected $redirectTo = 'admin/home';
```
---
and this:
```
$this->middleware('guest');
```
to:
```
$this->middleware('guest:admin');
```
---

At the `app/Http/Controllers/Admin/VerificationController.php` change de below lines

this:
```
protected $redirectTo = '/home';
```
to :
```
protected $redirectTo = 'admin/home';
```
---
and this:
```
$this->middleware('auth');
```
to:
```
$this->middleware('auth:admin');
```
---

Create a migration to `Admin` typing the code below at the terminal

```
php artisan make:migration createAdminTable --create=Admins
```

Copy all the `Schema` from `User` migration to `Admin` migrate you just created
- Example of user's Schema:
```
$table->bigIncrements('id');
$table->string('name');
$table->string('email')->unique();
$table->timestamp('email_verified_at')->nullable();
$table->string('password');
$table->rememberToken();
$table->timestamps();
```

At the terminal run migrate with the code below

```
php artisan migrate
```

Go to `app/Http/Controllers/Admin/LoginController.php` and you'll see the code `use AuthenticatesUsers;`

If your IDE permite, press Ctrl and click on the `AuthenticatesUsers` or open the `AuthenticatesUsers.php` file at `vendor/laravel/framework/src/Illuminate/Foundation/Auth/`

Copy the function `showLoginForm()` from `AuthenticateUsers.php` to `app/Http/Controllers/Admin/LoginController.php`

```
public function showLoginForm()
{
    return view('auth.login');
}
```

In the function `showLoginForm()` inside `app/Http/Controllers/Admin/LoginController.php` make the change below

---
this:
```
return view('auth.login');
```
to:
```
return view('admin.login');
```
---

Copy the function `guard()` from `AuthenticateUsers.php` to `app/Http/Controllers/Admin/LoginController.php`

```
protected function guard()
{
    return Auth::guard('admin');
}
```

Use Auth Facade in `LoginController.php` file

```
use Illuminate\Support\Facades\Auth;
```

Uncomment the line `$this->middleware('auth');` at `app/Http/Controller/AdminController.php` file like example below 

```
public function __construct()
{
    $this->middleware('auth');
}
```

And still inside `AdminController.php` file make the change below

---
this:
```
$this->middleware('auth');
```
to:
```
$this->middleware('auth:admin');
```

Open the `RedirectIfAuthenticated.php` file at `app/Http/Middleware/` and make the changes below

---
this:
```
public function handle($request, Closure $next, $guard = null)
{
    if (Auth::guard($guard)->check()) {
        return redirect('/home');
    }

    return $next($request);
}
```
to:
```
public function handle($request, Closure $next, $guard = null)
{
    switch ($guard) {

        case 'admin':
            if (Auth::guard($guard)->check())
                return redirect('admin/home');

            break;

        default:
            if (Auth::guard($guard)->check())
                return redirect('/home');

            break;

    }


    return $next($request);
}
```

At the `Handler.php` file at `app/Exceptions` use the `AuthenticationException` like below

```
use Illuminate\Auth\AuthenticationException;
```

And still inside the `Handler.php` add the function `unauthenticated()` like below

```
protected function unauthenticated($request, AuthenticationException $exception)
{
    if ($request->expectsJson())
        return response()->json(['message' => $exception->getMessage()], 401);

    $guard = $exception->guards()[0];

    switch ($guard) {
        case 'admin':
            return redirect()->guest(route('admin.login'));
            break;

        default:
            return redirect()->guest($exception->redirectTo() ?? route('login'));
            break;
    }
}
```

The code above is to overwrite the function inside the `Illuminate Handler.php` file that is using at the class `Handler` at path `app/Exceptions/`

You can see inside the `Handler.php` file the class usage on the line containing `use Illuminate\Foundation\Exceptions\Handler as ExceptionHandler;` code
