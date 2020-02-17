
# REST

## HATEOAS - Hypermedia as the Engine of Application State
- Drives how to consume the API  
- Allows for self-documenting API  

e.g.
```json
"links": [
  {
    "href": "http://localhost:3000/api/authors/1",
    "rel": "self",
    "method": "GET"
  },
  {
      "href": "http://localhost:3000/api/authors/1",
      "rel": "delete_author",
      "method": "DELETE"
  }
]
``` 
