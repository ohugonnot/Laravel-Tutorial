# Laravel-Tutorial

### Les plugins interessants
https://github.com/fruitcake/laravel-cors - ajouter les cors pour les apis       
https://github.com/barryvdh/laravel-ide-helper - helper pour les ide et phpstorm      
https://github.com/barryvdh/laravel-debugbar - toolbar symfony like       
https://github.com/tymondesigns/jwt-auth - auth JWT pour les apis      
https://github.com/Intervention/image - gestion des images      
https://github.com/Maatwebsite/Laravel-Excel - lib excel     
      
1. Install
2. Configuration
3. Concepts du Framework        
    3.1 Le cycle de vie d'une requête         
    3.2 Service Container & Service providers +contracts & facades        
4. Les bases     
    4.1 Routes     
    4.2 Middleware    
    4.3 CSRF    
    4.4 Les controllers    
    4.5 Request     
    4.6 Response     
    4.7 Les views    
    4.8 Session    
    4.9 Validation    
    4.10 Errors handling et logs   
5. Front-end       
    5.1 Blade      
    5.2 Localisation   
    5.3 Scaffolding & Assets     
6. Sécurité      
    6.1 Authentication      
    6.2 Authorization      
7. Aller plus loin      
    7.1 Collection      
    7.2 Event   
    
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

### 3.2 Service Container & Service providers + contracts & facades

__Le Service Container__ est un outil qui permet de gérer les dépendances et les injections dans nos classes.      
Une des spécificités de ce container est sa capacité à résoudre les classes de manière automatique. En effet, si aucune fonction n'a été enregistrée pour définir notre dépendance le framework tentera de construire l'objet tout seul en y injectant les dépendance de manière automatique si le constructeur en demande.   

```php
$this->app->bind('HelpSpot\API', function ($app) {
    return new HelpSpot\API($app->make('HttpClient'));
});
$this->app->singleton('HelpSpot\API', function ($app) {
    return new HelpSpot\API($app->make('HttpClient'));
});
$this->app->instance('HelpSpot\API', $api);
$this->app->when('App\Http\Controllers\UserController')
          ->needs('$variableName')
          ->give($value);
// binder avec une interface
$this->app->bind(
    'App\Contracts\EventPusher',
    'App\Services\RedisEventPusher'
);

$api = $this->app->make('HelpSpot\API');
$api = resolve('HelpSpot\API');
$api = $this->app->makeWith('HelpSpot\API', ['id' => 1]);

// les event container
$this->app->resolving(function ($object, $app) {
    // Called when container resolves object of any type...
});

$this->app->resolving(HelpSpot\API::class, function ($api, $app) {
    // Called when container resolves objects of type "HelpSpot\API"...
});
```

__Les services provider__ permettent d'enregistrer de nouveaux éléments dans notre service container mais aussi d'ajouter une logique au démarrage (boot()) de notre application.        

```php
    public function register()
    {
        $this->app->singleton(Connection::class, function ($app) {
            return new Connection(config('riak'));
        });
    }
    
        /**
     * All of the container bindings that should be registered.
     *
     * @var array
     */
    public $bindings = [
        ServerProvider::class => DigitalOceanServerProvider::class,
    ];

    /**
     * All of the container singletons that should be registered.
     *
     * @var array
     */
    public $singletons = [
        DowntimeNotifier::class => PingdomDowntimeNotifier::class,
    ];
    
    // fonction boot après que tous les service soit instancier possible d'injection
        public function boot()
    {
        view()->composer('view', function () {
            //
        });
    }
    
    // possibilité de defer un provider
    protected $defer = true;
```
    
Les Facades permettent d'appeler le Service Container à travers une classe static.      
Les contracts sont des interface réutilisable      


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
Route::apiRessource($uri, $callback);
Route::ressources($uri, $callback);

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

### 4.8 Sessions

```php
// Dans la request
    $value = $request->session()->get('key', 'default'); // possibilité de passer une closure en param par defaut
    $data = $request->session()->all();
    $request->session()->put('key', 'value');
    // on peux ajouter dans un array
    $request->session()->push('user.teams', 'developers');
    // selectionner et supprimer de la session en meme temps
    $value = $request->session()->pull('key', 'default');
    
    // Les messages flash
    $request->session()->flash('status', 'Task was successful!');
    $request->session()->reflash();  // permet de garder les messages flash d'une requete à l'autre
    $request->session()->keep(['username', 'email'])  // permet de garder uniquement certain message
    
    // supprimer completement de la session
    $request->session()->forget('key')
    $request->session()->flush();
    
    if ($request->session()->has('users')) {
    // present et non null
    }
    if ($request->session()->exists('users')) {
    // present peu importe la valeur
    }
    
// Globalement
    // Specifying a default value...
    $value = session('key', 'default');

    // Store a piece of data in the session...
    session(['key' => 'value'])
```

