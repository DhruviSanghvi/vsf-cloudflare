## Vue Storefront - Cloudflare Autopurge
You might use CDN not only to serve dist & assets directory but also SSR Output. In this case, you would want to dynamicly purge cache in Cloudflare when it is being purged in Varnish. For that, you need to add `cloudflare` section in the PWA configuration:
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

That's all. Since now just after cache purge - it will send purge cache request to Cloudflare (This endpoint: https://api.cloudflare.com/#zone-purge-files-by-url). It respects limit of max 30 urls and divides them into chunks.