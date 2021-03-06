# SegAnnDB-login

This repository is a clone of original [pyramid_google_login](https://github.com/ludia/pyramid_google_login)

It contains modifications done in code to make it work with SegAnnDB

**Approach**

`pyramid_google_login` does not handle any kind of authentication policy by itself. It only helped in fetching the
user profile details from google. I made a workaround, now it sets a cookie in the response which is equivalent to the user_id
(email)

Some bugs were also fixed, turns out the original version did not work with the a latest version of pyramid anymore, so a 
couple bugs were fixed as well.

**On pypi -**

This package will be uploaded to pypi as well, under the same name.

Setup: Application
==================

Once `seganndb_login` is installed, you must use the `config.include`
mechanism to include it into your Pyramid project's configuration.  In your
Pyramid project's `__init__.py`:

```python
 config = Configurator(.....)
 config.include('seganndb_login')
```

Alternately you can use the `pyramid.includes` configuration value in your
`.ini` file:

```ini
   [app:myapp]
   pyramid.includes = seganndb_login
```

Setup: settings
===============

Mandatory settings:

```ini
   security.google_login.client_id = xxxxxxx.apps.googleusercontent.com
   security.google_login.client_secret = xxxxxxxxxxxxxxxxxxxxxxxxx
```

Optional settings:

```ini
   # List of Google scopes (`email` is automatically included)
   security.google_login.scopes = email

   # Set the access type to `offline` to get a refresh_token (default: online)
   security.google_login.access_type = online

   # Field used to extract the userid (generally `email` or `id`)
   security.google_login.user_id_field = email

   # Restrict authentication to a Google Apps domain
   security.google_login.hosted_domain = example.net

   # Redirect destination for logged in user.
   security.google_login.landing_url = /
   security.google_login.landing_route = my_frontend_route
   security.google_login.landing_route = mymodule:static/

   # Add a banner on the sign in page
   security.google_login.signin_banner = Welcome on Project Euler

   # Add an advice on the sign in page
   security.google_login.signin_advice = Ask Dilbert for access
```

Setup: Google project
=====================

- Create a project on https://console.developers.google.com
- Create a OAuth Client ID

   + Choose a `Web Application` application type
   + Add all variants of your host in Javascript Origins

      * Secure and non secure url are differentiated
      * Optionally include your development host with
        `http://localhost:6543` rather than an `http://127.0.0.1:6543`
        (it would be refused)

Notes:

- No `Permissions` are needed by `seganndb_login` itself.
- Client ID parameters are heavily cached. In development, re-creating a client
  id is often the best idea.


General Usage
=============

When a user must be authenticated by Google, he must be sent to the
`auth_signin` route url. The helper method
`seganndb_login.redirect_to_signin` redirect the user to the sign in
page. This helper is handy to specify the next url and an optional message.

```python

   @forbidden_view_config()
   def unauthenticated(context, request):
       return redirect_to_signin(request, url=request.path_qs)
```

Once the user is authenticated, the `UserLoggedIn` pyramid event is
broadcasted. The application can perform subsequent validations, create the
user profile or update it.

After that, the `pyramid.security.remember` helper is called.

Then, the user will be redirected to an url specified by:

- query parameter (signin page): `url`
- setting: `security.google_login.landing_url`
- fallback: `/`

When a user must be logged out, he must be directed on the `auth_logout`
route url. Once logged out, he will be redirected back to the sign in page.


Offline Usage
=============

If you want to call the Google APIs on behalf of the user, you must store the
OAuth2 tokens provided in the UserLoggedIn event. The `access_token` is
usable for an `expires_in` period. Then the `refresh_token` must be used to
refresh the `access_token`. This `refresh_token` is valide until the user
revoke the application permissions.

By default, the only scope requested is `email` to identify the user. To call
other Google APIs, you must add the related scopes as this:

```ini
   [app:myapp]

   security.google_login.scopes =
       email
       https://www.googleapis.com/auth/admin.directory.user.readonly
```

Events
======

UserLoggedIn
------------

The user has logged in by Google.

Properties:

- userid
- oauth2_token

  + access_token
  + expires_in
  + refresh_token

- user_info

  + Google user_info properties...

UserLoggedOut
-------------

The user has logged out.

Properties:

- userid

All the related information to user_profiles can be found in file `views.py`

Remembering User Authentication
-------------------------------

To remember user authentication, we set a cookie in response. You will have to
handle client side login/logout by yourself. 

Development
===========

Running tests::

   $ pip install -r requirements-test.txt
   $ nosetests

Running pylama (linters)::

   $ pip install pylama
   ...
   $ pylama
