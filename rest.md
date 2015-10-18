REST API For Namecoin
=====================

1. The path shall have no version prefix such as "/v1", as that is not RESTful.

2. The namespace is divided via currency symbol.

3. The raw value can be requested via an Accept header of `application/octet-stream`.

       https://host/nmc/name/d%2fexample
       GET /nmc/name/d%2fexample HTTP/1.1
       Host: host
       Accept: application/octet-stream

       HTTP/1.1 200 OK
       Content-Type: application/octet-stream
       Content-Length: ...
       Last-Modified: 2001-01-01T12:00:00Z
       ETag: "(height)-(txid)"

       (name value)

4. The raw value, along with metadata, can be requested via an Accept header of `application/json`.

       https://host/nmc/name/d%2fexample
       GET /nmc/name/d%2fexample HTTP/1.1
       Host: host
       Accept: application/json

       HTTP/1.1 200 OK
       Content-Type: application/json
       Content-Length: ...
       Last-Modified: 2001-01-01T12:00:00Z
       ETag: "(height)-(txid)"

       {
         "key": "d/example",
         "value": "(base64-encoded name data",
         "height": 200000,
         "height_timestamp": "2001-01-01T12:00:00Z",
         "cur_height": 250000,
         "expired": false,
         "expires_in": 1000,
         "txid": "(txid)"
       }

   Base64 encoding of the value is necessary as values may contain binary data.

5. A HTTP Last-Modified header is sent representing the timestamp of the block containing the current
   name value.

6. A HTTP ETag header is sent representing the identity of the current name value. It SHOULD incorporate
   the block height and transaction ID.

7. HTTP conditional request semantics are supported.

       https://host/nmc/name/d%2fexample
       GET /nmc/name/d%2fexample HTTP/1.1
       Host: host
       Accept: application/json
       If-Modified-Since: 2001-01-01T12:00:00Z

       HTTP/1.1 304 Not Modified
       Last-Modified: 2001-01-01T12:00:00Z
       ETag: "(height)-(txid)"


       https://host/nmc/name/d%2fexample
       GET /nmc/name/d%2fexample HTTP/1.1
       Host: host
       Accept: application/json
       If-None-Match: "(height)-(txid)"

       HTTP/1.1 304 Not Modified
       Last-Modified: 2001-01-01T12:00:00Z
       ETag: "(height)-(txid)"

8. Expired names may be requested.

9. Names which do not exist result in 404s as expected.

       https://host/nmc/name/d%2fneverexisted
       GET /nmc/name/d%2fneverexisted HTTP/1.1
       Host: host
       Accept: application/json

       HTTP/1.1 404 Not Found
       Content-Type: text/plain

       Not Found
     
10. The server responds to complex Accept headers appropriately.

        https://host/nmc/name/d%2fneverexisted
        GET /nmc/name/d%2fneverexisted HTTP/1.1
        Host: host
        Accept: application/json; q=1.0, application/octet-stream; q=0.1

        HTTP/1.1 200 OK
        Content-Type: application/json

        ...


        https://host/nmc/name/d%2fneverexisted
        GET /nmc/name/d%2fneverexisted HTTP/1.1
        Host: host
        Accept: application/octet-stream; q=1.0, application/json; q=0.1

        HTTP/1.1 200 OK
        Content-Type: application/octet-stream

        ...


        https://host/nmc/name/d%2fneverexisted
        GET /nmc/name/d%2fneverexisted HTTP/1.1
        Host: host
        Accept: application/x-some-unknown

        HTTP/1.1 406 Not Satisfiable
        Content-Type: text/plain

        Not Satisfiable

11. If both `application/octet-stream` and `application/json` are Acceptable
    and are ranked equally, `application/octet-stream` shall be preferred.

12. Implementations should consider appropriate adoption of TLS, HTTP Authorization,
    Strict-Transport-Security, CORS, IP-based access restrictions, rate limiting, etc.

13. Implementations MAY support other MIME types (for example, clients
    requesting `text/html` could be shown an informational page).
    `application/octet-stream` must remain the default; this is fine as web
    browsers always send `Accept` headers.

14. The API SHOULD NOT set any cookies.

15. Only the GET and HEAD (and OPTIONS) methods are supported. All other
    methods should result in a 405 response.
