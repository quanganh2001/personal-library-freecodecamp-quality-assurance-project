# Install package
Type `npm i`, then `npm i mongodb nodemon mongoose`
# Setup mongodb database
Go to mongodb.com, sign in account, then create new project and setup connection.

Setup environment:
```env
PORT=3000
NODE_ENV=
MONGO_URI=
```
## Connect to Mongodb database
```js
const mongoose = require("mongoose");
const db = mongoose.connect(process.env.MONGO_URI, {
  useUnifiedTopology: true,
  useNewUrlParser: true,
});

module.exports = db;
```
Require server: `require("./db-connection.js");`
# API
Setup model:
```js
const mongoose = require("mongoose");
const { Schema } = mongoose;

const BookSchema = new Schema({
  title: { type: String, required: true },
  comments: [String],
});
const Book = mongoose.model("Book", BookSchema);

exports.Book = Book;
```
## POST Request
You can send a **POST** request to `/api/books` with `title` as part of the form data to add a book. The returned response will be an object with the `title` and a unique `_id` as keys. If `title` is not included in the request, the returned response should be the string `missing required field title`.
```js
app
  .post(function (req, res) {
    let title = req.body.title;
    if (!title) {
      res.send("missing required field title");
      return;
    }
    const newBook = new Book({ title, comments: [] });
    newBook.save((err, data) => {
      if (err || !data) {
        res.send("there was an error saving");
      } else {
        res.json({ _id: data._id, title: data.title });
      }
    });
    //response will contain new book object including atleast _id and title
  })
```
You can send a **POST** request containing `comment` as the form body data to `/api/books/{_id}` to add a comment to a book. The returned response will be the books object similar to **GET** `/api/books/{_id}` request in an earlier test. If `comment` is not included in the request, return the string `missing required field comment`. If no book is found, return the string `no book exists`.
```js
app
  .post(function (req, res) {
    let bookid = req.params.id;
    let comment = req.body.comment;
    if (!comment) {
      res.send(`missing required field comment`);
      return;
    }
    //json res format same as .get
    Book.findById(bookid, (err, bookdata) => {
      if (!bookdata) {
        res.send("no book exists");
      } else {
        bookdata.comments.push(comment);
        bookdata.save((err, saveData) => {
          res.json({
            comments: saveData.comments,
            _id: saveData._id,
            title: saveData.title,
            commentcount: saveData.comments.length,
          });
        });
      }
    });
  })
```
## GET Request
You can send a **GET** request to `/api/books` and receive a JSON response representing all the books. The JSON response will be an array of objects with each object (book) containing `title`, `_id`, and `commentcount` properties.
```js
app
  .route("/api/books/:id")
  .get(function (req, res) {
    let bookid = req.params.id;
    //json res format: {"_id": bookid, "title": book_title, "comments": [comment,comment,...]}
    Book.findById(bookid, (err, data) => {
      if (!data) {
        res.send("no book exists");
      } else {
        res.json({
          comments: data.comments,
          _id: data._id,
          title: data.title,
          commentcount: data.comments.length,
        });
      }
    });
  })
```
## DELETE Request
You can send a `DELETE` request to `/api/books` to delete all books in the database. The returned response will be the string `complete delete successful` if successful.
```js
app
  .delete(function (req, res) {
    //if successful response will be 'complete delete successful'
    Book.remove({}, (err, data) => {
      if (err || !data) {
        res.send("error");
      } else {
        res.send("complete delete successful");
      }
    });
  });
```
You can send a DELETE request to `/api/books/{_id}` to delete a book from the collection. The returned response will be the string `delete successful` if successful. If no book is found, return the string `no book exists`.
```js
app
  .route('/api/books/:id')
  .delete(function (req, res) {
    let bookid = req.params.id;
    //if successful response will be 'delete successful'
    Book.findByIdAndRemove(bookid, (err, data) => {
      if (err || !data) {
        res.send("no book exists");
      } else {
        res.send("delete successful");
      }
    });
  });
```
Full source code:
```js
/*
 *
 *
 *       Complete the API routing below
 *
 *
 */

"use strict";
const Book = require("../models").Book;

module.exports = function (app) {
  app
    .route("/api/books")
    .get(function (req, res) {
      console.log("req.body :>> ", req.body);
      //response will be array of book objects
      //json res format: [{"_id": bookid, "title": book_title, "commentcount": num_of_comments },...]
      Book.find({}, (err, data) => {
        if (!data) {
          res.json([]);
        } else {
          const formatData = data.map((book) => {
            return {
              _id: book._id,
              title: book.title,
              comments: book.comments,
              commentcount: book.comments.length,
            };
          });
          res.json(formatData);
        }
      });
    })

    .post(function (req, res) {
      let title = req.body.title;
      if (!title) {
        res.send("missing required field title");
        return;
      }
      const newBook = new Book({ title, comments: [] });
      newBook.save((err, data) => {
        if (err || !data) {
          res.send("there was an error saving");
        } else {
          res.json({ _id: data._id, title: data.title });
        }
      });
      //response will contain new book object including atleast _id and title
    })

    .delete(function (req, res) {
      //if successful response will be 'complete delete successful'
      Book.remove({}, (err, data) => {
        if (err || !data) {
          res.send("error");
        } else {
          res.send("complete delete successful");
        }
      });
    });

  app
    .route("/api/books/:id")
    .get(function (req, res) {
      let bookid = req.params.id;
      //json res format: {"_id": bookid, "title": book_title, "comments": [comment,comment,...]}
      Book.findById(bookid, (err, data) => {
        if (!data) {
          res.send("no book exists");
        } else {
          res.json({
            comments: data.comments,
            _id: data._id,
            title: data.title,
            commentcount: data.comments.length,
          });
        }
      });
    })

    .post(function (req, res) {
      let bookid = req.params.id;
      let comment = req.body.comment;
      if (!comment) {
        res.send(`missing required field comment`);
        return;
      }
      //json res format same as .get
      Book.findById(bookid, (err, bookdata) => {
        if (!bookdata) {
          res.send("no book exists");
        } else {
          bookdata.comments.push(comment);
          bookdata.save((err, saveData) => {
            res.json({
              comments: saveData.comments,
              _id: saveData._id,
              title: saveData.title,
              commentcount: saveData.comments.length,
            });
          });
        }
      });
    })

    .delete(function (req, res) {
      let bookid = req.params.id;
      //if successful response will be 'delete successful'
      Book.findByIdAndRemove(bookid, (err, data) => {
        if (err || !data) {
          res.send("no book exists");
        } else {
          res.send("delete successful");
        }
      });
    });
};
```
# Functional Tests
```js
/*
 *
 *
 *       FILL IN EACH FUNCTIONAL TEST BELOW COMPLETELY
 *       -----[Keep the tests in the same order!]-----
 *
 */

const chaiHttp = require("chai-http");
const chai = require("chai");
const assert = chai.assert;
const server = require("../server");

chai.use(chaiHttp);

let bookID;
suite("Functional Tests", function () {
  /*
   * ----[EXAMPLE TEST]----
   * Each test should completely test the response of the API end-point including response status code!
   */
  test("#example Test GET /api/books", function (done) {
    chai
      .request(server)
      .get("/api/books")
      .end(function (err, res) {
        assert.equal(res.status, 200);
        assert.isArray(res.body, "response should be an array");
        if (!res.body[0]) {
          done();
        }
        assert.property(
          res.body[0],
          "commentcount",
          "Books in array should contain commentcount"
        );
        assert.property(
          res.body[0],
          "title",
          "Books in array should contain title"
        );
        assert.property(
          res.body[0],
          "_id",
          "Books in array should contain _id"
        );
        done();
      });
  });
  /*
   * ----[END of EXAMPLE TEST]----
   */

  suite("Routing tests", function () {
    suite(
      "POST /api/books with title => create book object/expect book object",
      function () {
        test("Test POST /api/books with title", function (done) {
          chai
            .request(server)
            .post("/api/books")
            .send({ title: "test-title" })
            .end(function (err, res) {
              assert.equal(res.status, 200);
              bookID = res.body._id;
              assert.equal(res.body.title, "test-title");
              done();
            });
        });

        test("Test POST /api/books with no title given", function (done) {
          chai
            .request(server)
            .post("/api/books")
            .send({})
            .end(function (err, res) {
              assert.equal(res.status, 200);
              assert.equal(res.text, "missing required field title");
              done();
            });
        });
      }
    );

    suite("GET /api/books => array of books", function () {
      test("Test GET /api/books", function (done) {
        chai
          .request(server)
          .get("/api/books")
          .end(function (err, res) {
            assert.equal(res.status, 200);
            assert.isArray(res.body, "it is an array");
            done();
          });
      });

      suite("GET /api/books/[id] => book object with [id]", function () {
        test("Test GET /api/books/[id] with id not in db", function (done) {
          chai
            .request(server)
            .get("/api/books/invalidID")
            .end(function (err, res) {
              assert.equal(res.status, 200);
              assert.equal(res.text, "no book exists");
              done();
            });

          test("Test GET /api/books/[id] with valid id in db", function (done) {
            chai
              .request(server)
              .get("/api/books/" + bookID)
              .end(function (err, res) {
                assert.equal(res.status, 200);
                assert.equal(res.body.title, "test-title");
                done();
              });
          });
        });

        suite(
          "POST /api/books/[id] => add comment/expect book object with id",
          function () {
            test("Test POST /api/books/[id] with comment", function (done) {
              chai
                .request(server)
                .post("/api/books/" + bookID)
                .send({ comment: "test-comment" })
                .end(function (err, res) {
                  assert.equal(res.status, 200);
                  assert.equal(res.body.comments[0], "test-comment");
                  done();
                });
            });

            test("Test POST /api/books/[id] without comment field", function (done) {
              chai
                .request(server)
                .post("/api/books/" + bookID)
                .send({})
                .end(function (err, res) {
                  assert.equal(res.status, 200);
                  assert.equal(res.text, "missing required field comment");
                  done();
                });
            });

            test("Test POST /api/books/[id] with comment, id not in db", function (done) {
              chai
                .request(server)
                .post("/api/books/" + "invalidID")
                .send({ comment: "test-comment" })
                .end(function (err, res) {
                  assert.equal(res.status, 200);
                  assert.equal(res.text, "no book exists");
                  done();
                });
            });
          }
        );

        suite("DELETE /api/books/[id] => delete book object id", function () {
          test("Test DELETE /api/books/[id] with valid id in db", function (done) {
            chai
              .request(server)
              .delete("/api/books/" + bookID)
              .end(function (err, res) {
                assert.equal(res.status, 200);
                assert.equal(res.text, "delete successful");
                done();
              });
          });

          test("Test DELETE /api/books/[id] with  id not in db", function (done) {
            chai
              .request(server)
              .delete("/api/books/" + "invalidID")
              .end(function (err, res) {
                assert.equal(res.status, 200);
                assert.equal(res.text, "no book exists");
                done();
              });
          });
        });
      });
    });
  });
});
```
