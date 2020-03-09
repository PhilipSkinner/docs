---
tags: [General]
---

# Idempotency-Key

Idempotency on requests allows for a failure in network communications and stops client errors when the same request has been received by a service multiple times when a retry pattern is in effect.

The client creates a unique idempotency key and transmits it to the service using the `X-idempotency-key` HTTP header. When the service receives this key, it keeps track of it and if it has seen this key before, on the same type of request, it does not action the modification that the request contains and instead just returns the same response it would have done, as if the request was successful.

The client is now free to send multiple requests with the same payload, to handle any networking issues that may be experienced.

Here is an example retry pattern with idempotency build in javascript (not tested):

```
const axios = require('axios').default;

const _deleteResource = (resourceId, idempotencyKey) => {
  try {  
    return axios.delete(`https://my.api.com/resource/${resourceId}`, {
      headers : {
        'X-idempotency-key' : idempotencyKey
      }
    });
  } catch(e) {
    return Promise.reject(e);
  }
};

const deleteResouce = (resourceId) => {  
  return new Promise((resolve, reject) => {
    const key = Math.random().toString(36).substring(7);
    let attempts = 0;
    const attempt = () => {
      attempts++;

      return _deleteResource(resourceId, key).then((res) => {
        return Promise.resolve(res);
      }).catch((err) => {
        console.log(err);
        if (attempts > 5) {
          return Promise.reject(err);
        }
        return attempt();
      });
    };

    return attempt().then(resolve).catch(reject);
  });
};
```