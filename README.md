# Laravel-Tutorial

1. Install
2. Configuration
3. Concepts du Framework        
    3.1 Le cycle de vie d'une requête         
    3.2 Service Container & Service providers         
4. Les bases     
    4.1 Routes     
    4.2 Middleware    
    4.3 CSRF    
    4.4 Les controllers    
    4.5 Request     
    4.6 Response     
    4.7 Les views     
    
Pour la déscription détaillée des différentes classes du framework
[Documentation API](https://laravel.com/api/5.7/)

## 1. Install

##### installation des packages
```
composer create-project --prefer-dist laravel/laravel project
```

##### lancement du serveur local
```
php artisan serve

// mode maintenance
php artisan down --message="Upgrading Database" --retry=60
php artisan down --allow=127.0.0.1 --allow=192.168.0.0/16
php artisan up

// serveur de catch des dump
php artisan dump-server
```

##### configuration apache en mode rewrite
```
Options +FollowSymLinks -Indexes
RewriteEngine On

RewriteCond %{HTTP:Authorization} .
RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```

## 2. Configuration

L'ensemble des configs dans le repertoire ```config```       
Utilise les variables d'environnement ```.env```     

```php
// Retrouver une variable d'environnement du .env, avec valeur par defaut en deuxieme param
'debug' => env('APP_DEBUG', false),

// accéder à la variable d'environnement
$environment = App::environment();

// lancer des conditions celon les environnements
if (App::environment(['local', 'staging'])) {
    // The environment is either local OR staging...
}

// retrouver et changer les variables de configurations
$value = config('app.timezone');
config(['app.timezone' => 'America/Chicago']);

// mettre en cache les confis pour la production
php artisan config:cache
```

## 3. Concepts du Framework

### 3.1 Le cycle de vie d'une requête

-> envoit de requête HTTP       
-> lancement de l'app et création du service container ``` bootstrap/app.php```          
-> envoit de la requète au Kernel HTTP : ```app/Http/Kernel.php```      
-> le kernel lance les boostrapers dont le service providers : ```app/Providers```    
-> le kernel définis une liste de middleware que les requêtes doivent passer (session, crsf) ```app/Http/Middleware```            
-> la route lance le controller et les routes middlewares ```app/Http/Controllers```          
-> retourne une reponse         

### 3.2 Service Container & Service providers

// Todo, pour l'instant je comprends rien         

## 4. Les bases

### 4.1 Les routes

```php
// les différentes méthodes disponibles
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);

// on peut ajouter plusieurs méthode à la fois
Route::match(['get', 'post'], '/', function () {
    //
});

// toutes les méthodes 
Route::any('/', function () {
    //
});

// les routes basiques avec réponse directe et params (facultatif ou non)
Route::get('/bonjour/{name?}/{age?}', function ($id, $name = "John", $age = null) {
    // créer un lien avec une route nommée
    $url = route("bonjour.nom.age", ["name" => "isabelle", "age" => 30]);
    return "Bonjour ${name}, tu as ${age} an(s) est l'url est ${url}";
    // ou..
    return redirect()->route('home');
})->name("bonjour.nom.age") // donner un nom à la route
  ->where(['id' => '[0-9]+', 'name' => '[a-z]+']); // contraintes regex sur les params
  
// Pour créer des contraintes globales sur les params il faut changer le boot() du RouteServiceProvider
public function boot()
{
    Route::pattern('id', '[0-9]+');
    parent::boot();
}

// connexion entre route et controlleurs avec un middleware
Route::get('/user', 'UserController@index')->middleware('auth');
// si le controlleur et créer avec --resource pour avoir les principales fonctions CRUD
Route::resources([
    'photos' => 'PhotoController',
    'posts' => 'PostController',
    'users' => 'UserController'
]);

// Créer une redirection
Route::redirect('/here', '/there', 301);

// si on ne passe par aucun controller on peut créer directement des routes -> view
Route::view('/welcome', 'welcome', ['name' => 'Taylor']);

// Accéder à la route, au nom et a l'action partout dans l'outil
$route = Route::current();
$name = Route::currentRouteName();
$action = Route::currentRouteAction();
```

Il y a pas mal d'info pour des cas particuliers d'utilisation de route dans la doc       
https://laravel.com/docs/5.7/routing -> (Route Groupe, Route Binding, Route Callback, Rate limit)     

### 4.2 Middleware

Les middlewares sont des mécanismes qui s'enclenchent avant ou après l'envoi de la requete au kernel.     
Ils permettent par exemple de vérifier l'authentification, de binder la session, d'ajouter un CSRF      
Ils sont situés dans ```app/Http/Middldeware```       
On peut créér un middleware avec la commande artisan ```php artisan make:middleware CheckAge```     

__BeforeMiddleware__
```php
namespace App\Http\Middleware;
use Closure;

class BeforeMiddleware
{
    public function handle($request, Closure $next)
    {
        // Perform action

        return $next($request);
    }
}
```

__AfterMiddleware__
```php
namespace App\Http\Middleware;
use Closure;

class AfterMiddleware
{
    public function handle($request, Closure $next)
    {
        $response = $next($request);

        // Perform action

        return $response;
    }
}
```

Ensuite il faut register le middleware dans ```app/Http/Kernel.php```     
On peut les register soit en __Globale__, soit en __Groupe__, soit en __Route Key__

```php
    // Globale
    protected $middleware = [
        \App\Http\Middleware\CheckForMaintenanceMode::class,
        .....
    ];

    // Par Groupe
    protected $middlewareGroups = [
        'web' => [
            \App\Http\Middleware\EncryptCookies::class,
            \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
            ....
        ],

        'api' => [
            'throttle:60,1',
            'bindings',
        ],
    ];

    // Par Route key
    protected $routeMiddleware = [
        'auth' => \App\Http\Middleware\Authenticate::class,
        ...
        'after' => \App\Http\Middleware\AfterMiddleware::class,
    ];

    Route::get('/', function () {
        // possibilité d'utilisé la key ou le full name
    })->middleware('auth', AfterMiddleware::class);
    
    // On peu également passer des paramètres au middleware
    Route::put('post/{id}', function ($id) {
        //
    })->middleware('role:editor');
    
    // on peut aussi ajouter la methode therminate() pour catcher la réponse complète et la request de départ
```

### 4.3 CSRF

```html
<meta name="csrf-token" content="{{ csrf_token() }}">

<form method="POST" action="/profile">
    @csrf
    ...
</form>
```

```javascript
$.ajaxSetup({
    headers: {
        'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content')
    }
});
```

### 4.4 Les controllers

Dans le repertoire  ```app/Http/Controllers```        
Dans les controlleurs l'injection de dépendances est possible avec le TypeHint du service dans le construct ou dans les méthodes.      
      
```php
namespace App\Http\Controllers;

use App\User;
use App\Http\Controllers\Controller;

class UserController extends Controller
{
    public function __construct()
    {
        $this->middleware('auth');

        $this->middleware('log')->only('index');

        $this->middleware('subscribed')->except('store');
    }
    
    public function show($id)
    {
        return view('user.profile', ['user' => User::findOrFail($id)]);
    }
}

// Dans les routes
Route::get('user/{id}', 'UserController@show');
```

```
php artisan make:controller ShowProfile --invokable  // pour les controllers mono action
php artisan make:controller PhotoController --resource --model=Photo // créer un controller avec toutes les ressources CRUD
```

### 4.5 Request

```php
// Les principales méthodes de request
$uri = $request->path();
if ($request->is('admin/*')) {
    //
}

// Without Query String...
$url = $request->url();

// With Query String...
$url = $request->fullUrl();

// Retrouver la méthode
$method = $request->method();

if ($request->isMethod('post')) {
    //
}

// retrouver les inputs
$input = $request->all();
$name = $request->input('name', 'Sally');
$name = $request->input('products.0.name');
$names = $request->input('products.*.name');
$name = $request->name;
$input = $request->only('username', 'password');
$input = $request->except('credit_card');

if ($request->has(['name', 'email'])) {
    //
}
// present et non vide
if ($request->filled('name')) {
    //
}

// retrouver les query
$name = $request->query('name', 'Helen');
$query = $request->query();

// mémoriser toutes les information de la request dans la session
$request->flash();
$request->flashOnly(['username', 'email']);
$request->flashExcept('password');
return redirect('form')->withInput(
    $request->except('password')
);

// pour retrouver les anciennes valeurs
$username = $request->old('username');
<input type="text" name="username" value="{{ old('username') }}">

// les cookies
$value = $request->cookie('name');
return response('Hello World')->cookie(
    'name', 'value', $minutes
);

// les files
$file = $request->file('photo');
$file = $request->photo;
if ($request->hasFile('photo')) {
    //
}
if ($request->file('photo')->isValid()) {
    //
}
$path = $request->photo->path();
$extension = $request->photo->extension()
$path = $request->photo->store('images');
$path = $request->photo->storeAs('images', 'filename.jpg');
```

### 4.6 Response

Il est possible de retourner un string -> html       
Il est possible de retourner un array -> json         
 
```php
// Retourner une réponse classique
return response($content)
            ->header('Content-Type', $type)
            ->header('X-Header-One', 'Header Value')
            or
            ->withHeaders([
                'Content-Type' => $type,
                'X-Header-One' => 'Header Value',
            ])
            
             ->cookie('name', 'value', $minutes);
    
// retourner une redirection
return redirect('home/dashboard');
return back()->withInput();
return redirect()->route('profile', ['id' => 1]);
// prepopulate avec eloquent
return redirect()->route('profile', [$user]);
return redirect()->action(
    'UserController@profile', ['id' => 1]
);
return redirect()->away('https://www.google.com');
return redirect('dashboard')->with('status', 'Profile updated!');

// Retourner un view
return response()
            ->view('hello', $data, 200)
            ->header('Content-Type', $type);
            ->cookie
            
// Retourner un json
return response()->json([
    'name' => 'Abigail',
    'state' => 'CA'
]);


// Retourner un fichier à télécharger
return response()->download($pathToFile);
return response()->download($pathToFile, $name, $headers);
return response()->download($pathToFile)->deleteFileAfterSend();
return response()->streamDownload(function () {
    //
}, $name);

// Retourner l'affichage d'un fichier par exemple PDF/Image
return response()->file($pathToFile, $headers);
```

### 4.7 Les vues

```php
// si la vue est dans un sous repertoire utiliser le . pour créer le path
Route::get('/', function () {
    return view('pages.name', ['name' => 'James'])->with('lastname', 'Victoria');
});
// on peut utiliser la fonction comptact() pour créer facilement le [] de data

// Déterminer si une vue existe
use Illuminate\Support\Facades\View;

if (View::exists('pages.name')) {
    //
}

// On peut renvoyer la première vue dans une liste de vues définis
return view()->first(['pages.name', 'name'], $data);

// Pour créer des urls
url()->current()->full()->previous()
route('post.show', ['post' => 1]);
$url = action([HomeController::class, 'index']);
```

```html
<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

### Blade : moteur de template

https://laravel.com/docs/5.7/blade   

```html
// Création de commentaire en blade
{{-- This comment will not be present in the rendered HTML --}}
```html

#### Héritage

Les tags __@yield @section @extends @include @component @parent__ utilisé pour l'héritage

```html
<!-- Stored in resources/views/layouts/app.blade.php -->
<html>
    <head>
        <title>App Name - @yield('title')</title>
    </head>
    <body>
        @section('sidebar')
            This is the master sidebar.
        @show
        
        @component('alert')
            <strong>Whoops!</strong> Something went wrong!
        @endcomponent
        <div class="container">
            @yield('content')
        </div>
    </body>
</html>

<!-- Stored in resources/views/child.blade.php -->
@extends('layouts.app')

@section('title', 'Page Title')

@section('sidebar')
    @parent

    <p>This is appended to the master sidebar.</p>
@endsection

@section('content')
    <p>This is my body content.</p>
@endsection
```

#### Conditions

Les tags __@yield @if @elseif @else @unless @isset @empty__ utilisés pour les conditions      
Pour les droits __@auth__ et __@geth__     
Pour la vérification de template __@hasSection__

```html
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif

@unless (Auth::check())
    You are not signed in.
@endunless

@isset($records)
    // $records is defined and is not null...
@endisset

@empty($records)
    // $records is "empty"...
@endempty

// Pour checker les authentifcations
@auth('admin')
    // The user is authenticated...
@endauth

@guest('admin')
    // The user is not authenticated...
@endguest
```

#### Les boucles

Les tags __@for @foreach @forelse @empty @while @continue @break__ utilisés pour les boucles      
Il y a un objet __$loop__ accessibe dans les boucles       
- __$loop->index__	The index of the current loop iteration (starts at 0).       
- __$loop->iteration__	The current loop iteration (starts at 1).       
- __$loop->remaining__	The iterations remaining in the loop.       
- __$loop->count__	The total number of items in the array being iterated.       
- __$loop->first__	Whether this is the first iteration through the loop.       
- __$loop->last__	Whether this is the last iteration through the loop.       
- __$loop->depth__	The nesting level of the current loop.       
- __$loop->parent__	When in a nested loop, the parent's loop variable.       
       
```html
@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
                      
    @if ($loop->first)
        This is the first iteration.
    @endif

    @if ($loop->last)
        This is the last iteration.
    @endif
    
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>

    @break($user->number == 5)
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile
```

## Databases

### Migrations

```
// Créer une migration
php artisan make:migration create_users_table --create=users
php artisan make:migration add_votes_to_users
_table --table=users

// Appliquer une migration
php artisan migrate --force

// Revenir en arrière ou reset
php artisan migrate:rollback --step=5
php artisan migrate:reset

// rebuild la database
php artisan migrate:refresh
```

### Schema, Table, Column, Index

```php
Schema::create('users', function (Blueprint $table) {
    $table->increments('id');
});

if (Schema::hasTable('users')) {
    //
}

if (Schema::hasColumn('users', 'email')) {
    //
}

Schema::rename($from, $to);

Schema::drop('users');
Schema::dropIfExists('users');
```

## Modèle


