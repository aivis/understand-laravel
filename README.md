## Laravel 5 service provider for Understand.io

[![Build Status](https://api.travis-ci.org/understand/understand-laravel5.svg?branch=master)](https://api.travis-ci.org/understand/understand-laravel5.svg?branch=master)
[![Latest Stable Version](https://poser.pugx.org/understand/understand-laravel5/v/stable.svg)](https://packagist.org/packages/understand/understand-laravel5) 
[![Total Downloads](https://poser.pugx.org/understand/understand-laravel5/downloads)](https://packagist.org/packages/understand/understand-laravel5)
[![License](https://poser.pugx.org/understand/understand-laravel5/license.svg)](https://packagist.org/packages/understand/understand-laravel5)

- [Quick start](#quick-start)  
- [Upgrading To Version 2](#upgrading-to-version-2-of-the-service-provider)

### Introduction

This packages provides a full abstraction for Understand.io and provides extra features to improve Laravel's default logging capabilities. It is essentially a wrapper around Laravel's event handler to take full advantage of Understand.io's data aggregation and analysis capabilities.

### Quick start

1. Add the package to your project
    
```
composer require understand/understand-laravel5
```

2. Add the ServiceProvider to the `providers` array in `config/app.php`
  
```php
Understand\UnderstandLaravel5\UnderstandLaravel5ServiceProvider::class,
```

3. Set your Understand.io input token in your `.env` file
  
```php
UNDERSTAND_ENABLED=true
UNDERSTAND_TOKEN=your-input-token-from-understand-io
```

4. Send your first error

```php 
// anywhere inside your Laravel app
\Log::error('Understand.io test error');
```

- We recommend that you make use of a async handler - [How to send data asynchronously](#how-to-send-data-asynchronously)  
- If you are using Laravel 5.0 (`>= 5.0, < 5.1`) version, please read about - [How to report Laravel 5.0 exceptions](#how-to-report-laravel-50--50--51-exceptions).
- For advanced configuration please read about - [Advanced configuration](#advanced-configuration)


### How to send events/logs

#### Laravel logs
By default, Laravel automatically stores its [logs](http://laravel.com/docs/errors#logging) in `storage/logs`. By using this package, your log data will also be sent to your Understand.io channel. This includes error and exception logs, as well as any log events that you have defined (for example, `Log::info('my custom log')`).

```php 
\Log::info('my message', ['my_custom_field' => 'my data']);
```
#### PHP/Laravel exceptions
By default, all errors and exceptions with code fragments and stack traces will be sent to Understand.io. 

In addition, all SQL queries that were executed before each exception or error will also be sent to Understand.io. Note that for security and privacy reasons, we do NOT substitute query parameters. The SQL logging feature can be disabled if you wish -  please see [Advanced configuration](#advanced-configuration).

### How to send data asynchronously

##### Async handler
By default each log event will be sent to Understand.io's api server directly after the event happens. If you generate a large number of logs, this could slow your app down and, in these scenarios, we recommend that you make use of an async handler. To do this, set the config parameter `UNDERSTAND_HANDLER` to `async` in your `.env` file.

```php
# Specify which handler to use - sync, queue or async. 
# 
# Note that the async handler will only work in systems where 
# the CURL command line tool is installed
UNDERSTAND_HANDLER=async
```

The async handler is supported in most systems - the only requirement is that the CURL command line tool is installed and functioning correctly. To check whether CURL is available on your system, execute following command in your console:

```
curl -h
```

If you see instructions on how to use CURL then your system has the CURL binary installed and you can use the ```async``` handler.

> Keep in mind that Laravel allows you to specify different configuration values in different environments. You could, for example, use the async handler in production and the sync handler in development.

### How to report Laravel 5.0 (`>= 5.0, < 5.1`) exceptions 

Laravel's (`>= 5.0, < 5.1`) exception logger doesn't use event dispatcher (https://github.com/laravel/framework/pull/10922) and that's why you need to add the following line to your `Handler.php` file (otherwise Laravel's exceptions will not be sent Understand.io).

- Open `app/Exceptions/Handler.php` and put this line `\UnderstandExceptionLogger::log($e)` inside `report` method.
  
  ```php
  public function report(Exception $e)
  {
      \UnderstandExceptionLogger::log($e);

      return parent::report($e);
  }
  ```
 
 
### Advanced Configuration

1. Publish configuration file

```
php artisan vendor:publish --provider="Understand\UnderstandLaravel5\UnderstandLaravel5ServiceProvider"
```

### Upgrading To Version 2 of the Service Provider

1. Update your `package.json` file, change the version of `understand/understand-laravel5` package and run `composer update understand/understand-laravel5`.
```json
"understand/understand-laravel5": "^2.0",
```

2. Add the following configuration variable to your `.env` file.
```php
UNDERSTAND_ENABLED=true
```

3. If you previously created a `understand-laravel.php` config file in your `config` directory, please delete it and follow [Advanced configuration](#advanced-configuration) steps to publish a new version if necessary.

### Requirements 
##### UTF-8
This package uses the json_encode function, which only supports UTF-8 data, and you should therefore ensure that all of your data is correctly encoded. In the event that your log data contains non UTF-8 strings, then the json_encode function will not be able to serialize the data.

http://php.net/manual/en/function.json-encode.php

### License

The Laravel Understand.io service provider is open-sourced software licensed under the [MIT license](http://opensource.org/licenses/MIT)
