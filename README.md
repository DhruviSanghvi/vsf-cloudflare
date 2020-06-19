## Vue Storefront - Cloudflare Autopurge
You might use CDN not only to serve dist & assets directory but also SSR Output. In this case, you would want to dynamicly purge cache in Cloudflare when it is being purged in Varnish. For that, you need to clone this repo:
```
git submodule add https://github.com/new-fantastic/vsf-cloudflare src/modules/vsf-cloudflare
```

Then add `cloudflare` section in the PWA configuration:
```
"cloudflare": {
  "purge": true,
  "key": "someKey",
  "zoneIdentifier": "someZoneIdentifier"
},
```

`purge` should be equal `true`
To get `key` open: https://dash.cloudflare.com/profile/api-tokens and create a token.   
It is important to select `Create Custom Token` option, then in the permissions section set `Zone` - `Cache Purge` - `Purge`. For test purposes, I recommend to do not enable IP Address Filtering and set long enough TTL. Then `Continue to summary` and you will see your key.

To get `zoneIdentifier` open: https://dash.cloudflare.com/ - scroll down, then in the right panel you should see `API` and `Zone ID` - that's it.

You should also add `config.server.baseUrl` field in your config. Url **must** end with `/`, e.g.:
```
{
  "server": {
    // ...
    "trace": {
      "enabled": false,
      "config": {}
    },
    "baseUrl" : "https://my-super-fast-ecommerce-shop.com/"
  },
  ```

Now you need to apply purge-config loader to your app. Your config will be exposed in page's source or produced by Webpack - app.js. We cannot allow that because someone might use this data to purging your cache all the time! To prevent this merge this PR to your project: https://github.com/DivanteLtd/vue-storefront/pull/4540

It is important to put in your config:
```
"purgeConfig": [
    "server.invalidateCacheKey",
    "server.invalidateCacheForwardUrl",
    "server.trace",
    "redis",
    "install",
    "expireHeaders",
    "fastly",
    "nginx",
    "varnish",
    "cloudflare"
  ]
```
This array tells app which parts of config should be available only server side! Presented values will be very good base in most cases (in comparison to original config diff from a PR - I've added `cloudflare` value to the array).

That's all. Since now just after cache purge - it will send purge cache request to Cloudflare (This endpoint: https://api.cloudflare.com/#zone-purge-files-by-url). It respects limit of max 30 urls and divides them into chunks.
