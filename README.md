# JWT with Laravel Sanctum



## 1st Step.
###### [Refrence Laravel Sanctum](https://laravel.com/docs/8.x/sanctum)
> Install Laravel Sanctum `composer require laravel/sanctum`

> Then run the command `php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"`

It will create a file in config/sanctum.php

## 2nd Step.
> Run Migration `php artisan migrate`

> Run the command `php artisan make:controller AuthController`

> Paste the codes in User Model `use HasApiTokens` also add this on before class define `use Laravel\Sanctum\HasApiTokens;`

![image](https://user-images.githubusercontent.com/54832640/177614692-ff29e4c4-6f86-4d72-ab06-08c68fd15511.png)

It should be look like this.

## 3rd Step.
> Paste the below code in **AuthController.php**
Paset these lines before defining **class**



```
use App\Models\User;

use Illuminate\Http\Request;
use Illuminate\Http\Response;
use Illuminate\Support\Facades\Cookie;
use Illuminate\Support\Facades\Validator;
use Illuminate\Support\Facades\Hash;

class AuthController extends Controller
{
    public function register(Request $request)
    {
        $fields = Validator::make($request->all(), [
            'name' => 'required|string',
            'email' => 'required|email|unique:users,email',
            'password' => 'required|string|confirmed'
        ]);
        if ($fields->fails()) {
            return response([
                'message' => $fields->errors(),
            ], Response::HTTP_UNAUTHORIZED);
        }
        $user = User::create([
            'name' => $request->name,
            'email' => $request->email,
            'password' => bcrypt($request->password)
        ]);

        $token = $user->createToken('ims_token')->plainTextToken;

        $cookie = cookie('jwt', $token, 60 * 24); // 1 Day.
        return response([
            'message' => 'success',
            'user' => $user,
            'token' => $token
        ])->withCookie($cookie);
    }

    public function login(Request $request)
    {
        $fields = $request->validate([
            'email' => 'required|email',
            'password' => 'required|string'
        ]);
        $user = User::where('email', $fields['email'])->first();

        if (!$user || !Hash::check($fields['password'], $user->password)) {
            return response([
                'message' => 'Bad Credentials'
            ], Response::HTTP_UNAUTHORIZED);
        }

        $token = $user->createToken('ims_token')->plainTextToken;
        $cookie = cookie('jwt', $token, 60 * 24); // 1 Day.

        $response = [
            'message' => 'success',
            'user' => $user,
            'token' => $token
        ];
        return response($response, Response::HTTP_CREATED)->withCookie($cookie);
    }

    public function logout()
    {
        auth()->user()->tokens()->delete();

        $cookie = Cookie::forget('jwt');

        return response([
            'message' => 'Logged Out'
        ])->withCookie($cookie);
    }
}
```

## 4th step.
> Open config/cors.php and find `supports_credentials' => false` and make this **false** to **true**.

## 5th step.
> Open app/Http/Middleware and paste the below codes after **redirectTo** function.
```
use Closure;
public function handle($request, Closure $next, ...$guards)
{
    if ($jwt = $request->cookie('jwt')) {
        $request->headers->set('Authorization', 'Bearer ' . $jwt);
    }
    $this->authenticate($request, $guards);

    return $next($request);
}
```
## 6th step.
> Add the below codes in api.php.

```
Route::post('/register', [App\Http\Controllers\AuthController::class, 'register']);
Route::post('/login', [App\Http\Controllers\AuthController::class, 'login']);

Route::group(['middleware' => ['auth:sanctum']], function () {
    Route::get('/user', function () {
            return App\Models\User::all();
    });

    Route::post('/logout', [App\Http\Controllers\AuthController::class, 'logout']);
});
```
Try with Postman.
