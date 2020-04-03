# Override the count route

This example shows you how to override the count default route.  
  
Forest Admin comes packaged with a set of existing routes, which execute Forest Admin's default logic. At installation, they are generated in `/routes`**.** 

## Requirements

* An admin backend running on forest-express-sequelize

## How it works

### Directory: `/routes`

This directory contains the `users.js` files where the routes are declared.

To override the count route, simply remove the `next()` statement and add your own logic:

{% code title="/routes/users.js" %}
```javascript
...

router.get('/customers/count', permissionMiddlewareCreator.list(), (request, response, next) => {
  response.send({ count: 100000000 });
});

...
```
{% endcode %}



