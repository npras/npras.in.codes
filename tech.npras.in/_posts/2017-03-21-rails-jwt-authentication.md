---
layout: post
title: "Rails API: Using Json Web Token for Authentication"
excerpt: "Notes from Chris' GoRails episode on implementing api authentication using JWT"
---

GoRails has a great [video series](https://gorails.com/series/how-to-build-apis-with-rails) explaining JWT and how it can be used to build authentication. I implemented it in a test app and here are the notes from the experience.

The basic idea for the api authentication is this:

* First send a `POST /login` request with username and password. If the details match, then,
* Return a json with the jwt. It will have the user's id as the primary payload
* The client app - either mobile or react/angular/vue app's - will then have to save this jwt (localstorage/sessionstorage etc), and then,
* the client app, in all subsequent api requests, will pass this jwt as a "AUTHORIZATION" header in the http request,
* the server will validate this jwt in the header like so: It will decode it with the same secret key it encoded, and then get the current user from the id in the payload. If the decode failed, then the api will be denied access. If not, the api will be grant access.
* At the server side, we can strengthen the JWT security using 2 means: using a secret to encode and decode, and using an expiration value for the token.

First install the `jwt` gem.

We'll need a place to encode the payload as JWT, and decode the payload from the JWT. This is the `app/models/json_web_token.rb` file:

```rb
class JsonWebToken

  def self.encode(payload, expiration=expiration_from_config)
    payload = payload.merge(exp: expiration)
    JWT.encode(payload, ENV['SECRET_KEY_BASE'])
  end


  def self.decode(token)
    JWT.decode(token, ENV['SECRET_KEY_BASE'])&.first
  end


  def self.expiration_from_config
    Chronic
      .parse( "#{ENV['JWT_TOKEN_EXPIRATION_MINS']} mins from now")
      .to_i
  end

end
```

I've begin to use the Chronic gem in place of the rails default date helpers. They feel better.

The `payload` is a hash that holds the currently signed in user's id in the `sub` key:

```rb
{ sub: user.id }
```


The login request is served by a `authentication_controller.rb`:

```rb
class AuthenticateController < ApplicationController

  skip_before_action :authenticate_token!, only: [:login]


  def login
    user = User.find_by(email: login_params[:email])

    if user&.authenticate(login_params[:password])
      render json: { token: JsonWebToken.encode(sub: user.id) }
    else
      render json: { errors: [ {title: "email or password is incorrect"} ] }, status: :unauthorized
    end
  end


  private def login_params
    @_login_params ||= params.require(:auth).permit(
      :email,
      :password
    )
  end

end
```

All the other api requests will be authenticated using a before_action like so:

```rb
  before_action def authenticate_token!
    payload = JsonWebToken.decode(auth_token)
    @current_user = User.find_by(id: payload['sub'])
  # https://github.com/jwt/ruby-jwt/blob/master/lib/jwt/error.rb
  rescue JWT::ExpiredSignature
    render json: { errors: [ {title: "Auth token has expired"} ] }, status: :unauthorized
  rescue JWT::DecodeError
    render json: { errors: [ {title: "Invalid auth token"} ] }, status: :unauthorized
  end


  private def auth_token
    @_auth_token ||= request.headers.fetch('Authorization', '').split(' ').last
  end
```
