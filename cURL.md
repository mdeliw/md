# cURL

```bash
$ curl https://www.domain.com
$ curl --user "USERNAME:PASSWORD" https://www.domain.com
$ curl --data -X POST "param1=value1&param2=value2" https://www.domain.com
$ curl -X "DELETE" http://localhost:3000/schedules/:id
$ curl -H "Content-Type: application/json" --data '{"name": "Sample Schedule Name"}' http://localhost:3000/schedules
$ curl http://wttr.in/LOCATION
$ curl -Is https://www.medium.com -L | grep HTTP/
```

#### # Reference

- https://itnext.io/curls-just-want-to-have-fun-9267432c4b55
- https://ec.haxx.se/