---

template:    article
reviewed:    2016-08-04
title:       Using secure App secrets
naviTitle:   App secrets
lead:        App secrets provide a secure storage and access method for all the credentials your App needs to run.
group:       deployment

keywords:
    - Secrets
    - Credentials
    - ENV vars
    - Environment variables

seeAlsoLinks:
   - env-vars

tags:
  - php
  - beginner

---

## Problem

Your App needs confidential credentials to connect to other services (user-names, passwords, API keys or alike). Those can be on fortrabbit like the MySQL database access, or externally like an API key for a cloud storage.

We advise to [not store secrets within your code base](//blog.fortrabbit.com/how-to-keep-a-secret#not-in-git), since it's controlled by Git - so version controlled. You [should not store them unencrypted in ENV vars](//blog.fortrabbit.com/how-to-keep-a-secret#not-in-an-env-var-either-) either.

In addition: your App will run in at least two environments: locally and on fortrabbit. Any solution must address the problems resulting from multiple runtime environments.

## Solution

Use fortrabbits App secrets to store your credentials safely. App secrets are stored in a JSON file called `secrets.json` which is only accessible by you and your App. The location of this JSON file is stored in a predefined environment variable called `APP_SECRETS`.


## App secrets in your App

Access App secrets from inside your App via PHP like so:

```php
// read all App secrets from the JSON file, get the location via ENV var
$secrets = json_decode(file_get_contents($_SERVER["APP_SECRETS"]), true);

// use a specific secret
$meaning_of_life = $secrets['CUSTOM']['MEANING_OF_LIFE'];
```

```php
// App secrets are ordered in a tree structure:
$secrets == [
    'MYSQL' => [
        'PASSWORD' => "{{mysql-password}}",
        'USER'     => "{{mysql-user}}",
        'DATABASE' => "{{mysql-database}}",
        'HOST'     => "{{mysql-host}}",
        'PORT'     => "{{mysql-port}}",
    ],
    'CUSTOM' => [
        'YOUR_CUSTOM_SECRET'    => "{{YOUR_CUSTOM_SECRET}}",
        'yourOtherCustomSecret' => "{{yourOtherCustomSecret}}"
    ]
];
```

See examples to use the App secrets to connect to MySQL for: [Laravel](install-laravel#toc-mysql), [Symfony](install-symfony#toc-mysql), [WordPress](install-wordpress#toc-mysql), [Craft CMS](install-craft-2#toc-mysql), [Drupal](install-drupal-8#toc-mysql).


## App secrets from local

Read App secrets from your local machine by using an [SSH remote exec](/remote-ssh-execution) command in your terminal:

```bash
# show all App secrets
$ ssh {{ssh-user}}@deploy.{{region}}.frbit.com secrets
# {
#     "MYSQL": {
#         "PASSWORD": "{{mysql-password}}",
#         "USER": "{{app-name}}",
#         "DATABASE": "{{app-name}}",
#         "HOST": "{{app-name}}.mysql.{{region}}.frbit.com",
#         "PORT": "3306",
#     },
#     "CUSTOM": {
#         "YOUR_CUSTOM_SECRET": "The custom content",
#         "yourOtherCustomSecret": "The custom content"
#     }
# }

# show only MySQL secrets
$ ssh {{ssh-user}}@deploy.{{region}}.frbit.com secrets MYSQL
# {
#     "PASSWORD": "{{mysql-password}}",
#     "USER": "{{app-name}}",
#     "DATABASE": "{{app-name}}",
#     "HOST": "{{app-name}}.mysql.{{region}}.frbit.com",
#     "PORT": "3306",
# }

# show only MySQL password
$ ssh {{ssh-user}}@deploy.{{region}}.frbit.com secrets MYSQL.PASSWORD
# {{mysql-password}}
```


## Adding custom App secrets

You can add or remove custom App secrets in the [Dashboard](dashboard). You'll do so in the settings of your [App](app). The contents of the App secrets cannot be viewed in the Dashboard due to the underlying encryption, which we consider a feature, not a bug.

<div markdown="1" data-user="known">

[See App secrets of your App **{{app-name}}**](https://dashboard.fortrabbit.com/apps/{{app-name}}/secrets)
[Add new App secrets for your App **{{app-name}}**](https://dashboard.fortrabbit.com/apps/{{app-name}}/secrets/new)

</div>


## App secrets vs ENV vars

App secrets are closely related to ENV vars insofar that they are both available to your App at runtime. The big difference between them is that App secrets are stored highly secured and they are not automatically dumped out by debug tools - such as `phpinfo()` or your favorite debug toolbar.

To achieve a level of security approaching the App secrets with ENV vars you can encrypt them, as described in [the ENV var article](env-vars#toc-env-vars-vs-security).

## App secrets vs local environment

Since access to your App secrets should be done using `$_SERVER['APP_SECRETS']`, you can easily set this environment variable locally with a path to a local JSON file containing your local (dummy) secrets.
