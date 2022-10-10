acme-dns-01-cloudflare
==============
[![npm version](https://badge.fury.io/js/acme-dns-01-cloudflare.svg)](https://badge.fury.io/js/acme-dns-01-cloudflare)
[![Actions Status](https://github.com/nodecraft/acme-dns-01-cloudflare/workflows/Test/badge.svg)](https://github.com/nodecraft/acme-dns-01-cloudflare/actions)

[Cloudflare](https://www.cloudflare.com/) DNS + Let's Encrypt. This module handles ACME dns-01 challenges, compatible with [Greenlock.js](https://www.npmjs.com/package/greenlock) and [ACME.js](https://www.npmjs.com/package/acme). It passes [acme-dns-01-test](https://www.npmjs.com/package/acme-dns-01-test).

## Install
```bash
npm install acme-dns-01-cloudflare --save
```

## Cloudflare API Token

Whilst you can use a global API key and email to generate certs, we heavily encourage that you use a Cloudflare API token for increased security.

From your [Cloudflare Profile page](https://dash.cloudflare.com/profile), create an API Token with the following permissions:

- Zone -> Zone: Read
- Zone -> DNS: Edit

You can select specific zones or assign the token to all zones if preferred. The resulting API token should look something like this:


![Cloudflare API Token generation](https://up.jross.me/x2dcm)

## Usage

First, create an instance of the library with your Cloudflare API credentials or an API token. See the instructions above for more information.


```js
const acmeDnsCloudflare = require('acme-dns-01-cloudflare');

const cloudflareDns01 = new acmeDnsCloudflare({
	token: 'xxxxxx',
	verifyPropagation: true,
	verbose: true // log propagation delays and other debug information
});
````
Other options include `waitFor` and `retries` which control the number of propagation retries, and delay between retries. You probably won't need to tweak these unless you're seeing regular DNS related failures.

Then you can use it with any compatible ACME library, such as [Greenlock.js](https://www.npmjs.com/package/greenlock) or [ACME.js](https://www.npmjs.com/package/acme).


### Greenlock.js v4

See the [Greenlock.js documentation](https://www.npmjs.com/package/greenlock) for more information.

```js
const Greenlock = require('greenlock');
const pkg = require('./package.json');

const greenlock = Greenlock.create({
	packageAgent: pkg.name + '/' + pkg.version,
	configDir: "./store",
	maintainerEmail: "example@example.com"
});

greenlock.manager.defaults({
	agreeToTerms: true,
	subscriberEmail: "example@example.com",
	store: {
		module: "greenlock-store-fs",
		basePath: "./store/certs"
	},
	challenges: {
		"dns-01": cloudflareDns01
	}
});

greenlock.add({
	subject: "example.com",
	altnames: ["example.com", "www.example.com"]
}).then(function(){
	console.log("SUCCESS");
}).catch(console.error);
```

### Greenlock.js v2

The example below uses the `greenlock-store-fs` module to write these certs to disk for demonstration.

```js
const Greenlock = require('greenlock'),
	greenlockStore = require('greenlock-store-fs');

const store = greenlockStore.create({
	configDir: './store/certs',
	debug: true
});

const greenlock = Greenlock.create({
	server: 'https://acme-staging-v02.api.letsencrypt.org/directory',
	store: store,
	challenges: {
		'dns-01': cloudflareDns01
	},
	challengeType: 'dns-01',
	debug: true
});

greenlock.register({
	domains: ['example.com'],
	email: 'example@example.com',
	agreeTos: true,
	rsaKeySize: 2048,
	debug: true
}).then(() => {
	console.log('SUCCESS');
}).catch((err) => {
	console.error(err);
});
```



### ACME.js

```js
// TODO
```


## Tests
```bash
# CLOUDFLARE_TOKEN or both CLOUDFLARE_EMAIL and CLOUDFLARE_APIKEY env vars must be set, as well as DOMAIN
node ./test.js
```
