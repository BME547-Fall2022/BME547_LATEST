# Web Server Development With `Flask`
`flask` is a Python package for small-scale web server development.

### Example
```
from flask import Flask, jsonify

app = Flask(__name__)

@app.route("/", methods=["GET"])
def server_on():
    return "Ideal Weight Server is On"

@app.route("/info", methods=["GET"])
def information():
    info_string = ("This server calculates the ideal weight of a person"
                  " based on their height, age, and gender.")
    calc_info_string = ("The calculation can be accessed by POST to the "
                        "'/calculate_iwc' endpoint with a JSON containng "
                        "{'age': age, 'height_in': height_inches, "
                        "'gender': male_or_female}")
    out_dictionary = {"info": info_string, "calc_info": calc_info_string}
    return jsonify(out_dictionary)
                 
if __name__ == '__main__':
    app.run()
```

What did the above do?

`from flask import Flask jsonify` imports the `Flask` class and `jsonify`
function from the `flask` package.

`app = Flask(__name__)`
creates an instance of the Flask class that will run the server.  It is 
supplied with the name of the current running module.

The above server code has two API endpoints:
* `GET /'` which returns the server status as a string
* `GET /info` which returns information about the server in a JSON string.

`@app.route("/", methods=["GET"]) `
is a decorator that tells the Flask server to attach the following procedure
to the route given by the first parameter.  In this case, the route is simply
the `"/"` path.  This route is accessed using GET requests.

`def server_on():` is the function that will be run when the '"/"' path
is sent a request.  In this case, the function returns a string intended to
let the client know that the server is active.

`@app.route("/info", methods=["GET"])`
Similar to the previous decorator, it is attaching the procedure on the next
line to the route `/info`.

`def information():`
is the function that will run when `/info` receives a GET request.  It
returns a JSON string of the dictionary defined in the procedure.  The
`flask` function `jsonify` encodes the given variable into a JSON string
to be sent over the web.

`app.run()` starts the server.  

When the program is run, you will see something like the following in the
console:
```
FLASK_APP = iwc_server.py
FLASK_ENV = development
FLASK_DEBUG = 0
In folder C:/Users/dwonl/repos/WebServerLectures
C:\Users\dwonl\repos\WebServerLectures\venv\Scripts\python.exe -m flask run
 * Serving Flask app "iwc_server.py"
 * Environment: development
 * Debug mode: off
 * Running on http://127.0.0.1:5000/ (Press CTRL+C to quit)
 ```
The server has started on your local computer and can be accessed through
the URL `http://127.0.0.1:5000`.  The IP adress of `127.0.0.1` is called the
loopback address or localhost address.  This establishes an IP connection
to the local computer.

To access the server, you could do one of two things:
1.  In a browser, visit `http://127.0.0.1:5000/info`, or
2.  make a request such as follows:  
  ```
    import requests

    r = requests.get("http://127.0.0.1:5000/info")
    print(r.text)
```
Next, lets add another route to our server.  First, we must import an
additional item from flask:  the `request` object.
```
from flask import Flask, jsonify, request
```
Then, here is the function to add a third route to our server:
```
@app.route("/calculate_iwc", methods=["POST"])
def calculate_iwc():
    in_data = request.get_json()
    age = float(in_data["age"])
    gender = in_data["gender"]
    height = float(in_data["height_in"])

    ideal_weight_kg = 48.0 + 2.7 * (height - 60)
    ideal_weight_lb = ideal_weight_kg * 2.20462
    return jsonify({"input data": in_data, 
                    "ideal weight in lb": ideal_weight_lb})
```
The `@app.route` decorator defines that the URL path `/calculate_iwc` will
run the function below it called `calculate_iwc`.  Note that in this case,
the route name and function name are the same, but they do not have to be.
The decorator also defines that this route is accessed using POST requests.

Inside the function, the line `in_data = request.get_json()` uses the `flask`
object `request` to access the data accompanying the POST request.  In this
case, the JSON associated with the POST is copied into the variable `in_data`.
The remainder of the procedure calculates the ideal weight based on the given
input data.  

Finally, the `flask` function `jsonify()` is used to serialize the answer
into a JSON string for transmittal back to the client.  In this case, the
function defines a dictionary with the input variables and the result.

#### `jsonify` Important Usage Note
`jsonify()` is not a function that can be used anywhere to create a 
JSON-encoded string.  It will only work properly if used with a `return` 
statement of a Flask handler function (a function decorated with `@app.route`).

An example of proper usage:
```python
def get_result(a):
    # Code to do work
    return result

@app.route("/get_result", methods=["POST"])
    a = request.get_json()
    result = get_result(a)
    return jsonify(result)
```
An example that will not work:  
```python
def get_result(a):
    # Code to do work
    return result

@app.route("/get_result", methods=["POST"])
    a = request.get_json()
    result = get_result(a)
    result_json = jsonify(result)  # Do not use jsonify without a return in a flask handler
    return result_json
```

Another example that will not work:
```python
def get_result(a):
    # Code to do work
    return jsonify(result)  # Do not use jsonify in a non-flask handler 

@app.route("/get_result", methods=["POST"])
    a = request.get_json()
    result = get_result(a)
    return jsonify(result)
```


### Variable URLs
You can send information to a web server by using a variable name in
a URL.  Here is an example:
```
@app.route("/sayhello/<name>", methods=["GET"])
def say_hello(name):
    hello_string = "Hello {}".format(name)
    return hello_string
```
The part of the URL that should contain variable information is defined
using `<>` in the URL.  Then, whatever variable name is put inside the `<>`
should then be added as an argument or parameter in the accompanying function.

An example of calling the above route is given here:
```
r = requests.get("Http://127.0.0.1:5000/sayhello/David")
print(r.text)

OUTPUT: 'Hello David'
```
Here, `David` was included as part of the URL in the position of the `<name>`
variable.  Therefore, the `name` argument to the `say_hello` function
contains the value `David`.  

## Running External Server
In the examples above, the `flask` server is being hosted on the local
computer with a URL of `http://127.0.0.1:5000`.  This is unaccessible from
the general web.  To allow the server to be accessed by other computers
across the web, we need to set the host as follows:
```
app.run(host="0.0.0.0")
```
The IP address `0.0.0.0` refers to all of the IPv4 addresses of the server's
computer.  By running the program on that host, the server program can
be accessed from the internet by the IP address or URL of the local computer.

For the Duke VMs used in this class, the URL to access your server would be
of the format `http://vcm-####.vm.duke.edu:5000`.  
