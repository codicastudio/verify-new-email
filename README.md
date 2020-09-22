# Laravel Verify New Email

Laravel supports verifying email addresses out of the box. This package adds support for verifying *new* email addresses. When a user updates its email address, it won't replace the old one until the new one is verified. Super easy to set up, still fully customizable. If you want it can be used as a drop-in replacement for the built-in Email Verification features as this package supports unauthenticated verification and auto-login. Support for Laravel 6.0 and higher and requires PHP 7.2 or higher.


## Installation

You can install the package via composer:

```bash
composer require protonemedia/laravel-verify-new-email
```

## Configuration

Publish the database migration, config file and email view:

```bash
php artisan vendor:publish --provider="ProtoneMedia\LaravelVerifyNewEmail\ServiceProvider"
```

You can set the redirect path in the `verify-new-email.php` config file. The user will be redirected to this path after verification.

The expire time of the verification URLs can be changed by updating the `auth.verification.expire` setting and defaults to 60 minutes.

## Usage

Add the `MustVerifyNewEmail` trait to your `User` model and make sure it implements the framework's `MustVerifyEmail` interface as well.

``` php
<?php

namespace App;

use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use ProtoneMedia\LaravelVerifyNewEmail\MustVerifyNewEmail;

class User extends Authenticatable implements MustVerifyEmail
{
    use MustVerifyNewEmail, Notifiable;
}
```

Now your `User` model has a few new methods:

``` php
// generates a token and sends a verification mail to 'me@newcompany.com'.
$user->newEmail('me@newcompany.com');

// returns the currently pending email address that needs to be verified.
$user->getPendingEmail();

// resends the verification mail for 'me@newcompany.com'.
$user->resendPendingEmailVerificationMail();

// deletes the pending email address
$user->clearPendingEmail();
```

The `newEmail` method doesn't update the user, its current email address stays current until the new one if verified. It stores a token (associated to the user and new email address) in the `pending_user_emails` table. Once the user verifies the email address by clicking the link in the mail, the user model will be updated and the token will be removed from the `pending_user_emails` table.

The `resendPendingEmailVerificationMail` does exactly the same, it just grabs the new email address from the previous attempt.

### Login after verification

The user that verified its email address will be logged in automatically. You can disable this by chaning the `login_after_verification` configuration setting to `false`.

### Overriding the default Laravel Email Verification

The default [Laravel implementation](https://laravel.com/docs/master/verification) requires the user to be logged in before it can verify its email address. If you want to use this package's logic to handle that first verification flow as well, override the `sendEmailVerificationNotification` method as shown below.

``` php
<?php

namespace App;

use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use ProtoneMedia\LaravelVerifyNewEmail\MustVerifyNewEmail;

class User extends Authenticatable implements MustVerifyEmail
{
    use MustVerifyNewEmail, Notifiable;

    public function sendEmailVerificationNotification()
    {
        $this->newEmail($this->getEmailForVerification());
    }
}
```

### Customization

You can change the content of the verification mail by editing the published views which can be found in the `resources/views/vendor/verify-new-email` folder. The `verifyNewEmail.blade.php` view will be sent when verifying *updated* email addresses. The `verifyFirstEmail.blade.php` view will be sent when a User verifies its initial email address for the first time (after registering). Alternatively you set your own custom Mailables classes in the config file:

``` php
<?php

return [

    'mailable_for_first_verification' => \ProtoneMedia\LaravelVerifyNewEmail\Mail\VerifyFirstEmail::class,

    'mailable_for_new_email' => \ProtoneMedia\LaravelVerifyNewEmail\Mail\VerifyNewEmail::class,

];
```

You can also override the `sendPendingEmailVerificationMail` method to change the behaviour of sending the verification mail:

``` php
<?php

namespace App;

use Illuminate\Contracts\Auth\MustVerifyEmail;
use Illuminate\Foundation\Auth\User as Authenticatable;
use Illuminate\Notifications\Notifiable;
use ProtoneMedia\LaravelVerifyNewEmail\MustVerifyNewEmail;
use ProtoneMedia\LaravelVerifyNewEmail\PendingUserEmail;

class User extends Authenticatable implements MustVerifyEmail
{
    use MustVerifyNewEmail, Notifiable;

    public function sendPendingEmailVerificationMail(PendingUserEmail $pendingUserEmail)
    {
        // send the mail...
    }
}
```

The package has a controller to handle the activation of the new email address. You can specify a custom route in the config file which will be used to generate the verification URL. The token will be passed in as a parameter and the URL will be signed.

``` php
<?php

return [

    'route' => 'user.email.verify',

];

```

### Testing

``` bash
composer test
```