### 4.9 Validation

```php
// Dans le controller on catch la request
    $validatedData = $request->validate([
         // bail permet d'arrêter la validation à l'erreur en cours
        'title' => 'bail|required|unique:posts|max:255',
        'body' => 'required',
        // possibilité d'utiliser des attribut nested
        'author.name' => 'required',
        'author.description' => 'required',
        'publish_at' => 'nullable|date',  // nullable pour optionnelle, car middleware ConvertEmptyStringsToNull 
    ]);
    
 // Pour les validations plus complète
 php artisan make:request StoreBlogPost
 // Cela créer un extend de request avec des rules
public function rules()
{
    return [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ];
}
// il suffit ensuite de type hinter la request dans le controller
public function store(StoreBlogPost $request)
{
    $validated = $request->validated();
}
// on peut également modifier le wording
public function messages()
{
    return [
        'title.required' => 'A title is required',
        'body.required'  => 'A message is required',
    ];
}
// et vérifier qu'on a l'authorisation d'acceder à cette request
public function authorize()
{
    $comment = Comment::find($this->route('comment'));
    return $comment && $this->user()->can('update', $comment);
}
```
Si la validation échoue on retourne a la page précedente avec variable errors

```html
@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
```
Pour créer des Validation sur mesure et connaitre toutes les validation possible sur laravel     
https://laravel.com/docs/5.7/validation#available-validation-rules
https://laravel.com/docs/5.7/validation#custom-validation-rules

### 4.10 Les logs et les erros handling

```php
namespace App\Exceptions;

use Exception;

class RenderException extends Exception
{
    public function report()
    {
        // gére la logic de l'exeption, log ou envoit a service externe
    }
    
    // on peu également utilisé le report helper pour reporter un erreur sans forcement arreter l'application
    try {
        // Validate the value...
    } catch (Exception $e) {
        report($e);
        return false;
    }

    public function render($request)
    {
        // gère l'affichage de l'exeption
        return response(...);
    }
    
    // on peut également utiliser le helper abot
    abort(404, "Message");
}
```

Il existe plusieurs drivers de log notament pour slack

```php
Log::emergency($message);
Log::alert($message);
Log::critical($message);
Log::error($message);
Log::warning($message);
Log::notice($message);
Log::info($message);
Log::debug($message);

Log::info('User failed to login.', ['id' => $user->id]);
Log::channel('slack')->info('Something happened!');
Log::stack(['single', 'slack'])->info('Something happened!');
```

## 5 Front-end

### 5.1 Blade : moteur de template

https://laravel.com/docs/5.7/blade   

```html
// Création de commentaire en blade
{{-- This comment will not be present in the rendered HTML --}}
```

#### Héritage

Les tags __@yield @section @extends @include @component @parent__ utilisé pour l'héritage
__@yield__ : pour remplacer le contenu dans une section ou une balise
__@section__ : créer une section

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

### 5.2 Localisation

Les traductions sont stockées dans le repertoire ```resources/lang```     
Les fichiers retournent des array key => string

```php
return [
    'welcome' => 'Welcome to our application'
];

// On peu changer la locale
Route::get('welcome/{locale}', function ($locale) {
    App::setLocale($locale);

    //
});

// Déterminer quelle est la locale
$locale = App::getLocale();
if (App::isLocale('en')) {
    //
}

'welcome' => 'Welcome, :name',
{{ __('messages.welcome', ['name' => 'dayle']); }}
@lang('messages.welcome')

// pour les pluriels
'apples' => 'There is one apple|There are many apples',
'apples' => '{0} There are none|[1,19] There are some|[20,*] There are many',
{{ trans_choice('messages.apples', 10) }}

```

### 5.3 Scaffolding & assets

De base laravel céer une préconfiguration qui embarque Boostrap et Vuejs    
On peut supprimer la préconfig ```php artisan preset none```     

```
npm install  ou  yarn install
npm run dev/production   ou   yarn run dev/production
npm run watch
```

Exemple de webpack.mix      

