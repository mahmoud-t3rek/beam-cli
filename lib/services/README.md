# Service (Behaviour Layer)
create a service ts/js that acts as a bridge layer between the CLI commands and the js-behaviours core execution engine.

## Overview
The service should provide all what is needed for the main controller to run the remote behaviours/commands.

### Key Concepts

- **js-behaviours**: Core engine that discovers behaviours and returns executable handlers. it is npm module

---
### js-behaviours example
tenants to be env var. you can use async/await according to the overall style. baseURL will be passed from the main controller along with the any other parameters needed. user to be env var.

```
const Behaviours = require('js-behaviours');
const logger = require('../utils/logger');
const tenants = process.env.TENANTS;

var behaviours;
var behaviours_queue = [];
var caches = {};

module.exports = {
  run: function (behaviour, parameters, user, callback) {
    var promise = new Promise((resolve, reject) => {
      if (!behaviours) {
        var baseURL = 'http://localhost:8385/api/v1';
        if (typeof behaviour !== 'string' || behaviour.length === 0) {
          throw new Error('Invalid behaviour name');
        } else if (typeof baseURL === 'string' && baseURL.length > 0) {
          behaviours = new Behaviours(baseURL, function (error) {
            if (error && error.code == 0) {
              logger.error(error);
            } else if (error && error.code == 401) {
              caches[user] = undefined;
            }
          });
        } else throw new Error('Invalid SSO URL');
      }
      behaviours.ready(function () {
        behaviours_queue.push(function () {
          if (user) {
            if (!caches[user]) caches[user] = {};
            behaviours.setCache(caches[user]);
          } else behaviours.setCache();
          var tenantID, tenant = Object.keys(tenants).find(function (key) {
            return tenants[key].port == process.env.PORT;
          });
          if (tenant) tenantID = tenants[tenant].id;
          behaviours[behaviour](parameters, function (response, error) {
            if (response && response.token) {
              var { user: tokenUser } = response;
              if (tokenUser && tokenUser.login && tokenUser.login != user) {
                caches[tokenUser.login] = caches[user];
              }
            }
            if (error && error.code != 0 && error.code != 401) {
              reject(error.message);
            } else {
              if (error) response = error;
              resolve(response);
            }
            behaviours_queue.shift();
            if (behaviours_queue.length > 0) behaviours_queue[0]();
          }, tenantID ? {

            __tenant__: {

              key: 'Behaviour-Tenant',
              type: 'header',
              value: tenantID
            }
          } : undefined);
        });
        if (behaviours_queue.length === 1) behaviours_queue[0]();
      });
    });
    if (callback) promise.then(function (response) {
      callback(response);
    }).catch(function (error) {
      callback(null, new Error(error));
    }); else return promise;
  },
  getCache: function (user) {
    return (caches[user] || {}).Behaviours;
  },
  setCache: function (user, cache) {
    caches[user] = { Behaviours: cache };
  },
  clearCache: function (user) {
    return caches[user] = undefined;
  }
}
```