<p align="center">
  <img src="resources/img/misc/win5x_logo_black.png" alt="Win5X Logo">
</p>
<p align="center">
    <img src="https://img.shields.io/static/v1.svg?label=version&message=3.9.1&color=black">
</p>

# Installation

## WebSockets Setup

Install **fork** of `laravel-echo-server`:

```
npm install -g laravel-echo-server-whisper
```

- If NPM repository is down, install it from [https://github.com/Win5X-Team/laravel-echo-server](https://github.com/Win5X-Team/laravel-echo-server)

### Private Keys & WS Encryption

#### Recaptcha

Recaptcha is required to authorize on the website.

Steps to enable Recaptcha:

1. Create Recaptcha v2 token
2. Create file `/storage/recaptcha.key` and put token in it
3. Run `npm run dev` to activate Recaptcha
4. Set `recaptcha_secret_key` in admin panel settings

#### WebPush

Generate vapid keys: `php artisan webpush:vapid`

#### `.env` file

Copy `.env.example` file and rename it to `.env`, afterwards run `php artisan key:generate`.

Change `APP_URL` to your website URL (Important for webhooks!)

#### Private keys

Our Client <-> Server WS implementation requires RSA public key & server key to prevent access token stealing.

Generation process is simple: run `php artisan win5x:keys`.

If you intend to use BitGo service, backup `BITGO_PASSPHRASE` from `.env` to somewhere so you won't lose access to BitGo wallet by some accident.

*Note: you need to execute `npm run dev` to include RSA public key in js source. If you test website straight after w/o compiling it won't authenticate you to WS server.*

## Scheduler

Configure scheduler:

`crontab -e`

`* * * * * cd /path-to-your-project && php artisan schedule:run >> /dev/null 2>&1`

#### WS Server Installation

```
laravel-echo-server init  # When asked, use port 2096 and setup SSL if you want to use it
```

Modify **laravel-echo-server.json**
```$json
"databaseConfig": {
	"redis": {
		"password": "redis password (set "redis" to {} if none)"
	},
	"sqlite": {
		"databasePath": "/database/laravel-echo-server.sqlite"
	},
	"listenWhishper": true,
	"prefixWhishper": "whisper"
},
```

### Bots

To create fake presence of users, you can start a bot to do that. Navigate to `/admin/bot`, modify settings to suit your needs and click at "Start".

If you wish to **stop** bot spawning, use `php artisan queue:clear`.

*Note: this command clears every job in the queue. After that you must restart multiplayer game queue with `php artisan game:chain all`.*

Bet value is hardcoded to be random from 1$ to 25$ (determined from current `CoinGecko` price), where higher bets are less common.

## Multiplayer games

First step is to [setup supervisor](https://laravel.com/docs/7.x/queues#supervisor-configuration).

After setup is complete you should start the chain so the game would work infinitely (```php artisan game:chain <game_id>```).

Example:
```
php artisan queue:clear        # Clear queue so unexpected things wouldn't happen
php artisan game:chain all     # Start chain for all games

# Example (debugging)
php artisan game:chain crash   # Start chain for Crash only
```

Current multiplayer games identifiers:

* crash
* baccarat
* slide

## Troubleshooting

#### Website and client-side provably fair results are different

Float precision tends to work differently in PHP, so there will be slight float differences.

Difference is very small, but sometimes it will be enough to make result invalid.

To fix this, make this change in ```php.ini```:
```
float_precision = -1
```

#### "Connecting to the server..."

Make sure that WS server (`laravel-echo-server`) is running.

## Post-install

Use this shell script to quickly launch WS server, Laravel queue and multiplayer games:

`./start.sh`

## Notes

* Never run ```php artisan game:chain``` twice. If you need to clear & restart queue chain, run `php artisan queue:clear` before.
