# Geocode an address with Algolia

This example shows you how to use an autocomplete address smart field to update a PostreSQL geography point \(lat, long\).

{% embed url="https://youtu.be/SWldaV29s9U" %}

## Context

Here for the collection `events` I need to fill in two fields in my database to set an event location: `address` \(a string\) and `locationGeo` \(a postresql geography point\). 

Here is the model definition:

{% code title="models/events.js" %}
```javascript
module.exports = (sequelize, DataTypes) => {
  const { Sequelize } = sequelize;
  const Model = sequelize.define('events', {
    name: {
      type: DataTypes.STRING,
      primaryKey: true
    },
    locationGeo: {
      type: DataTypes.GEOMETRY('POINT', 4326),
    },
    address: {
      type: DataTypes.STRING,
    },
  }, {
    tableName: 'events',
    underscored: true,
    timestamps: false,
    schema: process.env.DATABASE_SCHEMA,
  });

  Model.removeAttribute('id');

  Model.associate = (models) => {
  };

  return Model;
};


```
{% endcode %}

Although a widget with autocomplete exists to fill an address string in the UI \([https://docs.forestadmin.com/documentation/reference-guide/fields/customize-your-fields/edit-widgets\#address](https://docs.forestadmin.com/documentation/reference-guide/fields/customize-your-fields/edit-widgets#address)\), the location coordinates can only be obtained manually by looking up the address in our search engine which is not optimal.

**The implementation proposed below allows to update both fields in one input using the address autocomplete.**

## Requirements

* An admin backend running on forest-express-sequelize
* An algolia account
* [algoliasearch](https://www.npmjs.com/package/algoliasearch) npm package

## How it works

### Directory: /forest

This directory contains the `events.js` file where the Smart Field `Location setter`is declared.

{% code title="/forest/events.js" %}
```javascript
const { collection } = require('forest-express-mongoose');

const algoliasearch = require('algoliasearch');
const places = algoliasearch.initPlaces(process.env.PLACES_APP_ID, process.env.PLACES_API_KEY);

collection('events', {
  fields: [{
    field: 'Location setter',
    type: 'String',
    get: (event) => {
      return event.address
    },
    set: (event, query) => {
      async function getLocationCoordinates(query){
          try {
            const location = await places.search({query: query, type:'address'});
            console.log('search location coordinates result', location.hits[0]._geoloc);
            return location.hits[0]._geoloc
          } catch (err) {
            console.log(err);
            console.log(err.debugData);
          }
      }

      async function setEvent(event, query) {
        const coordinates = await getLocationCoordinates(query)
        event.address = query
        console.log('new address', event.address)
        event.locationGeo = `{"type": "Point", "coordinates": [${coordinates.lat}, ${coordinates.lng}]}`
        console.log('new location', event.locationGeo)
        return event
      }

      return setEvent(event, query)
    }
  }],
});
```
{% endcode %}

First step is to create a smart field \(virtual field that does not exist in DB but is displayed and editable in Forest Admin\) in your Forest Admin backend app that will serve as the input field.

The smart field, that I named `location setter`, is a string and returns the value of the address field for display purposes.

Upon edit of the field, the input provided will be used to fetch the address coordinates through an API \(I used algolia places as our widget relies on their API - so I know for sure the address string passed will return the correct result from the API\) and update both the field address and the field location with the relevant values.

### Folder: /forest

This file contains the declaration of the Smart Field `Location setter`. As mentioned in [the documentation regarding smart fields](https://docs.forestadmin.com/documentation/reference-guide/fields/create-and-manage-smart-fields), the `get` method is used to get the data to be displayed and the `set` method is the logic applicable when an input is entered for this field in edit mode.

{% code title="forest/events.js" %}
```javascript
const Liana = require('forest-express-sequelize');
const algoliasearch = require('algoliasearch');
const places = algoliasearch.initPlaces(process.env.PLACES_APP_ID, process.env.PLACES_API_KEY);

Liana.collection('events', {
  fields: [{
    field: 'Location setter',
    type: 'String',
    get: (event) => {
      return event.address
    },
    set: (event, query) => {
      async function getLocationCoordinates(query){
          try {
            const location = await places.search({query: query, type:'address'});
            console.log('search location coordinates result', location.hits[0]._geoloc);
            return location.hits[0]._geoloc
          } catch (err) {
            console.log(err);
            console.log(err.debugData);
          }
      }

      async function setEvent(event, query) {
        const coordinates = await getLocationCoordinates(query)
        event.address = query
        console.log('new address', event.address)
        event.locationGeo = `{"type": "Point", "coordinates": [${coordinates.lat}, ${coordinates.lng}]}`
        console.log('new location', event.locationGeo)
        return event
      }

      return setEvent(event, query)
    }
  }],
});
```
{% endcode %}



###  Forest admin UI

Lastly I need to specify that the field `location setter` should use the edit widget `address` to enable the address autocomplete.

This directory contains the file `events.js` where the model's definition had been generated by Forest Admin.

