# 0x02. Session authentication

## Background Context

In this project, you will implement a Session Authentication. You are not allowed to install any other module.

In the industry, you should not implement your own Session authentication system and use a module or framework that does it for you (like in Python-Flask: Flask-HTTPAuth). Here, for the learning purpose, we will walk through each step of this mechanism to understand it by doing.

## Resources
Read or watch:

- [REST API Authentication Mechanisms - Only the session auth part](https://www.youtube.com/watch?v=501dpx2IjGY)
- [HTTP Cookie](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
- [Flask](https://flask.palletsprojects.com/en/2.0.x/)
- [Flask Cookie](https://flask.palletsprojects.com/en/2.0.x/api/#flask.Response.set_cookie)

## Learning Objectives
At the end of this project, you are expected to be able to explain to anyone, without the help of Google:

### General
- What authentication means
- What session authentication means
- What Cookies are
- How to send Cookies
- How to parse Cookies

## Requirements
### Python Scripts
- All your files will be interpreted/compiled on Ubuntu 18.04 LTS using python3 (version 3.7)
- All your files should end with a new line
- The first line of all your files should be exactly `#!/usr/bin/env python3`
- A `README.md` file, at the root of the folder of the project, is mandatory
- Your code should use the `pycodestyle` style (version 2.5)
- All your files must be executable
- The length of your files will be tested using `wc`
- All your modules should have a documentation (`python3 -c 'print(__import__("my_module").__doc__)'`)
- All your classes should have a documentation (`python3 -c 'print(__import__("my_module").MyClass.__doc__)'`)
- All your functions (inside and outside a class) should have a documentation (`python3 -c 'print(__import__("my_module").my_function.__doc__)'` and `python3 -c 'print(__import__("my_module").MyClass.my_function.__doc__)'`)
- A documentation is not a simple word, it’s a real sentence explaining what’s the purpose of the module, class or method (the length of it will be verified)

## Tasks
### 0. Et moi et moi et moi!
- Copy all your work from the `0x06. Basic authentication` project into this new folder.

- In this version, you implemented Basic authentication to give you access to all User endpoints:

  - GET /api/v1/users
  - POST /api/v1/users
  - GET /api/v1/users/<user_id>
  - PUT /api/v1/users/<user_id>
  - DELETE /api/v1/users/<user_id>

- Now, you will add a new endpoint: GET /users/me to retrieve the authenticated User object.

- Copy the `models` and `api` folders from the previous project `0x06. Basic authentication`.

- Please make sure all mandatory tasks of the previous project are done at 100% because this project (and the rest of this track) will be based on it.

- Update `@app.before_request` in `api/v1/app.py`:
  - Assign the result of `auth.current_user(request)` to `request.current_user`

- Update the method for the route GET /api/v1/users/<user_id> in `api/v1/views/users.py`:
  - If `<user_id>` is equal to `me` and `request.current_user` is None: `abort(404)`
  - If `<user_id>` is equal to `me` and `request.current_user` is not None: return the authenticated User in a JSON response (like a normal case of GET /api/v1/users/<user_id> where `<user_id>` is a valid User ID)
  - Otherwise, keep the same behavior

### 1. Empty session
- Create a class `SessionAuth` that inherits from `Auth`. For the moment, this class will be empty. It’s the first step for creating a new authentication mechanism:
  - Validate if everything inherits correctly without any overloading
  - Validate the “switch” by using environment variables
- Update `api/v1/app.py` to use a `SessionAuth` instance for the variable `auth` depending on the value of the environment variable `AUTH_TYPE`. If `AUTH_TYPE` is equal to `session_auth`:
  - Import `SessionAuth` from `api/v1/auth/session_auth`
  - Create an instance of `SessionAuth` and assign it to the variable `auth`
- Otherwise, keep the previous mechanism.

### 2. Create a session
- Update the `SessionAuth` class:
  - Create a class attribute `user_id_by_session_id` initialized by an empty dictionary
  - Create an instance method `def create_session(self, user_id: str = None) -> str` that creates a Session ID for a `user_id`:
    - Return `None` if `user_id` is `None`
    - Return `None` if `user_id` is not a string
    - Otherwise:
      - Generate a Session ID using the `uuid` module and `uuid4()` like `id` in Base
      - Use this Session ID as the key of the dictionary `user_id_by_session_id` - the value for this key must be `user_id`
      - Return the Session ID
  - The same `user_id` can have multiple Session IDs - indeed, the `user_id` is the value in the dictionary `user_id_by_session_id`
- Now you have an “in-memory” Session ID storing. You will be able to retrieve a User ID based on a Session ID.

### 3. User ID for Session ID
- Update the `SessionAuth` class:
  - Create an instance method `def user_id_for_session_id(self, session_id: str = None) -> str` that returns a User ID based on a Session ID:
    - Return `None` if `session_id` is `None`
    - Return `None` if `session_id` is not a string
    - Return the value (the User ID) for the key `session_id` in the dictionary `user_id_by_session_id`.
    - You must use `.get()` built-in for accessing a value in a dictionary based on a key
- Now you have 2 methods (`create_session` and `user_id_for_session_id`) for storing and retrieving a link between a User ID and a Session ID.
### Session Cookie Documentation

#### 4. Session Cookie

**Mandatory**

Update `api/v1/auth/auth.py` by adding the method `def session_cookie(self, request=None):` that returns a cookie value from a request:

- Return `None` if `request` is `None`
- Return the value of the cookie named `_my_session_id` from `request` - the name of the cookie must be defined by the environment variable `SESSION_NAME`
- You must use `.get()` built-in for accessing the cookie in the request cookies dictionary
- You must use the environment variable `SESSION_NAME` to define the name of the cookie used for the Session ID

**Example Usage:**

In the first terminal:

```bash
bob@dylan:~$ cat main_3.py
#!/usr/bin/env python3
""" Cookie server
"""
from flask import Flask, request
from api.v1.auth.auth import Auth

auth = Auth()

app = Flask(__name__)

@app.route('/', methods=['GET'], strict_slashes=False)
def root_path():
    """ Root path
    """
    return "Cookie value: {}\n".format(auth.session_cookie(request))

if __name__ == "__main__":
    app.run(host="0.0.0.0", port="5000")

bob@dylan:~$ API_HOST=0.0.0.0 API_PORT=5000 AUTH_TYPE=session_auth SESSION_NAME=_my_session_id ./main_3.py 
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
....
```

In a second terminal:

```bash
bob@dylan:~$ curl "http://0.0.0.0:5000"
Cookie value: None
bob@dylan:~$
bob@dylan:~$ curl "http://0.0.0.0:5000" --cookie "_my_session_id=Hello"
Cookie value: Hello
bob@dylan:~$
bob@dylan:~$ curl "http://0.0.0.0:5000" --cookie "_my_session_id=C is fun"
Cookie value: C is fun
bob@dylan:~$
bob@dylan:~$ curl "http://0.0.0.0:5000" --cookie "_my_session_id_fake"
Cookie value: None
bob@dylan:~$
```

**Repository:**

- GitHub repository: `alx-backend-user-data`
- Directory: [`0x02-Session_authentication`](command:_github.copilot.openRelativePath?%5B%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fmedia%2Ftosh%2Fdisk%202%2FLearn%2FALX%2Fspecialization%2Falx-backend-user-data%2F0x02-Session_authentication%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%5D "/media/tosh/disk 2/Learn/ALX/specialization/alx-backend-user-data/0x02-Session_authentication")
- File: `api/v1/auth/auth.py`

#### 5. Before Request

**Mandatory**

Update the `@app.before_request` method in `api/v1/app.py`:

- Add the URL path `/api/v1/auth_session/login/` in the list of excluded paths of the method `require_auth` - this route doesn’t exist yet but it should be accessible outside authentication
- If `auth.authorization_header(request)` and `auth.session_cookie(request)` return `None`, abort(401)

**Example Usage:**

In the first terminal:

```bash
bob@dylan:~$ API_HOST=0.0.0.0 API_PORT=5000 AUTH_TYPE=session_auth SESSION_NAME=_my_session_id python3 -m api.v1.app
 * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)
....
```

In a second terminal:

```bash
bob@dylan:~$ curl "http://0.0.0.0:5000/api/v1/status"
{
  "status": "OK"
}
bob@dylan:~$
bob@dylan:~$ curl "http://0.0.0.0:5000/api/v1/auth_session/login" # not found but not "blocked" by an authentication system
{
  "error": "Not found"
}
bob@dylan:~$
bob@dylan:~$ curl "http://0.0.0.0:5000/api/v1/users/me"
{
  "error": "Unauthorized"
}
bob@dylan:~$ curl "http://0.0.0.0:5000/api/v1/users/me

"

 -H "Authorization: Basic Ym9iQGhidG4uaW86SDBsYmVydG9uU2Nob29sOTgh" # Won't work because the environment variable AUTH_TYPE is equal to "session_auth"
{
  "error": "Forbidden"
}
bob@dylan:~$
bob@dylan:~$ curl "http://0.0.0.0:5000/api/v1/users/me" --cookie "_my_session_id=5535d4d7-3d77-4d06-8281-495dc3acfe76" # Won't work because no user is linked to this Session ID
{
  "error": "Forbidden"
}
bob@dylan:~$
```

**Repository:**

- GitHub repository: `alx-backend-user-data`
- Directory: [`0x02-Session_authentication`](command:_github.copilot.openRelativePath?%5B%7B%22scheme%22%3A%22file%22%2C%22authority%22%3A%22%22%2C%22path%22%3A%22%2Fmedia%2Ftosh%2Fdisk%202%2FLearn%2FALX%2Fspecialization%2Falx-backend-user-data%2F0x02-Session_authentication%22%2C%22query%22%3A%22%22%2C%22fragment%22%3A%22%22%7D%5D "/media/tosh/disk 2/Learn/ALX/specialization/alx-backend-user-data/0x02-Session_authentication")
- File: `api/v1/app.py`