// Import required modules
const express = require('express');
const bodyParser = require('body-parser');
const mongodb = require('mongodb');
const { MongoClient } = mongodb;
const app = express();
const port = 3000;


const url = 'mongodb://127.0.0.1:27017';
const dbName = 'productDB';


app.use(bodyParser.urlencoded({ extended: true }));
app.use(express.static('public'));

MongoClient.connect(url, (err, client) => {
  if (err) {
    console.error('Error connecting to the database:', err);
    return;
  }

  console.log('Connected to MongoDB');

  const db = client.db(dbName);

 
  app.get('/', async (req, res) => {
    const products = await db.collection('products').find({}).toArray();
    res.render('index', { products });
  });

  app.get('/add', (req, res) => {
    res.render('add');
  });

  app.post('/add', async (req, res) => {
    const product = req.body;
    await db.collection('products').insertOne(product);
    res.redirect('/');
  });

  app.get('/edit/:id', async (req, res) => {
    const productId = req.params.id;
    const product = await db.collection('products').findOne({ _id: mongodb.ObjectId(productId) });
    res.render('edit', { product });
  });

  app.post('/edit/:id', async (req, res) => {
    const productId = req.params.id;
    const updatedProduct = req.body;
    await db.collection('products').updateOne(
      { _id: mongodb.ObjectId(productId) },
      { $set: updatedProduct }
    );
    res.redirect('/');
  });

  app.get('/delete/:id', async (req, res) => {
    const productId = req.params.id;
    await db.collection('products').deleteOne({ _id: mongodb.ObjectId(productId) });
    res.redirect('/');
  });

  app.listen(port, () => {
    console.log(`Server running on http://localhost:${port}`);
  });
});