```javascript
const mix = require('laravel-mix');

mix.js('resources/js/app.js', 'public/js').extract(['vue', "bootstrap","jquery","popper.js","lodash","axios"])
mix.sass('resources/sass/app.scss', 'public/css');

if (mix.inProduction()) {
    mix.version();
} else {
    mix.sourceMaps();
}
```

## 6 Sécurité

### 6.1 Authentification

```
// créer les controllers, les layouts pour login, register, reset password ect dans resources/views/auth
php artisan make:auth
// Pour gérer les accès utilisateurs éplucher les controlleurs
LoginController, RegisterController, ResetPasswordController, and VerificationController
// ainsi que les middleware
Authenticate
RedirectIfAuthenticated 
// Il est possible de changer la redirection post connexion soit en changeant le param soit en créer une méthode
protected $redirectTo = '/';
protected function redirectTo()
{
    return '/path';
}

// Pour modifier le param qui sert d'username il faut modifier le controller LoginController
public function username()
{
    return 'username';
}
// Pour pouvoir ce connecter par email ou username
public function username()
{
   $login = request()->input('login');
   $field = filter_var($login, FILTER_VALIDATE_EMAIL) ? 'email' : 'username';
   request()->merge([$field => $login]);
   return $field;
}

// Get the currently authenticated user...
$user = Auth::user();

// Get the currently authenticated user's ID...
$id = Auth::id();

// l'user est aussi accessible directement depuis la requete
$request->user() 

// Vérifier si un user est authentifié
if (Auth::check()) {
    // The user is logged in...
}

// Pour proteger les routes
Route::get('profile', function () {

})->middleware('auth');

// On peut également proteger tout un controller dans le construct avec le guard
public function __construct()
{
    $this->middleware('auth:api');
}

// logout
Auth::logout();

// loguer une instance user déjà existante
Auth::login($user);
Auth::loginUsingId(1);

// Login and "remember" the given user...
Auth::login($user, true);
Auth::loginUsingId(1, true);

// Les listeners d'authentification user
protected $listen = [
    'Illuminate\Auth\Events\Registered' => [
        'App\Listeners\LogRegisteredUser',
    ],

    'Illuminate\Auth\Events\Attempting' => [
        'App\Listeners\LogAuthenticationAttempt',
    ],

    'Illuminate\Auth\Events\Authenticated' => [
        'App\Listeners\LogAuthenticated',
    ],

    'Illuminate\Auth\Events\Login' => [
        'App\Listeners\LogSuccessfulLogin',
    ],

    'Illuminate\Auth\Events\Failed' => [
        'App\Listeners\LogFailedLogin',
    ],

    'Illuminate\Auth\Events\Logout' => [
        'App\Listeners\LogSuccessfulLogout',
    ],

    'Illuminate\Auth\Events\Lockout' => [
        'App\Listeners\LogLockout',
    ],

    'Illuminate\Auth\Events\PasswordReset' => [
        'App\Listeners\LogPasswordReset',
    ],
];

```

### 6.1 Authorization

#### Les Gates

```php
// App\Providers\AuthServiceProvider
public function boot()
{
    $this->registerPolicies();

    Gate::define('update-post', function ($user, $post) {
        return $user->id == $post->user_id;
    });
    
    // ou
    Gate::define('update-post', 'App\Policies\PostPolicy@update');
    Gate::resource('posts', 'App\Policies\PostPolicy');
    // revient au même que 
    Gate::define('posts.view', 'App\Policies\PostPolicy@view');
    Gate::define('posts.create', 'App\Policies\PostPolicy@create');
    Gate::define('posts.update', 'App\Policies\PostPolicy@update');
    Gate::define('posts.delete', 'App\Policies\PostPolicy@delete');
    
    Gate::resource('posts', 'PostPolicy', [
        'image' => 'updateImage',
        'photo' => 'updatePhoto',
    ]);
    
    // Utiliser les gates, le user est automatiquement injecté
    if (Gate::allows('update-post', $post)) {
        // The current user can update the post...
    }

    if (Gate::denies('update-post', $post)) {
        // The current user can't update the post...
    }
    
    // si on veut tester pour un $user définis
    if (Gate::forUser($user)->allows('update-post', $post)) {
    // The user can update the post...
    }

    if (Gate::forUser($user)->denies('update-post', $post)) {
        // The user can't update the post...
    }
    
    // Pour bypasser les gates
    Gate::before(function ($user, $ability) {
        if ($user->isSuperAdmin()) {
            return true;
        }
    });
}
```

#### Les policies

