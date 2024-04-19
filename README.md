OAuth2 PAM module
=================

This PAM module enables login with OAuth2 token instead of password.

## How to install it:

```bash
$ sudo apt-get install libcurl4-openssl-dev libpam-dev
$ git submodule init
$ git submodule update
$ make
$ sudo make install
```

## Configuration

```
auth sufficient pam_oauth2.so <tokeninfo url> <login field> [POST] [:[<Authz scheme>:]<Authz parameters>] key1=value2 key2=value2
account sufficient pam_oauth2.so
```

Optional parameter `POST` indicates to use POST method when sending request to the `<tokeninfo url>`. This also adds 
`Content-Type: application/x-www-form-urlencoded` header and moves CGI parameters from `<tokeninfo url>` into request body.
If this parameter is omitted, the GET method is used.

Optional parameter `:[<Authz scheme>:]<Authz parameters>` specify a field value for `Authorization` HTTP header to be sent along with `<tokeninfo url>`. If authorisation scheme `<Authz scheme>` is omitted, the Basic authorisation scheme is used. Examples:
- `:dXNlcjpwYXNzd29yZA==` will result in `Authorization: Basic dXNlcjpwYXNzd29yZA==` header to be added;
- `:Bearer:mF_9.B5f-4.1JqM` will result in `Authorization: Bearer mF_9.B5f-4.1JqM` header to be added.
If this parameter is omitted, no `Authorization` header is added.

## How it works

Lets assume that configuration is looking like:

```
auth sufficient pam_oauth2.so https://foo.org/oauth2/tokeninfo?access_token= uid grp=tester
```

And somebody is trying to login with login=foo and token=bar.

pam\_oauth2 module will make http request https://foo.org/oauth2/tokeninfo?access_token=bar (tokeninfo url is simply concatenated with token) and check response code and content.

If the response code is not 200 - authentication will fail. After that it will check response content:

```json
{
  "access_token": "bar",
  "expires_in": 3598,
  "grp": "tester",
  "scope": [
    "uid"
  ],
  "token_type": "Bearer",
  "uid": "foo"
}
```

It will check that response is a valid JSON object and top-level object contains following key-value pairs:
```json
  "uid": "foo",
  "grp": "tester"
```

If some keys haven't been found or values don't match with expectation - authentication will fail.


### Sample configuration for [Keycloak](https://github.com/keycloak/keycloak)

*As of Keycloak version 24.0.2* 

```
auth sufficient pam_oauth2.so http://keycloak.local:8080/realm/master/protocol/openid-connect/token/introspect?token= client_id POST :Y2xpZW50X2lkOnNlY3JldA==
account sufficient pam_oauth2.so http://keycloak.local:8080/realm/master/protocol/openid-connect/token/introspect?token= client_id POST :Y2xpZW50X2lkOnNlY3JldA==
```

*Hint:* you can use the following command to obtain authorisation parameter from values of `client_id` and `secret`:

```
echo -n <client_id>:<secret> | base64
```

### Issues and Contributing

Oauth2 PAM module welcomes questions via our [issues tracker](https://github.com/CyberDem0n/pam-oauth2/issues). We also greatly appreciate fixes, feature requests, and updates; before submitting a pull request, please visit our [contributor guidelines](https://github.com/CyberDem0n/pam-oauth2/blob/master/CONTRIBUTING.rst).

License
-------

This project uses the [MIT license](https://github.com/CyberDem0n/pam-oauth2/blob/master/LICENSE).
