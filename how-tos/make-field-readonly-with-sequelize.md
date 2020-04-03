# Make field readOnly with Sequelize

Sequelize does not allow by itself to define a field as readOnly. But you can do so by using on of Sequelize's addons named [sequelize-noupdate-attributes](https://www.npmjs.com/package/sequelize-noupdate-attributes).

![the LastName field is readOnly in Sequelize but not the email field](http://g.recordit.co/FkXyBYq06o.gif)

## Requirements

* An admin backend running on forest-express-sequelizet
* [sequelize-noupdate-attributes](https://www.npmjs.com/package/sequelize-noupdate-attributes) npm package

## How it works

### Directory: `/models`

This directory contains the files where your models are defined with Sequelize. To make fields readOnly you need to require the `sequelize-noupdate-attributes` package and add the `noUpdate` option to the relevant fields.

{% hint style="info" %}
The `noUpdate` option will throw an error if the user tries to update a field that already has a value. However the user will be able to update a field that has a `null` value.
{% endhint %}



{% code title="models/users.js" %}
```javascript
const sequelizeNoUpdateAttributes = require('sequelize-noupdate-attributes');

module.exports = (sequelize, DataTypes) => {
  const { Sequelize } = sequelize;
  sequelizeNoUpdateAttributes(sequelize);
  
  const Users = sequelize.define('users', {
    firstName: {
      type: DataTypes.STRING,
      noUpdate: true,
    },
    lastName: {
      type: DataTypes.STRING,
      noUpdate: true,
    },
    email: {
      type: DataTypes.STRING,
    },
  }, {
    tableName: 'users',
    underscored: true,
    timestamps: false,
    schema: process.env.DATABASE_SCHEMA,
  });

  return Users;
};

```
{% endcode %}

