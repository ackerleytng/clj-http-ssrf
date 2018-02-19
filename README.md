# clj-http-ssrf [![Build Status](https://travis-ci.org/tirkarthi/clj-http-ssrf.svg?branch=master)](https://travis-ci.org/tirkarthi/clj-http-ssrf)

A clj-http middleware designed to prevent SSRF attacks

## Leiningen

`[xtreak/clj-http-ssrf "0.1.0"]`

## Rationale

Since many web applications accept URLs to which some payload will be posted from the app like webhook integrations there is a possibility that the URL might be an internal URL.

E.g. On notification a payload `{"success": true}` will be posted to some Webhook URL so that the integrated application will consume the payload. But in some cases malicious users might enter private URLs of the app that can be accessed only from the app subnet. Since HTTP requests from the apps pass the subnet checks the app might end up posting a payload to it's own private endpoint. In the above case when the URL is given as http://private.app.com/private/secret/url/ then the payload will be passed to that endpoint which might cause some unforseen actions.

clj-http enables building custom middlewares so that we can effectively block the request from being made. We can return custom status like 404 instead of 403 so that the attacker might think the URL doesn't exist reducing the attack vector.

## Usage

```
foo.core> (require [clj-http-ssrf :refer :all])
foo.core> (client/with-middleware (conj client/default-middleware (wrap-validators :hosts ["10.0.0.0/1"]))
                      (client/get "http://10.0.10.11/secret/private/endpoint/"))
{:status 403, :headers {}, :body ""}
foo.core> (client/with-middleware (conj client/default-middleware (wrap-validators :hosts ["10.0.0.0/1"] :status 404))
                      (client/get "http://10.0.10.11/secret/private/endpoint/"))
{:status 404, :headers {}, :body ""}
```

## Thanks

* https://github.com/JordanMilne/Advocate for the python library. This is a port of the same.
* colleagues at work for their ideas and suggestions

## Implementation in other languages

* https://github.com/JordanMilne/Advocate#acknowledgements

## Resources regarding SSRF

* https://www.owasp.org/index.php/Server_Side_Request_Forgery
* https://www.acunetix.com/blog/articles/server-side-request-forgery-vulnerability/

## License

Copyright © 2018 Karthikeyan S

Distributed under the MIT License