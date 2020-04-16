# Import data from a JSON file

This example shows you how to implement a smart action to upload a JSON file to a string field using the file picker widget available in the Forest Admin UI.  
  
Here an `activities` model has a `details` field of the type `string`. This field should contain information from a JSON file. 

![](http://g.recordit.co/IixzA100Gk.gif)



## Requirements

* An admin backend running on forest-express-sequelize

## How it works

### Directory: /models

This directory contains the `activities.js` file where the `activities` model is declared. 

{% tabs %}
{% tab title="activities.js" %}
```javascript
module.exports = (sequelize, DataTypes) => {
  const { Sequelize } = sequelize;

  const Activities = sequelize.define('activities', {
    details: {
      type: DataTypes.STRING,
    },
    startDate: {
      type: DataTypes.DATE,
    },
    finishDate: {
      type: DataTypes.DATE,
    },
  }, {
    tableName: 'activities',
    underscored: true,
    timestamps: false,
    schema: process.env.DATABASE_SCHEMA,
  });

  return Activities;
};

```
{% endtab %}
{% endtabs %}

### Directory: /forest

This directory contains the `activities.js` file where the smart action `upload-JSON` is declared. 

{% hint style="info" %}
You need to specify that the widget `file picker` is applicable to the input field used to upload the file.
{% endhint %}

{% code title="activities.js" %}
```javascript
const { collection } = require('forest-express-sequelize');

collection('activities', {
  actions: [
    {
      name: 'Upload JSON',
      type: 'single',
      fields: [{
        field: 'json',
        widget: 'file picker',
      }],
    },
  ],
  fields: [],
  segments: [],
});

```
{% endcode %}

### Directory: /routes

This directory contains the `activities.js` file where the logic of the smart action is implemented.

{% code title="activities.js" %}
```javascript
const express = require('express');
const { activities } = require('../models');

const router = express.Router();

router.post('/actions/upload-json', (req, res) => {
  const activityId = req.body.data.attributes.ids[0];
  // get the raw base64 file => if your field is a string and you want to insert the JSON as a base64 to use the file viewer, this is the value you want to return
  const rawFile = req.body.data.attributes.values.json;
  // trim the base64 string to delete the prefix
  const rawFileCleaned = rawFile.replace('data:application/json;base64', '');
  // get string from base64 string => if your field is a string and you want to insert the JSON in it, this is the value you want to return
  const stringFile = Buffer.from(rawFileCleaned, 'base64').toString('utf8');
  // check that you can properly parse json from the string obtained
  try {
    JSON.parse(stringFile);
  } catch (error) {
    return res.status(400).send({ error: 'not a correctly formatted json file' });
  }
  // find and update the current record's details field with the json file as a string
  activities
    .update(
      { details: stringFile },
      { where: { id: activityId } },
    )
    .then(() => {
      return res.send({ success: 'record updated!' });
    })
    .catch((e) => {
      console.log(e);
      return res.status(400).send({ error: 'could not update file' });
    });
});

module.exports = router;

```
{% endcode %}

