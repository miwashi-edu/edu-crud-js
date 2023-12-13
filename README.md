# edu-crud-js

## Instructions


```bash
pip3 install Flask
pip3 install flask-swagger-ui
cd ~
cd ws
mkdir crud-js-flask
cd crud-js-flask
mkdir -p ./src/{static,controllers,domain}
touch ./src/{server.py,controllers/book_controller.py,domain/books.py}
#python3 -m venv venv
source venv/bin/activate
echo "Flask" > requirements.txt
```

## ./src/server.js

```bash
cat > ./src/server.py << 'EOF'
from flask import Flask, jsonify, request
from src.controllers import book_controller

app = Flask(__name__)

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

## ./src/controllers/book_controller.py

```bash
cat > ./src/controllers/book_controller.py << 'EOF'
from src.domain import books

def list_all_books():
    return books.list_all_books(), 200

def get_book_by_id(book_id):
    book = books.get_book_by_id(book_id)
    if book:
        return book, 200
    else:
        return {'message': 'Book not found'}, 404

def create_book(book_data):
    new_book = books.create_book(book_data)
    return new_book, 201

def update_book(book_id, book_data):
    updated_book = books.update_book(book_id, book_data)
    if updated_book:
        return updated_book, 200
    else:
        return {'message': 'Book not found'}, 404

def delete_book(book_id):
    deleted_book = books.delete_book(book_id)
    if deleted_book:
        return deleted_book, 200
    else:
        return {'message': 'Book not found'}, 404

EOF
```

## ./src/domain/books.py

```bash
cat > ./src/domain/books.py << 'EOF'
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
cat > ./static/swagger.yaml << 'EOF'
swagger: "2.0"
info:
  title: "Books API"
  description: "API for managing books"
  version: "1.0.0"
host: "localhost:5000"
basePath: "/"
schemes:
  - "http"
paths:
  /books:
    get:
      summary: "List all books"
      responses:
        200:
          description: "A list of books"
EOF
```