```php
php artisan make:policy PostPolicy --model=Post

// register la policy dans AuthServiceProvider 
    protected $policies = [
        Post::class => PostPolicy::class,
    ];

// Le policies sont des classes qui gèrent les droits
// il est possible de faire passer les policies au guest en typehint le ?User
// on peut également utiliser la fonction before pour bypassé
namespace App\Policies;

use App\User;
use App\Post;

class PostPolicy
{
    /**
     * Determine if the given post can be updated by the user.
     *
     * @param  \App\User  $user
     * @param  \App\Post  $post
     * @return bool
     */
    public function update(?User $user, Post $post)
    {
        return $user->id === $post->user_id;
    }
    
    public function before($user, $ability)
    {
        if ($user->isSuperAdmin()) {
            return true;
        }
    }
    
    // comment utiliser les policies
    if ($user->can('update', $post)) {
        //
    }
    // pour les create quand il n'y a pas de model instancier
    if ($user->can('create', Post::class)) {
    // Executes the "create" method on the relevant policy...
    }
    
    // on peut également les passer dans les middleware directement dans les routes
    Route::put('/post/{post}', function (Post $post) {
        // The current user may update the post...
    })->middleware('can:update,post');
    }
    
    // on peut également les utiliser dans les controlleurs grace au helpers
    $this->authorize('update', $post);
    
    // et enfin dans les blades
    @can('update', $post)
        <!-- The Current User Can Update The Post -->
    @elsecan('create', App\Post::class)
        <!-- The Current User Can Create New Post -->
    @endcan

    @cannot('update', $post)
        <!-- The Current User Can't Update The Post -->
    @elsecannot('create', App\Post::class)
        <!-- The Current User Can't Create New Post -->
    @endcannot
```

### 7 Aller plus loin

### 7.1 Collection

https://laravel.com/docs/5.7/collections les differents fonctions disponible dans les collections.       
Le retour de Eloquent sont des collections      

```php
// Création d'une collection
$collection = collect(['taylor', 'abigail', null])->map(function ($name) {
    return strtoupper($name);
})
->reject(function ($name) {
    return empty($name);
});

// Etendre la collection
use Illuminate\Support\Str;

Collection::macro('toUpper', function () {
    return $this->map(function ($value) {
        return Str::upper($value);
    });
});

$collection = collect(['first', 'second']);

$upper = $collection->toUpper();

// ['FIRST', 'SECOND']

// On peut créer des raccourci d'action sur les collections
// pour les fonctions suivante average, avg, contains, each, every, filter, first, flatMap, groupBy,  keyBy, map, max, min, partition, reject, sortBy, sortByDesc, sum, and unique.

$users = User::where('votes', '>', 500)->get();

$users->each->markAsVip();
Likewise, we can use the sum higher order message to gather the total number of "votes" for a collection of users:

$users = User::where('group', 'Development')->get();
return $users->sum->votes;
```

### 7.2 Event

Les events sont stockés dans ```app/events```    
Les listeners sont stockés dans ```app/listeners```     

