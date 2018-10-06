# Laravel-Tutorial

Pour la déscriptions détailler des différentes classes du framework
[Documentation API](https://laravel.com/api/5.7/)

## Install

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

RewriteCond %{REQUEST_FILENAME} !-d
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [L]
```

## Configuration

L'ensemble des configs dans le repertoire ```config```       
Utilise les variables d'environnement ```.env```     

```php
// Retrouver une variable d'environnement du .env, avec valeur par defaut en deuxieme param
'debug' => env('APP_DEBUG', false),

// accéder à la variable d'environnement
$environment = App::environment();

// retrouver et changer les variables de configurations
$value = config('app.timezone');
config(['app.timezone' => 'America/Chicago']);

// mettre en cache les confis pour la production
php artisan config:cache
```

## Concepts du Framework

### Le cycle de vie d'une requête

-> envoit de requête HTTP
-> lancement de l'app et création du service container      
-> envoit de la requète au Kernel HTTP : ```app/Http/Kernel.php```    
-> le kernel lance les boostrapers dont le service providers : ```app/Providers```  
-> le kernel définis une liste de middleware que les requêtes doivent passer (session, crsf)     
-> la route lance le controller et les routes middlewares    
-> retourne une reponse      

### Service Container



## Les routes

```php
// les routes basique avec réponse directe
Route::get('/bonjour/{name}/{age}', function ($name, $age) {
    $url = route("bonjour.nom.age", ["name" => "isabelle", "age" => 30]);
    return "Bonjour ${name}, tu as ${age} an(s) est l'url est ${url}";
})->name("bonjour.nom.age");

// connexion entre route et controlleurs avec un middleware
Route::get('/user', 'UserController@index')->middleware('auth');;
// si le controlleur et créer avec --resource pour avoir les principales fonctions CRUD
Route::resources([
    'photos' => 'PhotoController',
    'posts' => 'PostController',
    'users' => UserController
]);
```

## Les vues

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
```

```html
<!-- View stored in resources/views/pages/name.blade.php -->
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

## Les controllers

Créer un controlleur avec les fonctions de base en CRUD     
```
php artisan make:controller --resource TestController --model=Test
```

Dans les controlleurs l'injection de dépendances est possible avec le TypeHint du service dans le construct ou dans les méthodes.

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


