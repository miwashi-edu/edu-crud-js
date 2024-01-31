# edu-crud-js

## Instructions


```bash
pip3 install Flask
pip3 install flask-swagger-ui
pip3 install requests
cd ~
cd ws
mkdir crud-js-flask
cd crud-js-flask
mkdir ./{static,controllers,domain}
touch ./{server.py,controllers/book_controller.py,domain/books.py}
touch ./{controllers/__init__.py,domain/__init__.py}
#python3 -m venv venv
#source venv/bin/activate
echo "Flask" > requirements.txt
echo "flask-swagger-ui" >> requirements.txt
```

## ./server.js

```bash
cd ~
cd ws
cd crud-js-flask
cat > ./server.py << 'EOF'
from flask import Flask, jsonify, request, send_from_directory
from flask_swagger_ui import get_swaggerui_blueprint
from controllers import book_controller

app = Flask(__name__)

# Swagger UI configuration
SWAGGER_URL = '/swagger'  # URL for exposing Swagger UI (without trailing '/')
API_URL = '/static/swagger.yaml'  # URL for your Swagger specification

swaggerui_blueprint = get_swaggerui_blueprint(
    SWAGGER_URL,
    API_URL,
    config={
        'app_name': "Books API"
    }
)

app.register_blueprint(swaggerui_blueprint, url_prefix=SWAGGER_URL)

@app.route('/static/<path:path>')
def send_static(path):
    """
    Route for serving static files (like your Swagger specification).
    """
    return send_from_directory('static', path)

# Existing routes

# Route to list all books
@app.route('/books', methods=['GET'])
def list_all_books():
    return jsonify(book_controller.list_all_books())

# Route to get a book by ID
@app.route('/books/<int:book_id>', methods=['GET'])
def get_book_by_id(book_id):
    return book_controller.get_book_by_id(book_id)

# Route to create a new book
@app.route('/books', methods=['POST'])
def create_book():
    return book_controller.create_book(request.json)

# Route to update a book
@app.route('/books/<int:book_id>', methods=['PUT'])
def update_book(book_id):
    return book_controller.update_book(book_id, request.json)

# Route to delete a book
@app.route('/books/<int:book_id>', methods=['DELETE'])
def delete_book(book_id):
    return book_controller.delete_book(book_id)

if __name__ == '__main__':
    app.run(debug=True, port=3000)
EOF
```

## ./controllers/book_controller.py

```bash
cd ~
cd ws
cd crud-js-flask
cat > ./controllers/book_controller.py << 'EOF'
from domain import books


def list_all_books():
    # No need to return status code here as it defaults to 200
    return books.list_all_books()


def get_book_by_id(book_id):
    book = books.get_book_by_id(book_id)
    if book:
        return book
    else:
        return {'message': 'Book not found'}, 404


def create_book(book_data):
    new_book = books.create_book(book_data)
    return new_book, 201


def update_book(book_id, book_data):
    updated_book = books.update_book(book_id, book_data)
    if updated_book:
        return updated_book
    else:
        return {'message': 'Book not found'}, 404
  
  
def delete_book(book_id):
    deleted_book = books.delete_book(book_id)
    if deleted_book:
        return deleted_book
    else:
        return {'message': 'Book not found'}, 404
EOF
```

## ./domain/books.py

```bash
cd ~
cd ws
cd crud-js-flask
cat > ./domain/books.py << 'EOF'
class Book:
    def __init__(self, id, title, author):
        self.id = id
        self.title = title
        self.author = author

# This will act as our in-memory database for simplicity
books_db = [
    Book(1, "Example Book One", "Author A"),
    Book(2, "Example Book Two", "Author B")
]

def list_all_books():
    return [book.__dict__ for book in books_db]


def get_book_by_id(book_id):
    for book in books_db:
        if book.id == book_id:
            return book.__dict__
    return None



def create_book(book_data):
    new_book_id = max(book.id for book in books_db) + 1
    new_book = Book(new_book_id, book_data['title'], book_data['author'])
    books_db.append(new_book)
    return new_book.__dict__



def update_book(book_id, book_data):
    for book in books_db:
        if book.id == book_id:
            book.title = book_data.get('title', book.title)
            book.author = book_data.get('author', book.author)
            return book.__dict__
    return None



def delete_book(book_id):
    global books_db
    book_to_delete = next((book for book in books_db if book.id == book_id), None)
    if book_to_delete:
        books_db = [book for book in books_db if book.id != book_id]
        return book_to_delete.__dict__
    return None
EOF
```

## ./static/swagger.yaml

