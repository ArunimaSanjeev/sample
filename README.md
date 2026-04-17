for v3

from flask import Flask, request, jsonify
app = Flask(__name__)

students = []

@app.route('/add', methods=['POST'])
def add():
    data = request.json
    students.append(data)
    return jsonify(students)

@app.route('/students', methods=['GET'])
def view():
    return jsonify(students)

app.run(host='0.0.0.0', port=5003)
-----------------------------------------
courses = {}

@app.route('/assign', methods=['POST'])
def assign():
    data = request.json
    name = data.get('name')
    course = data.get('course')

    courses[name] = course
    return jsonify(courses)
-----------------------------------------
@app.route('/remove', methods=['POST'])
def remove():
    data = request.json
    name = data.get('name')

    courses.pop(name, None)
    return jsonify(courses)
-------------------------------------
version 1 without flask
import http.server
import socketserver
import json

PORT = 5000
books = []

class Handler(http.server.BaseHTTPRequestHandler):

    def do_GET(self):
        if self.path == "/books":
            self.send_response(200)
            self.send_header("Content-type", "application/json")
            self.end_headers()
            self.wfile.write(json.dumps(books).encode())

    def do_POST(self):
        if self.path == "/add":
            content_length = int(self.headers['Content-Length'])
            post_data = self.rfile.read(content_length)
            book = json.loads(post_data)
            books.append(book)

            self.send_response(200)
            self.end_headers()
            self.wfile.write(json.dumps(books).encode())

with socketserver.TCPServer(("", PORT), Handler) as httpd:
    print("Running on port", PORT)
    httpd.serve_forever()

  -------------------------
  version 2 wihtout flask

  import http.server
import socketserver
import json

PORT = 5000
books = []
issued = {}

class Handler(http.server.BaseHTTPRequestHandler):

    def do_GET(self):
        if self.path == "/books":
            self.send_response(200)
            self.end_headers()
            self.wfile.write(json.dumps(books).encode())

        elif self.path == "/issued":
            self.send_response(200)
            self.end_headers()
            self.wfile.write(json.dumps(issued).encode())

    def do_POST(self):
        content_length = int(self.headers['Content-Length'])
        post_data = self.rfile.read(content_length)
        data = json.loads(post_data)

        if self.path == "/add":
            books.append(data)

        elif self.path == "/issue":
            book = data.get("book")
            student = data.get("student")
            issued[book] = student

        self.send_response(200)
        self.end_headers()
        self.wfile.write(json.dumps({"status": "ok"}).encode())

with socketserver.TCPServer(("", PORT), Handler) as httpd:
    print("Running...")
    httpd.serve_forever()
--------------------------------------
version 3

import http.server
import socketserver
import json

PORT = 5000
books = []
issued = {}

class Handler(http.server.BaseHTTPRequestHandler):

    def do_GET(self):
        if self.path == "/books":
            self.send_response(200)
            self.end_headers()
            self.wfile.write(json.dumps(books).encode())

        elif self.path == "/issued":
            self.send_response(200)
            self.end_headers()
            self.wfile.write(json.dumps(issued).encode())

    def do_POST(self):
        content_length = int(self.headers['Content-Length'])
        post_data = self.rfile.read(content_length)
        data = json.loads(post_data)

        if self.path == "/add":
            books.append(data)

        elif self.path == "/issue":
            issued[data.get("book")] = data.get("student")

        elif self.path == "/return":
            issued.pop(data.get("book"), None)

        self.send_response(200)
        self.end_headers()
        self.wfile.write(json.dumps({"status": "ok"}).encode())

with socketserver.TCPServer(("", PORT), Handler) as httpd:
    print("Running...")
    httpd.serve_forever()
------------------------------
curl -X POST http://localhost:5000/add \
-H "Content-Type: application/json" \
-d '{"name":"DSA"}'

curl http://localhost:5000/books

curl -X POST http://localhost:5000/issue \
-H "Content-Type: application/json" \
-d '{"book":"DSA","student":"John"}'

curl -X POST http://localhost:5000/return \
-H "Content-Type: application/json" \
-d '{"book":"DSA"}'
  ---------------------------
  dockerfile
FROM python:3.11
WORKDIR /app
COPY . .
CMD ["python", "app.py"]

docker build -t username/library:v1 ./v1

docker push username/library:v1
docker push username/library:v2
docker push username/library:v3
----------------------------
yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: library-v1
spec:
  replicas: 1
  selector:
    matchLabels:
      app: library-v1
  template:
    metadata:
      labels:
        app: library-v1
    spec:
      containers:
      - name: library
        image: username/library:v1
        ports:
        - containerPort: 5000

---
apiVersion: v1
kind: Service
metadata:
  name: library-v1-service
spec:
  type: NodePort
  selector:
    app: library-v1
  ports:
    - port: 5000
      targetPort: 5000
      nodePort: 30001
    ---------------------
    yaml commands

kubectl apply -f v1.yaml
kubectl apply -f v2.yaml
kubectl apply -f v3.yaml
