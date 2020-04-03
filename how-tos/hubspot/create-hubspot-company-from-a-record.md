# Create a Hubspot company



This example shows you how to create a Smart Action `"Create company in Hubspot"` that generates a company in Hubspot based on information from your database.

![](http://recordit.co/5j310Z59tb.gif)



## Requirements <a id="requirements"></a>

* An admin backend running on forest-express-sequelize
* [superagent](https://www.npmjs.com/package/superagent) npm package
* a Hubspot account

## How it works <a id="requirements"></a>

### Directory: `/models`

This directory contains the `companies.js` file where the collection is declared.

{% code title="/models/companies.js" %}
```javascript
module.exports = (sequelize, DataTypes) => {
  const { Sequelize } = sequelize;
  const Companies = sequelize.define('companies', {
    description: {
      type: DataTypes.STRING,
    },
    industry: {
      type: DataTypes.STRING,
    },
    headquarters: {
      type: DataTypes.STRING,
    },
    name: {
      type: DataTypes.STRING,
    },
    status: {
      type: DataTypes.ENUM,
      values: ['lead', 'customer', 'churn'],
    },
    createdAt: {
      type: DataTypes.DATE,
      defaultValue: Sequelize.literal('now()'),
    },
    crmId: {
      type: DataTypes.BIGINT,
    },
  }, {
    tableName: 'companies',
    underscored: true,
    timestamps: false,
    schema: process.env.DATABASE_SCHEMA,
  });

  Companies.associate = (models) => {
  };

  return Companies;
};

```
{% endcode %}

### Directory: `/forest`

This directory contains the `companies.js` file where the smart action is declared. A smart field has also been added to add a link to the company's Hubspot profile if the company's `crmId` field is not `null`.

{% code title="/forest/hubspot-companies.js" %}
```javascript
const { collection } = require('forest-express-sequelize');
const { getHubspotCompany, hubspotCompaniesSerializer } = require('../utils.js');

collection('companies', {
  actions: [
    {
      name: 'Create company in Hubspot',
      type: 'single',
    },
  ],
  fields: [
    {
      // adding a field that will allow to be directed on click to the company's profile in hubspot
      field: 'crm link',
      type: 'String',
      get: (company) => {
        return company.crmId ? 'https://app.hubspot.com/contacts/6332498/company/' + company.dataValues.crmId : null;
      },
    },
  ],
  segments: [],
});

```
{% endcode %}

### Directory: `/routes`

This directory contains the `companies.js` file where the smart action logic is implemented. 

In this logic a Hubspot company instance is created through a /post create company call to the Hubspot API.

{% hint style="info" %}
The Hubspot API key is defined in the `.env` file and requested through the expression `process.env.HUBSPOT_API`.
{% endhint %}

{% code title="/forest/hubspot-companies.js" %}
```javascript
const express = require('express');
const { PermissionMiddlewareCreator } = require('forest-express-sequelize');
const { companies } = require('../models');
const { getRecord, createHubspotCompany, setCrmId } = require('../utils.js');

const router = express.Router();
const permissionMiddlewareCreator = new PermissionMiddlewareCreator('companies');

// function that returns a sequelize object
function getRecord(collection, recordId) {
  return collection.findOne({ where: { id: recordId } });
}

// function that update a company record crmId with the hubspot companyId
function setCrmId(record, hubspotId) {
  record.crmId = hubspotId;
  return record.save();
}

// function that creates a company in Hubspot through the hubspot API
function createHubspotCompany(company) {
  return superagent
    .post(`https://api.hubapi.com/companies/v2/companies?hapikey=${process.env.HUBSPOT_API}`)
    .send({properties: [
      {
        name: 'name',
        value: company.name,
      },
      {
        name: 'description',
        value: company.description,
      },
      {
        name: 'city',
        value: company.headquarters,
      },
      {
        name: 'industry',
        value: company.industry,
      },
    ]})
    .then((response) => JSON.parse(response.res.text));
}


router.post('/actions/create-company-in-Hubspot', async (req, res) => {
  const companyId = req.body.data.attributes.ids[0];
  const company = await getRecord(companies, companyId);

  if (company.crmId) {
    return res.status(400).send({ error: 'A lead from Hubspot is already assigned to this company' });
  }
  try {
    const hubspotCompany = await createHubspotCompany(company);
    await setCrmId(company, hubspotCompany.companyId);
  } catch (err) {
    console.log('error => ', err);
    res.status(400).send({ error: 'could not create lead' });
  }
  return res.send({ success: 'Lead has been created in Hubspot!' });
});
```
{% endcode %}