```php

// Pour register les events il faut les ajouter dans EventServiceProvider 
protected $listen = [
    'App\Events\OrderShipped' => [
        'App\Listeners\SendShipmentNotification',
    ],
];

// Pour créer automatiquement les event et les listener on peut inscrire les event et les listeners
// dans le EventServiceProvider et lancer la commande pour génerer les fichiers
php artisan event:generate 

// On peut aussi register les events avec des closure dans EventServiceProvider
public function boot()
{
    parent::boot();

    Event::listen('event.name', function ($foo, $bar) {
        //
    });
    
    Event::listen('event.*', function ($eventName, array $data) {
    //
    });
}

// Une classe event contient les data et l'objet event
namespace App\Events;

use App\Order;
use Illuminate\Queue\SerializesModels;

class OrderShipped
{
    // permet de sereializer les objets eloquent
    use SerializesModels;

    public $order;

    /**
     * Create a new event instance.
     *
     * @param  \App\Order  $order
     * @return void
     */
    public function __construct(Order $order)
    {
        $this->order = $order;
    }
}

// Une class  listener
namespace App\Listeners;

use App\Events\OrderShipped;

class SendShipmentNotification
{
    // possibilité d'injection depuis le service container
    public function __construct()
    {
        //
    }

    // la fonction handle recoit l'event typ-hinté en argument
    public function handle(OrderShipped $event)
    {
        // on peut stopper la propagation de l'event aux autres listeners en retournant false
        return false;
    }
}


// Pour créer des event queued il faut implémenter le listener avec ShouldQueue
namespace App\Listeners;

use App\Events\OrderShipped;
use Illuminate\Contracts\Queue\ShouldQueue;

class SendShipmentNotification implements ShouldQueue
{
    // on peut customier la connexion de la queued
    public $connection = 'sqs';

    // et le nom de la queued list
    public $queue = 'listeners';
    
    public function handle(OrderShipped $event)
    {
        //
    }
    
    // si la queued fail on peut catcher l'erreur dans la foncton failed
    public function failed(OrderShipped $event, $exception)
    {
        //
    }
    
    
}

// Dispatchter un event avec la fonction event()
   event(new OrderShipped($order));

// Créer un EventSubscriber qui est une factorisation de plusieurs listeners
namespace App\Listeners;

class UserEventSubscriber
{
    /**
     * Handle user login events.
     */
    public function onUserLogin($event) {}

    /**
     * Handle user logout events.
     */
    public function onUserLogout($event) {}

    /**
     * Register the listeners for the subscriber.
     *
     * @param  \Illuminate\Events\Dispatcher  $events
     */
    public function subscribe($events)
    {
        $events->listen(
            'Illuminate\Auth\Events\Login',
            'App\Listeners\UserEventSubscriber@onUserLogin'
        );

        $events->listen(
            'Illuminate\Auth\Events\Logout',
            'App\Listeners\UserEventSubscriber@onUserLogout'
        );
    }
}

// Pour register un EventSubscriber dans le EventServiceProvider il faut utiliser la proprieté subscribe
namespace App\Providers;

use Illuminate\Foundation\Support\Providers\EventServiceProvider as ServiceProvider;

class EventServiceProvider extends ServiceProvider
{
    // Register les event et les eventListener
    protected $listen = [
        //
    ];

    // Register les eventSubscriber
    protected $subscribe = [
        'App\Listeners\UserEventSubscriber',
    ];
}


```

## 8 Databases

### 8.1 Database query

Il y a bcp de fonction pour créer les query voir https://laravel.com/docs/5.7/queries      


```php
DB::select('select * from users where id = :id', ['id' => 1]);
DB::insert('insert into users (id, name) values (?, ?)', [1, 'Dayle']);
$affected = DB::update('update users set votes = 100 where name = ?', ['John']);
$deleted = DB::delete('delete from users');
DB::statement('drop table users');

// transaction
DB::transaction(function () {
    DB::table('users')->update(['votes' => 1]);

    DB::table('posts')->delete();
}, 5);

// retourne une collection
$users = DB::table('users')->get();
$user = DB::table('users')->where('name', 'John')->first();
$email = DB::table('users')->where('name', 'John')->value('email');
$titles = DB::table('roles')->pluck('title');
$roles = DB::table('roles')->pluck('title', 'name');

foreach ($roles as $name => $title) {
    echo $title;
}
DB::table('users')->orderBy('id')->chunk(100, function ($users) {
    foreach ($users as $user) {
        //
    }
    return false;
});

$users = DB::table('users')->count();
$users = DB::table('users')->latest()->get();
$price = DB::table('orders')->max('price');
$price = DB::table('orders')
                ->where('finalized', 1)
                ->avg('price');
                
return DB::table('orders')->where('finalized', 1)->exists();
return DB::table('orders')->where('finalized', 1)->doesntExist();

$users = DB::table('users')->select('name', 'email as user_email')->get();
$users = DB::table('users')->distinct()->get();
$query = DB::table('users')->select('name');
$users = $query->addSelect('age')->get();
$users = DB::table('users')
                     ->select(DB::raw('count(*) as user_count, status'))
                     ->where('status', '<>', 1)
                     ->groupBy('status')
                     ->get();
$orders = DB::table('orders')
                ->selectRaw('price * ? as price_with_tax', [1.0825])
                ->get();
$orders = DB::table('orders')
                ->whereRaw('price > IF(state = "TX", ?, 100)', [200])
                ->get();
$orders = DB::table('orders')
                ->select('department', DB::raw('SUM(price) as total_sales'))
                ->groupBy('department')
                ->havingRaw('SUM(price) > ?', [2500])
                ->get();
$orders = DB::table('orders')
                ->orderByRaw('updated_at - created_at DESC')
                ->get();
                
 // Les jointures               
$users = DB::table('users')
            ->join('contacts', 'users.id', '=', 'contacts.user_id')
            ->join('orders', 'users.id', '=', 'orders.user_id')
            ->select('users.*', 'contacts.phone', 'orders.price')
            ->get();
$users = DB::table('users')
            ->leftJoin('posts', 'users.id', '=', 'posts.user_id')
            ->get();
DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')->orOn(...);
        })
        ->get();
DB::table('users')
        ->join('contacts', function ($join) {
            $join->on('users.id', '=', 'contacts.user_id')
                 ->where('contacts.user_id', '>', 5);
        })
        ->get();
$latestPosts = DB::table('posts')
                   ->select('user_id', DB::raw('MAX(created_at) as last_post_created_at'))
                   ->where('is_published', true)
                   ->groupBy('user_id');

$users = DB::table('users')
        ->joinSub($latestPosts, 'latest_posts', function ($join) {
            $join->on('users.id', '=', 'latest_posts.user_id');
        })->get();
$first = DB::table('users')
            ->whereNull('first_name');
$users = DB::table('users')
            ->whereNull('last_name')
            ->union($first)
            ->get();
            
// Les clauses where
$users = DB::table('users')
                ->where('votes', '>=', 100)
                ->get();

$users = DB::table('users')
                ->where('votes', '<>', 100)
                ->get();

$users = DB::table('users')
                ->where('name', 'like', 'T%')
                ->get();
$users = DB::table('users')->where([
    ['status', '=', '1'],
    ['subscribed', '<>', '1'],
])->get();
$users = DB::table('users')
                    ->where('votes', '>', 100)
                    ->orWhere('name', 'John')
                    ->get();
```

