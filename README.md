MEAN Stack Book Management Application
This project is a simple Book Register web application deployed on an AWS EC2 Ubuntu instance using the MEAN stack (MongoDB, Express, AngularJS, Node.js). It allows users to add, view, and delete book records (Name, ISBN, Author, and Pages) via a web interface.

🚀 Live Application
Public URL: http://32.199.177.142:3300

Instance ID: i-077e7ff40cc2b28e1

📋 Table of Contents
Prerequisites

Step 1: Install Node.js

Step 2: Install MongoDB

Step 3: Install Express and Set Up Routes

Step 4: Configure AngularJS Frontend

Step 5: Run and Verify the Application

🛠️ Installation & Deployment Steps
Step 0: Prerequisites
An active AWS Account.

An EC2 Instance initialized with Ubuntu Server 24.04 LTS.

Inbound Security Group rule allowing traffic on TCP Port 3300.

Step 1: Install Node.js
Update the server packages and install Node.js (v22) to handle server runtime orchestration.

Bash
# Update and upgrade Ubuntu packages
sudo apt update && sudo apt upgrade -y

# Add certificates and dependencies
sudo apt -y install curl dirmngr apt-transport-https lsb-release ca-certificates

# Fetch NodeSource setup script for Node v22
curl -sL https://deb.nodesource.com/setup_22.x | sudo -E bash -

# Install NodeJS
sudo apt install -y nodejs
Step 2: Install MongoDB
Set up the official MongoDB repository and initialize the database engine.

Bash
# Install GPG keys
sudo apt-get install -y gnupg curl
curl -fsSL https://www.mongodb.org/static/pgp/server-7.0.asc | \
   sudo gpg -o /usr/share/keyrings/mongodb-server-7.0.gpg --dearmor

# Add the MongoDB repository list
echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-7.0.gpg ] https://repo.mongodb.org/apt/ubuntu jammy/mongodb-org/7.0 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-7.0.list

# Install MongoDB and npm
sudo apt-get update
sudo apt-get install -y mongodb-org npm

# Start and enable MongoDB service
sudo systemctl start mongod
sudo systemctl enable mongod
Step 3: Install Express and Set Up Routes
Initialize the project directory, download dependencies, and build the backend API endpoints.

Bash
# Install global body-parser dependency
sudo npm install body-parser

# Create project root directory
mkdir Books && cd Books
npm init -y
sudo npm install express mongoose
Create the core web server entrypoint file (server.js):

JavaScript
// server.js
const express = require('express');
const bodyParser = require('body-parser');
const mongoose = require('mongoose');
const path = require('path');

const app = express();
const PORT = process.env.PORT || 3300;

mongoose.connect('mongodb://localhost:27017/test', {})
.then(() => console.log('MongoDB connected'))
.catch(err => console.error('MongoDB connection error:', err));

app.use(express.static(path.join(__dirname, 'public')));
app.use(bodyParser.json());

require('./apps/routes')(app);

app.listen(PORT, () => {
  console.log(`Server up: http://localhost:${PORT}`);
});
Create the application routing file (apps/routes.js) using Express 5 compatible wildcard syntax:

JavaScript
// apps/routes.js
const Book = require('./models/book');
const path = require('path');

module.exports = function(app) {
  app.get('/book', async (req, res) => {
    try {
      const books = await Book.find();
      res.json(books);
    } catch (err) {
      res.status(500).json({ message: 'Error fetching books', error: err.message });
    }
  });

  app.post('/book', async (req, res) => {
    try {
      const book = new Book({
        name: req.body.name,
        isbn: req.body.isbn,
        author: req.body.author,
        pages: req.body.pages
      });
      const savedBook = await book.save();
      res.status(201).json({ message: 'Successfully added book', book: savedBook });
    } catch (err) {
      res.status(400).json({ message: 'Error adding book', error: err.message });
    }
  });

  app.delete('/book/:isbn', async (req, res) => {
    try {
      const result = await Book.findOneAndDelete({ isbn: req.params.isbn });
      if (!result) return res.status(404).json({ message: 'Book not found' });
      res.json({ message: 'Successfully deleted the book', book: result });
    } catch (err) {
      res.status(500).json({ message: 'Error deleting book', error: err.message });
    }
  });

  // Updated wildcard syntax to comply with path-to-regexp v8
  app.get('/*any', (req, res) => {
    res.sendFile(path.join(__dirname, '../public', 'index.html'));
  });
};
Define the data layout schema (apps/models/book.js):

JavaScript
// apps/models/book.js
const mongoose = require('mongoose');

const bookSchema = new mongoose.Schema({
  name: { type: String, required: true },
  isbn: { type: String, required: true, unique: true, index: true },
  author: { type: String, required: true },
  pages: { type: Number, required: true, min: 1 }
}, { timestamps: true });

module.exports = mongoose.model('Book', bookSchema);
Step 4: Configure AngularJS Frontend
Create a client UI folder named public to serve your assets.

Bash
mkdir -p public && cd public
Create the application module state controller (public/script.js):

JavaScript
// public/script.js
angular.module('myApp', [])
  .controller('myCtrl', function($scope, $http) {
    function fetchBooks() {
      $http.get('/book').then(response => { $scope.books = response.data; });
    }
    fetchBooks();

    $scope.del_book = function(book) {
      $http.delete(`/book/${book.isbn}`).then(() => { fetchBooks(); });
    };

    $scope.add_book = function() {
      const newBook = { name: $scope.Name, isbn: $scope.Isbn, author: $scope.Author, pages: $scope.Pages };
      $http.post('/book', newBook).then(() => {
        fetchBooks();
        $scope.Name = $scope.Isbn = $scope.Author = $scope.Pages = '';
      });
    };
  });
Create the structural document structure (public/index.html):

HTML
<!DOCTYPE html>
<html ng-app="myApp" ng-controller="myCtrl">
<head>
  <meta charset="UTF-8">
  <title>Book Management</title>
  <script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.8.2/angular.min.js"></script>
  <script src="script.js"></script>
</head>
<body>
  <h1>Book Management Dashboard</h1>
  </body>
</html>
Step 5: Run and Verify the Application
Return to your project root folder and execute the deployment process.

Bash
cd ~/Books
node server.js
Verify that the console confirms active connectivity:



Terminal Server Up Output: https://github.com/amarsaleem333/project4/blob/main/Server%20up%20Screen%20.png

Browser Management UI View: https://github.com/amarsaleem333/project4/blob/main/books%20final%20look%20as%20web%20interface%20after%20add%20a%20book%20.png
