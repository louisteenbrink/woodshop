# Load smart fields using hook

This example shows you how to implement an afterFind hook with Sequelize to load several smart fields at once when a record is retrieved from the database. Learn more about [Sequelize hooks](https://sequelize.org/master/manual/hooks.html).

The main advantage of using an afterFind hook is that it allows you to avoid multiplying queries to your database if those queries are redundant when retrieving smart fields values.  
  
Here a **Program** `hasMany` **Activities**.   
  
We want to display two smart fields:  

* `"hasActivity"`, that will return whether the program has any related activities in the database.
* `"length"`, that will return the length of the program in weeks.

![](../.gitbook/assets/screenshot-2020-04-09-at-15.53.47.png)

## Requirements

* An admin backend running on forest-express-sequelize

## How it works

### Directory: /models

This directory contains the `programs.js` file where the model is declared. 

As Sequelize hooks are declared upon the model definition, the afterFind hook is implemented in this file.

{% tabs %}
{% tab title="programs.js" %}
```javascript
module.exports = (sequelize, DataTypes) => {
  const { Sequelize } = sequelize;
   const Programs = sequelize.define('programs', {
    startDate: {
      type: DataTypes.DATE,
    },
    finishDate: {
      type: DataTypes.DATE,
    },
    description: {
      type: DataTypes.STRING,
    },
  }, {
    tableName: 'programs',
    underscored: true,
    timestamps: false,
    schema: process.env.DATABASE_SCHEMA,
    hooks: {
      afterFind: async (records) => {
        // Check if several records have been fetched or a single one
        const isFindOne = !records.length;
        const limit = isFindOne ? 1 : records.length;
        if (isFindOne) { records = [records]; }
        const recordsIds = records.map((record) => record.id);
        const activities = await sequelize.models.activities.findAll({
          where: { programIdKey: recordsIds },
        });
        function calculateLength(program) {
          const diff = program.startDate - program.finishDate;
          const diffInWeeks = (diff / (1000 * 60 * 60 * 24 * 7));
          return Math.abs(Math.round(diffInWeeks));
        }
        for (let i = 0; i < limit; i++) {
          let record = records[i];
          record.length = calculateLength(record);
          let recordActivities = []
          for (let j = 0; j < activities.length; j++) {
            if (activities[j].programIdKey === record.id) {
              recordActivities.push(activities[j]);
            }
          }
          record.hasActivity = recordActivities.length > 0;
        }
        return isFindOne ? records[0] : records;
      },
    },
  });

  Programs.associate = (models) => {
    Programs.belongsTo(models.users, {
      foreignKey: {
        name: 'userIdKey',
        field: 'user_id',
      },
      as: 'user',
    });
    Programs.hasMany(models.activities, {
      foreignKey: {
        name: 'programIdKey',
        field: 'program_id',
      },
      as: 'activities',
    });
  };

  return Programs;
};

```
{% endtab %}
{% endtabs %}

### Directory: /forest

This directory contains the `programs.js` file where the smart fields `length` and `hasActivity` are declared. 

{% hint style="info" %}
As the logic to retrieve the field value is defined in the hook, you do not need to define the get method for each smart field.
{% endhint %}

{% code title="/forest/programs.js" %}
```javascript
const { collection } = require('forest-express-sequelize');

collection('programs', {
  actions: [],
  fields: [
    {
      field: 'length',
      type: 'Number',
    }, {
      field: 'hasActivity',
      type: 'Boolean',
    }
  ],
  segments: [],
});
```
{% endcode %}