### 8.2 Migrations

```
// Créer une migration
php artisan make:migration create_users_table --create=users
php artisan make:migration add_votes_to_users_table --table=users

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

```php
php artisan make:migration create_users_table --create=users
php artisan make:migration add_votes_to_users_table --table=users

// Creation d'un modele ressource controller + factory + migration
php artisan make:model Model\Category -mrf

// Relation dans les modèles
// Les id dans la migration qui permette les relation doivent être unsigned() car les id auto increment le sont de base
$table->integer('user_id')->unsigned();

// One
public function user()
{
    return $this->belongsTo(User::class);
}
// Many
public function users()
{
    return $this->hasMany(User::class);
}
```

## Factory et Seed

```php
// Utilisation de faker pour créer rapidement des données dans la base de données
$faker->sentence->word->text;
'user_id'=> function() {
    return User::all()->random();
 }
  
// Seeder dans la base
class DatabaseSeeder {
    public function run()
    {
        factory(User::class,10)->create();
        factory(Reply::class,10)->create()->each(function($reply){
            return $reply->like()->save(factory(like::class)->make())
        });
    }
}

// Utiliser les seter pour les param qui recquiert des actions ou edit ou create
User::create($request->all());

// In User Model
public setPasswordAttribute($password)
{
    $this->attributes['passworsd'] = bcrypt($password);
}

php artisan db:seed

// Création de ressource pour le model afin de transformer toArray()
php artisan make:resource UserResource
return new UserResource($user);
return UserResource::collection($users);
```

## Sécurité

```php
// Suivre les recommandations https://jwt-auth.readthedocs.io/en/develop
// Regarder le tuto https://www.udemy.com/course/real-time-single-page-forum-app-with-pusher-laravel-vuejs/learn/lecture/9646566

// Pour avoir une meilleurs modularité et des logs consistants remplacer le middleware api:auth
// https://www.udemy.com/course/real-time-single-page-forum-app-with-pusher-laravel-vuejs/learn/lecture/9627424
php artisan make:middleware JWT
public function handle(Request $request, Closure $next)
{
    JWTAuth::parseToken()->authenticate();
    return $next($request);
}
// Ensuite l'ajouter dans le routeMiddleware = [] du kernel
// On peu aussi améliorer le handler des exeptions 
// https://www.udemy.com/course/real-time-single-page-forum-app-with-pusher-laravel-vuejs/learn/lecture/9627426

// Pour le localStorage du token voir https://www.udemy.com/course/laravel-vuejs-restful-api-course/learn/lecture/19820806
// Egalement https://www.udemy.com/course/real-time-single-page-forum-app-with-pusher-laravel-vuejs/learn/lecture/9627456
```

## Vue

```php

// Possibilite d'installer Vuetify
// https://jigeshraval.com/fr/blog/vue-js/laravel-vue-js-admin-using-vuetify
npm install vuetify
npm install sass sass-loader fibers deepmerge -D

// Install vue-router
npm install vue-router
```
