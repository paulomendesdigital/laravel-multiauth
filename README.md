# Laravel 6 Multiauth
Step by step to create a multiauth system with Laravel 6

At the example below we will create a multiauth with login and admin area

In the terminal run the code below to generate Laravel 6 default login system

```
composer require laravel/ui --dev
```

```
php artisan ui:auth
```

Create a folder at `app/Http/Controllers/` with name `Admin`

Copy all files from `app/Http/Controllers/Auth/` to `app/Http/Controllers/Admin/`
  - Change the namespace of the copied files from `Auth` to `Admin`

Create a folder at `resources/views/` with name `admin`

Copy all files and folders from `resources/views/auth/` to `resources/views/admin/`
  - Add `admin.` at the begin of routes

In the `routes/web.php` create the routes below

```php
<?php

Route::GET('admin/home', 'AdminController@index');

Route::group(['namespace' => 'Admin'], function() {

    Route::GET('admin', 'LoginController@showLoginForm')->name('admin.login');
    Route::POST('admin', 'LoginController@login');
    Route::POST('admin-password/email', 'ForgotPasswordController@sendResetLinkEmail')->name('admin.password.email');
    Route::GET('admin-password/reset', 'ForgotPasswordController@showLinkRequestForm')->name('admin.password.request');
    Route::POST('admin-password/reset', 'ResetPasswordController@reset')->name('admin.password.update');
    Route::GET('admin-password/reset/{token}', 'ResetPasswordController@showResetForm')->name('admin.password.reset');
});
```

Create a file at `app/Http/Controllers/` with name `AdminController.php`

Copy the content of `HomeController.php` file at `app/Http/Controllers/` to `AdminController.php`

Change the class name from `HomeController` to `AdminController` in the `AdminController.php` file

Comment the line `$this->middleware('auth');` at `AdminController.php` like example below

```php
<?php

public function __construct()
{
    // $this->middleware('auth');
}
```

Add the code below at `config/auth.php` in the `guards` array, after `web` array

```php
<?php

'admin' => [
    'driver' => 'session',
    'provider' => 'admins',
],
```

Add the code below at `config/auth.php` in the `providers` array, after `users` array

```php
<?php

'admins' => [
    'driver' => 'eloquent',
    'model' => App\Admin::class,
],
```

Add the code below at `config/auth.php` in the `passwords` array, after `users` array

```php
<?php

'admins' => [
    'provider' => 'admins',
    'table' => 'password_resets',
    'expire' => 60,
],
```

Create a file at `app/` with name `Admin.php`

Copy all content from `app/User.php` to `app/Admin.php`

Change the class name from `User` to `Admin` in the `app/Admin.php` file

---
At the `app/Http/Controllers/Admin/ForgotController.php` file make the changes like below

this:
```php
<?php

$this->middleware('guest');
```
to:
```php
<?php

$this->middleware('guest:admin');
```
---

At the `app/Http/Controllers/Admin/LoginController.php` file make the changes like below

this:
```php
<?php

protected $redirectTo = '/home';
```
to :
```php
<?php

protected $redirectTo = 'admin/home';
```
---
and this:
```php
<?php

$this->middleware('guest')->except('logout');
```
to:
```php
<?php

$this->middleware('guest:admin')->except('logout');
```
---

At the `app/Http/Controllers/Admin/RegisterController.php` file make the changes like below

this:
```php
<?php

protected $redirectTo = '/home';
```
to :
```php
<?php

protected $redirectTo = 'admin/home';
```
---
and this:
```php
<?php

$this->middleware('guest');
```
to:
```php
<?php

$this->middleware('guest:admin');
```
---

At the `app/Http/Controllers/Admin/ResetController.php` file make the changes like below

this:
```php
<?php

protected $redirectTo = '/home';
```
to :
```php
<?php

protected $redirectTo = 'admin/home';
```
---
and this:
```php
<?php

$this->middleware('guest');
```
to:
```php
<?php

$this->middleware('guest:admin');
```
---

At the `app/Http/Controllers/Admin/VerificationController.php` file make the changes like below

