[![Build Status](https://travis-ci.org/cgauge/Flask-AWSCognito.svg?branch=master)](https://travis-ci.org/cgauge/Flask-AWSCognito)
[![Documentation Status](https://readthedocs.org/projects/flask-awscognito/badge/?version=latest)](https://flask-awscognito.readthedocs.io/en/latest/?badge=latest)

# AWS Cognito for authentication in Flask

Documentation https://flask-awscognito.readthedocs.io

## Maintainer(s) Needed

This project is in search for new maintainer(s). Please see [Issue #13](https://github.com/cgauge/Flask-AWSCognito/issues/13) for details.

## Note about Reddig fork:

I created this fork as a way to use cookies to store the tokens, as I did not believe this was natively supported. 

## Example App Using Cookies

```python
from flask import Flask, redirect, request, jsonify
from flask_awscognito import AWSCognitoAuthentication
app = Flask(__name__)

jwt = JWTManager(app)

app.config['AWS_DEFAULT_REGION'] = 'eu-west-1'
app.config['AWS_COGNITO_DOMAIN'] = 'domain.com'
app.config['AWS_COGNITO_USER_POOL_ID'] = 'eu-west-1_XXX'
app.config['AWS_COGNITO_USER_POOL_CLIENT_ID'] = 'YYY'
app.config['AWS_COGNITO_USER_POOL_CLIENT_SECRET'] = 'ZZZZ'
app.config['AWS_COGNITO_REDIRECT_URL'] = 'http://localhost:5000/callback'
app.config['AWS_COGNITO_PREFER_COOKIE_TOKEN'] = True
app.config['AWS_COGNITO_COOKIE_TOKEN_NAME'] = app.config['JWT_ACCESS_COOKIE_NAME']
app.config['JWT_COOKIE_SECURE'] = True
app.config['JWT_COOKIE_CSRF_PROTECT'] = True

aws_auth = AWSCognitoAuthentication(app)


@app.route('/callback')
def aws_cognito_redirect():
    access_token = aws_auth.get_access_token(request.args)
    resp = make_response(redirect(url_for('index')))
    flask_jwt_extended.set_access_cookies(resp, access_token, max_age=30 * 60)
    return resp


@app.route('/')
@aws_auth.authentication_required
def index():
    claims = aws_auth.claims  # also available through g.cognito_claims
    if 'username' in claims:
        session['username'] = claims['username']
        return render_template('/index.html', username=claims['username'])
    else:
        return url_for('callback')

@app.route('/login')
def login():
    return redirect(aws_auth.get_sign_in_url())


if __name__ == '__main__':
    app.run(debug=True)

```

## Example App Using Tokens

```python
from flask import Flask, redirect, request, jsonify
from flask_awscognito import AWSCognitoAuthentication
app = Flask(__name__)

app.config['AWS_DEFAULT_REGION'] = 'eu-west-1'
app.config['AWS_COGNITO_DOMAIN'] = 'domain.com'
app.config['AWS_COGNITO_USER_POOL_ID'] = 'eu-west-1_XXX'
app.config['AWS_COGNITO_USER_POOL_CLIENT_ID'] = 'YYY'
app.config['AWS_COGNITO_USER_POOL_CLIENT_SECRET'] = 'ZZZZ'
app.config['AWS_COGNITO_REDIRECT_URL'] = 'http://localhost:5000/callback'


aws_auth = AWSCognitoAuthentication(app)


@app.route('/')
@aws_auth.authentication_required
def index():
    claims = aws_auth.claims # also available through g.cognito_claims
    return jsonify({'claims': claims})


@app.route('/callback')
def aws_cognito_redirect():
    access_token = aws_auth.get_access_token(request.args)
    return jsonify({'access_token': access_token})


@app.route('/login')
def login():
    return redirect(aws_auth.get_sign_in_url())


if __name__ == '__main__':
    app.run(debug=True)

```
