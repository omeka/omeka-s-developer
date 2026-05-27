# Inverse Properties

## Enabling inversing when using the API

API operations must opt in to allowing Inverse Properties to run. This is done
automatically for normal user interactions using the interface, but users of
the API must pass the following with their API request payload if inverse
creation is desired:

```json
"inverse_properties_set_inverses": true
```
