# Pre-fill a form with data from a relationship

This example shows you how to implement a smart action form where input fields are pre-filled with data coming from a hasOne relationship.  
  
Here a **Movie** `hasOne` **movieCharacteristic**.   
  
On the `movies` collection, we want to implement a smart action to edit the characteristics of a movie.

To do so, a smart action called `update-movie-characteristics`is defined with input fields corresponding to the fields of the `movieCharacteristic` we want to edit. Their value is pre-filled to ensure the user is aware of their current value before editing them.

![](http://g.recordit.co/b7DpYFpCQW.gif)



## Requirements

* An admin backend running on forest-express-sequelize

## How it works

### Directory: /models

This directory contains the `movies.js` and `movie-characteristics.js` files where the models are defined. 

{% tabs %}
{% tab title="movies.js" %}
```javascript
module.exports = (sequelize, DataTypes) => {
  const { Sequelize } = sequelize;

  const Movies = sequelize.define('movies', {
    title: {
      type: DataTypes.STRING,
    },
    releaseYear: {
      type: DataTypes.INTEGER,
    },
    rating: {
      type: DataTypes.INTEGER,
    },
    picture: {
      type: DataTypes.STRING,
    },
    academyAward: {
      type: DataTypes.BOOLEAN,
    },
  }, {
    tableName: 'movies',
    underscored: true,
    timestamps: false,
    schema: process.env.DATABASE_SCHEMA,
  });

  Movies.associate = (models) => {
    Movies.hasOne(models.movieCharacteristics, {
      foreignKey: {
        name: 'movieIdKey',
        field: 'movie_id',
      },
      as: 'movieCharacteristic',
    });
  };

  return Movies;
};

```
{% endtab %}

{% tab title="movie-characteristics.js" %}
```javascript
module.exports = (sequelize, DataTypes) => {
  const { Sequelize } = sequelize;
  
  const MovieCharacteristics = sequelize.define('movieCharacteristics', {
    language: {
      type: DataTypes.BOOLEAN,
    },
    graphicViolence: {
      type: DataTypes.BOOLEAN,
    },
    nudity: {
      type: DataTypes.BOOLEAN,
    },
    drugs: {
      type: DataTypes.BOOLEAN,
    },
    gore: {
      type: DataTypes.BOOLEAN,
    },
  }, {
    tableName: 'movie_characteristics',
    underscored: true,
    timestamps: false,
    schema: process.env.DATABASE_SCHEMA,
  });

  MovieCharacteristics.associate = (models) => {
    MovieCharacteristics.belongsTo(models.movies, {
      foreignKey: {
        name: 'movieIdKey',
        field: 'movie_id',
      },
      as: 'movie',
    });
  };

  return MovieCharacteristics;
};

```
{% endtab %}
{% endtabs %}

### Directory: /forest

This directory contains the `movies.js` file where the smart action `update-movie-characteristics` is declared. We use the [value method](https://docs.forestadmin.com/documentation/reference-guide/actions/create-and-manage-smart-actions#prefill-a-form-with-default-values) to pre-fill smart action forms. 

{% code title="/forest/programs.js" %}
```javascript
const { collection } = require('forest-express-sequelize');
const sequelize = require('sequelize');
const models = require('../models');

collection('movies', {
  actions: [{
    name: 'update movie characteristics',
    type: 'single',
    fields: [{
      field: 'language',
      type: 'Boolean',
      description: 'update the checkbox',
    }, {
      field: 'gore',
      type: 'Boolean',
      description: 'update the checkbox',
    }, {
      field: 'drugs',
      type: 'Boolean',
      description: 'update the checkbox',
    }, {
      field: 'graphicViolence',
      type: 'Boolean',
      description: 'update the checkbox',
    }, {
      field: 'nudity',
      type: 'Boolean',
      description: 'update the checkbox',
    }],
    values: async (context) => {
      console.log(context);
      // sequelize query to fetch the movie record 
      // do not forget to include the movie characteristics model
      const movie = await models.movies.findByPk(context.id, {
          include: [{ 
              model: models.movieCharacteristics,
              as: 'movieCharacteristic' 
          }] 
      });
      // Forest Admin will match all of the form fields that have the same name 
      // as your movie characteristics fields and pre-fill them
      return movie.movieCharacteristic;
    },
  }],
});

```
{% endcode %}

{% hint style="info" %}
Don't forget to make the values method asynchronous as you will need to await the resolve of the promise fetching the movie record.
{% endhint %}