this:
```php
<?php

protected $redirectTo = '/home';
```
to :
```php
<?php

protected $redirectTo = 'admin/home';
```
---
and this:
```php
<?php

$this->middleware('auth');
```
to:
```php
<?php

$this->middleware('auth:admin');
```
---

Create a migration to `Admin` typing the code below at the terminal

```
php artisan make:migration createAdminTable --create=Admins
```

Copy all the `Schema` from `User` migration to `Admin` migration you just created
- Example of user's Schema:
```php
<?php

$table->bigIncrements('id');
$table->string('name');
$table->string('email')->unique();
$table->timestamp('email_verified_at')->nullable();
$table->string('password');
$table->rememberToken();
$table->timestamps();
```

At the terminal run migrate with the line below

```
php artisan migrate
```

Go to `app/Http/Controllers/Admin/LoginController.php` and you'll see the code `use AuthenticatesUsers;`

If your IDE permite, press Ctrl and click on the `AuthenticatesUsers` or open the `AuthenticatesUsers.php` file at `vendor/laravel/framework/src/Illuminate/Foundation/Auth/`

Copy the function `showLoginForm()` from `AuthenticateUsers.php` to `app/Http/Controllers/Admin/LoginController.php`

```php
<?php

public function showLoginForm()
{
    return view('auth.login');
}
```

In the function `showLoginForm()` inside `app/Http/Controllers/Admin/LoginController.php` make the change below

---
this:
```php
<?php

return view('auth.login');
```
to:
```php
<?php

return view('admin.login');
```
---

Copy the function `guard()` from `AuthenticateUsers.php` to `app/Http/Controllers/Admin/LoginController.php`

```php
<?php

protected function guard()
{
    return Auth::guard('admin');
}
```

Use Auth Facade in `LoginController.php` file

```php
<?php

use Illuminate\Support\Facades\Auth;
```

Uncomment the line `$this->middleware('auth');` at `app/Http/Controller/AdminController.php` file like example below 

```php
<?php

public function __construct()
{
    $this->middleware('auth');
}
```

And still inside `AdminController.php` file make the change below

---
this:
```php
<?php

$this->middleware('auth');
```
to:
```php
<?php

$this->middleware('auth:admin');
```

Open the `RedirectIfAuthenticated.php` file at `app/Http/Middleware/` and make the changes below

---
this:
```php
<?php

public function handle($request, Closure $next, $guard = null)
{
    if (Auth::guard($guard)->check()) {
        return redirect('/home');
    }

    return $next($request);
}
```
to:
```php
<?php

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

At the `Handler.php` file at `app/Exceptions/` use the `AuthenticationException` like below

```php
<?php

use Illuminate\Auth\AuthenticationException;
```

And still inside the `Handler.php` add the function `unauthenticated()` like below

```php
<?php

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

ResetPassword
----------
Go to `app/Http/Controllers/Admin/ForgotPasswordController.php` and you'll see the code `use SendsPasswordResetEmails;`

If your IDE permite, press Ctrl and click on the `SendsPasswordResetEmails` or open the `SendsPasswordResetEmails.php` file at `vendor/laravel/framework/src/Illuminate/Foundation/Auth/`

Copy the function `showLinkRequestForm()` from `SendsPasswordResetEmails.php` to `app/Http/Controllers/Admin/ForgotPasswordController.php`

```php
<?php

public function showLinkRequestForm()
{
    return view('auth.passwords.email');
}
```

In the function `showLinkRequestForm()` inside `app/Http/Controllers/Admin/ForgotPasswordController.php` make the change below

---
this:
```php
<?php

return view('auth.passwords.email');
```
to:
```php
<?php

return view('admin.passwords.email');
```
---

And copy the function `broker()` from `SendsPasswordResetEmails.php` to `app/Http/Controllers/Admin/ForgotPasswordController.php`

```php
<?php

public function broker()
{
    return Password::broker('admins');
}
```

In the function `broker()` inside `app/Http/Controllers/Admin/ForgotPasswordController.php` make the change below

---
this:
```php
<?php

return Password::broker();
```
to:
```php
<?php

return Password::broker('admins');
```
---

