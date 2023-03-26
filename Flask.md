# Flask notes

Base file with multiple paths:
```
from flask import Flask
application = Flask(__name__)

@application.route("/")
def path1():
  return "Hello World!!"

@application.route("/test")
def path2():
  return "Hello, test!!"

if __name__ == "__main__":
    application.run(host='0.0.0.0', port='8080')
```


### Multiple files:
app.py:
```
from flask import Flask, request, jsonify

application = Flask(__name__)

if __name__ == "__main__":
    application.run(host='0.0.0.0', port='8080')

import api1
```

api1.py:
```
from flask import Flask, request, jsonify
from app import application


countries = [
  {"id": 1, "name": "Thailand", "capital": "Bangkok", "area": 513120},
  {"id": 2, "name": "Australia", "capital": "Canberra", "area": 7617930},
  {"id": 3, "name": "Egypt", "capital": "Cairo", "area": 1010408},
]

def _find_next_id():
  return max(country["id"] for country in countries) + 1

@application.get("/api/countries")
def get_countries():
  return jsonify(countries)

@application.post("/api/countries")
def add_country():
  if request.is_json:
      country = request.get_json()
      country["id"] = _find_next_id()
      countries.append(country)
      return country, 201
  return {"error": "Request must be JSON"}, 415

```

### Return static file

```
from flask import send_from_directory
@application.route('/page1')
def send_report():
  return send_from_directory('path/to/dir/', 'index.html')

```
