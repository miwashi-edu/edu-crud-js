# edu-crud-js

## Instructions

```bash
cd ~
cd ws
mkdir crud-js
cd crud-js
mkdir -p ./src/{routes,controllers,domain}
touch ./src/{service.js,server.js,/routes/book_routes.js,/controllers/book_controller.js,/domain/books.js}
npm init -y
npm install express
npm install swagger-ui-express swagger-jsdoc
npm pkg set scripts.start="node ./src/service.js"
npm pkg set scripts.dev="node --watch ./src/service.js"
```

## ./src/service.js

```bash
cat > ./src/service.js << 'EOF'
const app = require('./server');
const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
    console.log(`Server started on port ${PORT}`);
});
EOF
```

## ./src/server.js

```bash
cat > ./src/server.js << 'EOF'
const express = require('express');
const swaggerUi = require('swagger-ui-express');
const swaggerJsdoc = require('swagger-jsdoc');

// Swagger definition
const swaggerOptions = {
  swaggerDefinition: {
    openapi: '3.0.0',
    info: {
      title: 'Books API',
      version: '1.0.0',
      description: 'Simple API to manage books',
    },
    servers: [
      {
        url: `http://localhost:3000`,
      },
    ],
  },
  apis: ['./routes/*.js'], // path to the API docs
};
const swaggerSpec = swaggerJsdoc(swaggerOptions);

const app = express();
app.use(express.json());
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec));
app.use('/books', require('./routes/book_routes'));
module.exports = app;
EOF
```

## ./src/routes/book_routes.js

```bash
cat > ./src/routes/book_routes.js << 'EOF'
const express = require('express');
const bookController = require('../controllers/book_controller');
const router = express.Router();

/**
 * @swagger
 * components:
 *   schemas:
 *     Book:
 *       type: object
 *       required:
 *         - id
 *         - title
 *         - author
 *       properties:
 *         id:
 *           type: integer
 *           description: The auto-generated id of the book
 *         title:
 *           type: string
 *           description: The book title
 *         author:
 *           type: string
 *           description: The book author
 *       example:
 *         id: 1
 *         title: Example Book
 *         author: Author Name
 */

/**
 * @swagger
 * /books:
 *   get:
 *     summary: Lists all books
 *     responses:
 *       200:
 *         description: A list of books
 *         content:
 *           application/json:
 *             schema:
 *               type: array
 *               items:
 *                 $ref: '#/components/schemas/Book'
 */
router.get('/', bookController.listAllBooks);

/**
 * @swagger
 * /books/{bookId}:
 *   get:
 *     summary: Retrieves the book based on their ID
 *     parameters:
 *       - in: path
 *         name: bookId
 *         schema:
 *           type: integer
 *         required: true
 *     responses:
 *       200:
 *         description: A book object
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Book'
 */
router.get('/:bookId', bookController.getBookById);

/**
 * @swagger
 * /books:
 *   post:
 *     summary: Creates a book
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/Book'
 *     responses:
 *       201:
 *         description: The created book object
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Book'
 */
router.post('/', bookController.createBook);

/**
 * @swagger
 * /books/{bookId}:
 *   put:
 *     summary: Updates a book by their ID
 *     parameters:
 *       - in: path
 *         name: bookId
 *         schema:
 *           type: integer
 *         required: true
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/Book'
 *     responses:
 *       200:
 *         description: The updated book object
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Book'
 */
router.put('/:bookId', bookController.updateBook);

/**
 * @swagger
 * /books/{bookId}:
 *   delete:
 *     summary: Deletes a book based on their ID
 *     parameters:
 *       - in: path
 *         name: bookId
 *         schema:
 *           type: integer
 *         required: true
 *     responses:
 *       200:
 *         description: The deleted book object
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Book'
 */
router.delete('/:bookId', bookController.deleteBook);

module.exports = router;
EOF
```

## ./src/controllers/book_controller.js

```bash
cat > ./src/controllers/book_controller.js << 'EOF'
const booksDomain = require('../domain/books');

exports.listAllBooks = (req, res) => {
    res.json(booksDomain.listAllBooks());
};

exports.getBookById = (req, res) => {
    const bookId = parseInt(req.params.bookId, 10);
    const book = booksDomain.getBookById(bookId);

    if (book) {
        res.json(book);
    } else {
        res.status(404).send({ message: 'Book not found' });
    }
};

exports.createBook = (req, res) => {
    const newBook = booksDomain.createBook(req.body);
    res.status(201).json(newBook);
};

exports.updateBook = (req, res) => {
    const bookId = parseInt(req.params.bookId, 10);
    const updatedBook = booksDomain.updateBook(bookId, req.body);

    if (updatedBook) {
        res.json(updatedBook);
    } else {
        res.status(404).send({ message: 'Book not found' });
    }
};

exports.deleteBook = (req, res) => {
    const bookId = parseInt(req.params.bookId, 10);
    const deletedBook = booksDomain.deleteBook(bookId);

    if (deletedBook) {
        res.json(deletedBook);
    } else {
        res.status(404).send({ message: 'Book not found' });
    }
};

EOF
```

## ./src/domain/books.js

```bash
cat > ./src/domain/books.js << 'EOF'
let books = [
    {
        id: 1,
        title: "Example Book 1",
        author: "Author 1"
    },
    {
        id: 2,
        title: "Example Book 2",
        author: "Author 2"
    }
    // ... other books ...
];

function listAllBooks() {
    return books;
}

function getBookById(bookId) {
    return books.find(b => b.id === bookId);
}

function createBook(newBook) {
    newBook.id = books.length + 1;  // Simple ID generation
    books.push(newBook);
    return newBook;
}

function updateBook(bookId, updatedBook) {
    const index = books.findIndex(b => b.id === bookId);

    if (index !== -1) {
        books[index] = { ...books[index], ...updatedBook };
        return books[index];
    }
    return null;
}

function deleteBook(bookId) {
    const index = books.findIndex(b => b.id === bookId);
    
    if (index !== -1) {
        return books.splice(index, 1)[0];
    }
    return null;
}

module.exports = {
    listAllBooks,
    getBookById,
    createBook,
    updateBook,
    deleteBook
};
EOF
```
