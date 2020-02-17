
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
# Best practices for identifying resources
# Nouns: things, not actions
- GET api/authors
- GET api/authors/{authorId}
# Represent hierarchy when naming resources
- GET api/authors/{authorId}/courses
- GET api/authors/{authorId}/courses/{courseId}
# Filters, sorting orders, etc. aren't resources
- GET api/authors?orderby=name
# Sometimes, RPC-style calls don't easily map to pluralized resource names
- GET api/authors/{authorId}/totalamountofpages