```bash
cd ~
cd ws
cd crud-js-flask
cat > ./static/swagger.yaml << 'EOF'
swagger: "2.0"
info:
  title: "Books API"
  description: "API for managing books"
  version: "1.0.0"
host: "localhost:3000"
basePath: "/"
schemes:
  - "http"

paths:
  /books:
    get:
      summary: "List all books"
      description: "Retrieve a list of all books"
      responses:
        200:
          description: "A list of books"
          schema:
            type: "array"
            items:
              $ref: "#/definitions/Book"
    post:
      summary: "Create a new book"
      description: "Add a new book to the collection"
      parameters:
        - in: "body"
          name: "book"
          description: "Book to add"
          required: true
          schema:
            $ref: "#/definitions/Book"
      responses:
        201:
          description: "Book created"

  /books/{book_id}:
    get:
      summary: "Get a book by ID"
      description: "Retrieve a book by its ID"
      parameters:
        - in: "path"
          name: "book_id"
          type: "integer"
          required: true
          description: "ID of the book to retrieve"
      responses:
        200:
          description: "Desired book"
          schema:
            $ref: "#/definitions/Book"
    put:
      summary: "Update a book"
      description: "Update details of a book by its ID"
      parameters:
        - in: "path"
          name: "book_id"
          type: "integer"
          required: true
          description: "ID of the book to update"
        - in: "body"
          name: "book"
          required: true
          description: "Updated details of the book"
          schema:
            $ref: "#/definitions/Book"
      responses:
        200:
          description: "Book updated"
    delete:
      summary: "Delete a book"
      description: "Delete a book by its ID"
      parameters:
        - in: "path"
          name: "book_id"
          type: "integer"
          required: true
          description: "ID of the book to delete"
      responses:
        204:
          description: "Book deleted"

definitions:
  Book:
    type: "object"
    required:
      - "title"
      - "author"
    properties:
      id:
        type: "integer"
        format: "int64"
      title:
        type: "string"
      author:
        type: "string"
      description:
        type: "string"
      publishedYear:
        type: "integer"
EOF
```

## Test it

```
#!/bin/bash

# List all books
curl -X GET http://localhost:3000/books

# Get a book by ID (replace 1 with the actual book ID)
curl -X GET http://localhost:3000/books/1

# Create a new book (modify the JSON payload as necessary)
curl -X POST http://localhost:3000/books -H "Content-Type: application/json" -d '{"title": "New Book", "author": "Author Name"}'

# Update a book (replace 1 with the book ID and modify the JSON payload as necessary)
curl -X PUT http://localhost:3000/books/1 -H "Content-Type: application/json" -d '{"title": "Updated Book Title", "author": "Updated Author Name"}'

# Delete a book (replace 1 with the book ID to delete)
curl -X DELETE http://localhost:3000/books/1

```

## list_books

```bash
cd ~
cd ws
cd crud-js-flask
echo '#!'"$(which python3)" >  list_books
chmod +x list_books
cat >> list_books << EOF
import requests

class Book:
    def __init__(self, id, title, author):
        self.id = id
        self.title = title
        self.author = author

response = requests.get('http://localhost:3000/books')
# response is a data dictionary ** is an unpack operator and renders the dictionary to id=123, title='My Book', author='John Doe' which you can pass to Book()
books = [Book(**book_data) for book_data in response.json()]
for book in books:
    print(f'ID: {book.id}, Title: {book.title}, Author: {book.author}')
EOF
```

## get_book

```bash
cd ~
cd ws
cd crud-js-flask
echo '#!'"$(which python3)" >  get_book
chmod +x get_book
cat >> get_book << EOF
import requests

class Book:
    def __init__(self, id, title, author):
        self.id = id
        self.title = title
        self.author = author

book_id = input("Enter book ID: ")
response = requests.get(f'http://localhost:3000/books/{book_id}')
book_data = response.json()
book = Book(**book_data)
print(f'ID: {book.id}, Title: {book.title}, Author: {book.author}')
EOF
```

## create_book

```bash
cd ~
cd ws
cd crud-js-flask
echo '#!'"$(which python3)" >  create_book
chmod +x create_book
cat >> create_book << EOF
import requests
import json

class Book:
    def __init__(self, id, title, author):
        self.id = id
        self.title = title
        self.author = author

title = input("Enter book title: ")
author = input("Enter author name: ")

book = Book(None, title, author)  # ID is None for new book
book_data = json.dumps(book.__dict__)
headers = {'Content-Type': 'application/json'}

response = requests.post('http://localhost:3000/books', data=book_data, headers=headers)
print(response.json())
EOF
```

## update_book

```bash
cd ~
cd ws
cd crud-js-flask
echo '#!'"$(which python3)" >  update_book
chmod +x update_book
cat >> update_book << EOF
import requests
import json

class Book:
    def __init__(self, id, title, author):
        self.id = id
        self.title = title
        self.author = author

book_id = input("Enter book ID to update: ")
title = input("Enter new title: ")
author = input("Enter new author: ")

book = Book(book_id, title, author)
book_data = json.dumps(book.__dict__)
headers = {'Content-Type': 'application/json'}

response = requests.put(f'http://localhost:3000/books/{book_id}', data=book_data, headers=headers)
print(response.json())
EOF
```

## delete_book

```bash
cd ~
cd ws
cd crud-js-flask
echo '#!'"$(which python3)" >  delete_book
chmod +x delete_book
cat >> delete_book << EOF
import requests

class Book:
    def __init__(self, id, title, author):
        self.id = id
        self.title = title
        self.author = author

book_id = input("Enter book ID to delete: ")
response = requests.delete(f'http://localhost:3000/books/{book_id}')
print(response.json())
EOF
```