At the `app/Http/Controllers/Admin/ForgotPasswordController.php` file use the `Password` Facade like below

```php
<?php

use Illuminate\Support\Facades\Password;
```

At the terminal run the below code to create a new notification

```
php artisan make:notification AdminResetPasswordNotification
```

Go to `vendor/laravel/framework/src/Illuminate/Auth/Notifications/ResetPassword.php` and copy the `return` from `toMail` function to `toMail` function inside the new notification you just created `AdminResetPasswordNotification`

`return` from ResetPassword.php below:
```php
<?php

return (new MailMessage)
    ->subject(Lang::get('Reset Password Notification'))
    ->line(Lang::get('You are receiving this email because we received a password reset request for your account.'))
    ->action(Lang::get('Reset Password'), url(config('app.url').route('password.reset', ['token' => $this->token, 'email' => $notifiable->getEmailForPasswordReset()], false)))
    ->line(Lang::get('This password reset link will expire in :count minutes.', ['count' => config('auth.passwords.'.config('auth.defaults.passwords').'.expire')]))
    ->line(Lang::get('If you did not request a password reset, no further action is required.'));
```

At the `app/Notifications/AdminResetPasswordNotification.php` file use the `Lang` Facade like below

```php
<?php

use Illuminate\Support\Facades\Lang;
```

Inside the `toMail` function from `AdminResetPasswordNotification` change the route like below

this:
```
password.reset
```
to:
```
admin.password.reset
```

Still inside `AdminResetPasswordNotification` at the `construct` put the `$token` varial as parameter and set `$this->token = $token;` like below

```php
<?php

public function __construct($token)
{
    $this->token = $token;
}
```

Go to `vendor/laravel/framework/src/Illuminate/Auth/Passwords/CanResetPassword.php` and copy the function `sendPasswordResetNotification()` to `app/Admin.php`

```php
<?php

public function sendPasswordResetNotification($token)
{
    $this->notify(new ResetPasswordNotification($token));
}
```

At the `app/Admin.php` file use the `AdminResetPasswordNotification` like below

```php
<?php

use App\Notifications\AdminResetPasswordNotification;
```

In the function `sendPasswordResetNotification()` inside `app/Admin.php` make the change below

---
this:
```php
<?php

$this->notify(new ResetPasswordNotification($token));
```
to:
```php
<?php

$this->notify(new AdminResetPasswordNotification($token));
```
---

Go to `app/Http/Controllers/Admin/ResetPasswordController.php` and you'll see the code `use ResetsPasswords;`

If your IDE permite, press Ctrl and click on the `ResetsPasswords` or open the `ResetsPasswords.php` file at `vendor/laravel/framework/src/Illuminate/Foundation/Auth/`

Copy the function `showResetForm()` from `ResetsPasswords.php` to `app/Http/Controllers/Admin/ResetPasswordController.php`

```php
<?php

public function showResetForm(Request $request, $token = null)
{
    return view('auth.passwords.reset')->with(
        ['token' => $token, 'email' => $request->email]
    );
}
```

In the function `showResetForm()` inside `app/Http/Controllers/Admin/ResetPasswordController.php` make the change below

---
this:
```php
<?php

return view('auth.passwords.reset')->with(
```
to:
```php
<?php

return view('admin.passwords.reset')->with(
```
---

Copy the function `broker()` from `ResetsPasswords.php` to `app/Http/Controllers/Admin/ResetPasswordController.php`

```php
<?php

public function broker()
{
    return Password::broker();
}
```

In the function `broker()` inside `app/Http/Controllers/Admin/ResetPasswordController.php` make the change below

---
this:
```php
<?php

return Password::broker();
```
to:
```php
<?php

return Password::broker('admins');
```
---

Copy the function `guard()` from `ResetsPasswords.php` to `app/Http/Controllers/Admin/ResetPasswordController.php`

```php
<?php

protected function guard()
{
    return Auth::guard();
}
```

In the function `broker()` inside `app/Http/Controllers/Admin/ResetPasswordController.php` make the change below

---
this:
```php
<?php

return Auth::guard();
```
to:
```php
<?php

return Auth::guard('admin');
```
---
































