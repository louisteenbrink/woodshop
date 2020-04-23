# Upload files to AWS S3

This example shows you how to implement a smart action to upload image files to an AWS S3 bucket.  
  
Here we have a `companies` model that has two fields corresponding to files stored in an AWS S3 bucket:

* A certificate of incorporation
* A proof of identity

We implemented a smart action to upload the files for each company. 

![](http://g.recordit.co/4Fznmfl3Vh.gif)

## Requirements

* An admin backend running on forest-express-sequelize
* An AWS S3 bucket with access credentials
* The [aws-sdk](https://www.npmjs.com/package/aws-sdk) npm package
* The [bluebird](https://www.npmjs.com/package/bluebird) npm package
* The [parse-data-uri](https://www.npmjs.com/package/parse-data-uri) npm package

## How it works

### Directory: /models

This directory contains the `companies.js` file where the `companies` model is declared. 

{% tabs %}
{% tab title="companies.js" %}
```javascript
module.exports = (sequelize, DataTypes) => {
  const { Sequelize } = sequelize;
  const Companies = sequelize.define('companies', {
    name: {
      type: DataTypes.STRING,
    },
    industry: {
      type: DataTypes.STRING,
    },
    headquarter: {
      type: DataTypes.STRING,
    },
    status: {
      type: DataTypes.STRING,
    },
    description: {
      type: DataTypes.STRING,
    },
    createdAt: {
      type: DataTypes.DATE,
    },
    updatedAt: {
      type: DataTypes.DATE,
    },
    certificateOfIncorporationId: {
      type: DataTypes.UUID,
    },
    passportId: {
      type: DataTypes.UUID,
    },
  }, {
    tableName: 'companies',
    underscored: true,
    schema: process.env.DATABASE_SCHEMA,
  });

  return Companies;
};


```
{% endtab %}
{% endtabs %}

### Directory: /forest

This directory contains the `companies.js` file where the smart action `Upload Legal Docs` is declared. 

{% hint style="info" %}
You need to specify that the widget `file picker` is applicable to the input field used to upload the file.
{% endhint %}

{% code title="companies.js" %}
```javascript
const { collection } = require('forest-express-sequelize');

collection('companies', {
  actions: [
    {
      name: 'Upload Legal Docs',
      type: 'single',
      fields: [{
        field: 'Certificate of Incorporation',
        description: 'The legal document relating to the formation of a company or corporation.',
        type: 'File',
        isRequired: true,
      }, {
        field: 'Valid proof of ID',
        description: 'ID card or passport if the document has been issued in the EU, EFTA, or EEA / ID card or passport + resident permit or driving licence if the document has been issued outside the EU, EFTA, or EEA of the legal representative of your company',
        type: 'File',
        isRequired: true,
      }],
    },
  ],
  fields: [],
  segments: [],
});
```
{% endcode %}

### Directory: /services

This directory contains an `s3-helper.js` file where the methods to upload files to s3 is declared.

{% hint style="info" %}
You need to configure your AWS credentials inside your app to get access to your bucket. You can read more about it in the AWS documentation [here](https://docs.aws.amazon.com/sdk-for-javascript/v2/developer-guide/setting-credentials-node.html).
{% endhint %}

```javascript
const P = require('bluebird');
const parseDataUri = require('parse-data-uri');
const AWS = require('aws-sdk');

function S3Helper() {
  this.upload = (rawData, filename) => new P((resolve, reject) => {
    // Create the S3 client.
    const s3Bucket = new AWS.S3({ params: { Bucket: process.env.S3_BUCKET } });
    const parsed = parseDataUri(rawData);
    const base64Image = rawData.replace(/^data:(image|application)\/\w+;base64,/, '');

    const data = {
      Key: filename,
      Body: new Buffer(base64Image, 'base64'),
      ContentEncoding: 'base64',
      ContentDisposition: 'inline',
      ContentType: parsed.mimeType,
    };
      // Upload the image.
    s3Bucket.upload(data, (err, response) => {
      if (err) {
        return reject(err);
      }
      return resolve(response);
    });
  });
}

module.exports = S3Helper;
```

### Directory: /routes

This directory contains the `companies.js` file where the logic of the smart action is implemented.

{% code title="companies.js" %}
```javascript
const express = require('express');
const { PermissionMiddlewareCreator } = require('forest-express-sequelize');
const { companies } = require('../models');
const uuid = require('uuid/v4');
const S3Helper = require('../services/s3-helper');
const P = require('bluebird');


const router = express.Router();
const permissionMiddlewareCreator = new PermissionMiddlewareCreator('companies');

function uploadLegalDoc(companyId, doc, field) {
  const id = uuid();
  //upload the files using the helper
  return new S3Helper().upload(doc, `livedemo/legal/${id}`)
  //once the file is uploaded, update the relevant company field with the file id
    .then(() => companies.findByPk(companyId))
    .then((company) => {
      company[field] = id;
      return company.save();
    })
    .catch((e) => e);
}

router.post('/actions/upload-legal-docs', permissionMiddlewareCreator.smartAction(), (req, res) => {
  // Get the current company id
  const companyId = req.body.data.attributes.ids[0];

  // Get the values of the input fields entered by the admin user.
  const attrs = req.body.data.attributes.values;
  const certificateOfIncorporation = attrs['Certificate of Incorporation'];
  const passportId = attrs['Valid proof of ID'];

  P.all([
    uploadLegalDoc(companyId, certificateOfIncorporation, 'certificateOfIncorporationId'),
    uploadLegalDoc(companyId, passportId, 'passportId'),
  ])
    .then(() => {
      // Once the upload is finished, send a success message to the admin user in the UI.
      return res.send({ success: 'Legal documents are successfully uploaded.' });
    });
});

module.exports = router;
```
{% endcode %}

