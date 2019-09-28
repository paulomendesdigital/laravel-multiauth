# Laravel Multiauth
Step by step to create a multiauth system with Laravel

At the example below we will create a multiauth with login and admin area

In the terminal run the code below (this needs of composer)

```
composer require laravel/ui --dev
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

Copy the content of file `HomeController.php` at `app/Http/Controllers` to `app/Http/Controllers/AdminController.php`

Change the class name from `HomeController` to `AdminController` in the file `app/Http/Controllers/AdminController.php`

Comment the line `$this->middleware('auth');` at `app/Http/Controller/AdminController.php` like example below

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

Change the class name from `User` to `Admin` in the file `app/Admin.php`

At the `app/Http/Controllers/Admin/ForgotController.php` change de below lines

---
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

---
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

---
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

---
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

---
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


 





































