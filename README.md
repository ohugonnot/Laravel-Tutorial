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

## Les routes

```php
// les routes basique avec réponse directe
Route::get('/bonjour/{name}/{age}', function ($name, $age) {
    $url = route("bonjour.nom.age", ["name" => "isabelle", "age" => 30]);
    return "Bonjour ${name}, tu as ${age} an(s) est l'url est ${url}";
})->name("bonjour.nom.age");

// connexion entre route et controlleurs
Route::get('/user', 'UserController@index');
// si le controlleur et créer avec --resource
Route::resource('/user', 'UserController');
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

## Les controllers

Créer un controlleur avec les fonctions de base
```
php artisan make:controller --resource TestController
```

