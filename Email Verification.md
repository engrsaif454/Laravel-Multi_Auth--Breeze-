
Laravel Breeze install করার পরে ২টি step complete করার মাধ্যমে Email Verification Implement করা যাবে। সেগুলো হলোঃ


## Step-1:

1. `User` Model থেকে নিচের লাইনটি Uncomment করে দিতে হবেঃ

```php
use Illuminate\Contracts\Auth\MustVerifyEmail;
```

2. `User` Mode -এ `class User extends Authenticatable` এর পরে নিচের লাইনটি যুক্ত করতে হবেঃ

```php
	implements MustVerifyEmail
```

পুরো লাইনটি নিচের মতো হবেঃ

```php
	class User extends Authenticatable implements MustVerifyEmail
```
_____
## Step-2:

`.env` file এ নিচের SMTP Server -এ সেটাপ করতে হবেঃ

```php
MAIL_MAILER=smtp
MAIL_HOST=smtp.gmail.com
MAIL_PORT=587
MAIL_USERNAME=colorworld454@gmail.com
MAIL_PASSWORD=gjsbpjejcwitnnkw
MAIL_ENCRYPTION=tls

MAIL_FROM_ADDRESS="colorworld454@gmail.com"
MAIL_FROM_NAME="${APP_NAME}"
```
______
