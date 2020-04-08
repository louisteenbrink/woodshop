# Import data from a CSV file

This example shows you how to create a Smart Action `"Import data"` to import data from a CSV file.

Forest Admin atively supports data creation but it’s sometimes more efficient to simply import it.

![](../.gitbook/assets/import-data.png)

## Requirements

* An admin backend running on forest-express-sequelize/forest-express-mongoose
* [bluebird](https://www.npmjs.com/package/bluebird) npm package
* [parse-data-uri](https://www.npmjs.com/package/parse-data-uri) npm package
* [csv](https://www.npmjs.com/package/csv) npm package

## How it works

### Directory: /models

This directory contains the `products.js` file where the model is declared.

{% tabs %}
{% tab title="Sequelize" %}
{% code title="/models/products.js" %}
```javascript

module.exports = (sequelize, DataTypes) => {
  const { Sequelize } = sequelize;
  const Products = sequelize.define('products', {
    price: {
      type: DataTypes.INTEGER,
    },
    label: {
      type: DataTypes.STRING,
    },
    picture: {
      type: DataTypes.STRING,
    },
    ...
  }, {
    tableName: 'products',
    underscored: true,
    schema: process.env.DATABASE_SCHEMA,
  });
​
  Produtcs.associate = (models) => {
  };
​
  return Products;
};
```
{% endcode %}
{% endtab %}

{% tab title="Mongoose" %}
{% code title="/models/products.js" %}
```javascript
const mongoose = require('mongoose');

const schema = mongoose.Schema({
  'price': Integer,
  'label': String,
  'picture': String,
  ...
}, {
  timestamps: false,
});

module.exports = mongoose.model('products', schema, 'products');
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **Directory: /forest**

This directory contains the `products.js` file where the Smart Action `Import data`is declared.

{% tabs %}
{% tab title="Sequelize" %}
{% code title="/forest/products.js" %}
```javascript
const { collection } = require('forest-express-sequelize');
const models = require('../models');

collection('products', {
  actions: [{
    name: 'Import data',
    endpoint: '/forest/products/actions/import-data',
    type: 'global',
    fields: [{
      field: 'CSV file',
      description: 'A semicolon separated values file stores tabular data (numbers and text) in plain text',
      type: 'File',
      isRequired: true
    }, {
      field: 'Type',
      description: 'Specify the product type to import',
      type: 'Enum',
      enums: ['phone', 'dress', 'toy'],
      isRequired: true
    }]
  }],

  // ...
});
```
{% endcode %}
{% endtab %}

{% tab title="Mongoose" %}
{% code title="/forest/products.js" %}
```javascript
const { collection } = require('forest-express-mongoose');
const models = require('../models');

Liana.collection('products', {
  actions: [{
    name: 'Import data',
    endpoint: '/forest/products/actions/import-data',
    type: 'global',
    fields: [{
      field: 'CSV file',
      description: 'A semicolon separated values file stores tabular data (numbers and text) in plain text',
      type: 'File',
      isRequired: true
    }, {
      field: 'Type',
      description: 'Specify the product type to import',
      type: 'Enum',
      enums: ['phone', 'dress', 'toy'],
      isRequired: true
    }]
  }],

  // ...
});
```
{% endcode %}
{% endtab %}
{% endtabs %}

### **Directory: /routes**

This directory contains the `users.js` file where the implementation of the route is handled. The `POST /forest/actions/import-data` API call is triggered when you click on the Smart Action in the Forest UI. 

The CSV file passed into the body of the API call is serialized using a base64 encoding [Data URI scheme](https://en.wikipedia.org/wiki/Data_URI_scheme).

To deserialize the base64 encoded CSV file, we use the NPM package [parse-data-uri](https://www.npmjs.com/package/parse-data-uri). We also use the [csv parser](https://github.com/adaltas/node-csv) NPM package to iterate over each line of the CSV file.

You can find a sample CSV file we use here to feed our products table on the [Live demo Github repository](https://raw.githubusercontent.com/ForestAdmin/forest-live-demo-lumber/master/seed/walmart-toys.csv).

You may find below the coding examples you need to make this Smart action work:

{% tabs %}
{% tab title="Sequelize" %}
{% code title="/routes/products.js" %}
```javascript
const P = require('bluebird');
const express = require('express');
const router = express.Router();
const faker = require('faker');
const parseDataUri = require('parse-data-uri');
const csv = require('csv');
const models = require('../models');

...

router.post('/products/actions/import-data',
  (req, res) => {
    let parsed = parseDataUri(req.body.data.attributes.values['CSV file']);
    let productType = req.body.data.attributes.values['Type'];

    csv.parse(parsed.data, { delimiter: ';' }, function (err, rows) {
      if (err) {
        res.status(400).send({
          error: `Cannot import data: ${err.message}` });
      } else {
        return P
          .each(rows, (row) => {
            // Random price for the example purpose. In a real situation, the price 
            // should certainly be available in the CSV file.
            let price = faker.commerce.price(5, 1000) * 100;

            return models.products.create({
              label: row[0],
              price: price,
              picture: row[1]
            });
          })
          .then(() => {
            res.send({ success: 'Data successfuly imported!' });
          });
      }
    });
  });
  
...

module.exports = router;
```
{% endcode %}
{% endtab %}

{% tab title="Mongoose" %}
{% code title="/routes/products.js" %}
```javascript
const P = require('bluebird');
const express = require('express');
const router = express.Router();
const faker = require('faker');
const parseDataUri = require('parse-data-uri');
const csv = require('csv');
const models = require('../models');

...

router.post('/products/actions/import-data',
  (req, res) => {
    let parsed = parseDataUri(req.body.data.attributes.values['CSV file']);
    let productType = req.body.data.attributes.values['Type'];

    csv.parse(parsed.data, { delimiter: ';' }, function (err, rows) {
      if (err) {
        res.status(400).send({
          error: `Cannot import data: ${err.message}` });
      } else {
        return P
          .each(rows, (row) => {
            // Random price for the example purpose. In a real situation, the price 
            // should certainly be available in the CSV file.
            let price = faker.commerce.price(5, 1000) * 100;

            return models.products.create({
              label: row[0],
              price: price,
              picture: row[1]
            });
          })
          .then(() => {
            res.send({ success: 'Data successfuly imported!' });
          });
      }
    });
  });
  
...

module.exports = router;
```
{% endcode %}
{% endtab %}
{% endtabs %}

