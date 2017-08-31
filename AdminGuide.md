# Admin Guide

## create jwt token with python

```python
>>> import jwt
>>> encoded = jwt.encode({'iss': 'kong-key'}, 'kong-secret', algorithm='HS256')

'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzb21lIjoicGF5bG9hZCJ9.4twFt5NiznN84AWoo1d7KO1T_yoc0Z6XOpOVswacPZg'

>>> jwt.decode(encoded, 'secret', algorithms=['HS256'])
{'iss': 'payload'}
```