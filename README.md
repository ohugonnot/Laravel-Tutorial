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

```
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

