# W3ID Configuration for the CRIMA Ontology

This folder contains the configuration contributed to [W3ID](https://github.com/perma-id/w3id.org) to setup the redirect from `https://w3id.org/crima-ontology/` to `https://crima-ontology.github.io/` (e.g., [ccore.ttl](https://w3id.org/crima-ontology/ccore.ttl)).


## Contents

* `crima-ontology` directory: the namespace directory contributed to W3ID repo, with the exact `.htaccess` and `README.md` files of the [PR](https://github.com/perma-id/w3id.org/pull/6138) to W3ID
* `docker-compose.yml` file: the Docker Compose setup used to test the W3ID configuration locally before opening the PR


## Instructions

Start and keep running the Docker Compose application while testing the configuration for W3ID, stopping it at the end:
```bash
docker compose up       # start the application in foreground, press CTRL-C once done
docker compose down -v  # remove all resources (container and network)
```

While running Docker Compose, use `curl` to check if the W3ID redirection configuration in `crima-ontology/.htaccess` works correctly.
Note that you need replacign `https://w3id.org/` with `http://localhost:8080/`. For instance:

```bash
curl -v http://localhost:8080/crima-ontology/ccore
```

Just pay attention to the HTTP response status code and headers (displayed via `-v`), e.g.:
```
...
< HTTP/1.1 303 See Other
< Date: Fri, 05 Jun 2026 12:52:55 GMT
< Server: Apache/2.4.58 (Unix)
< Location: https://crima-ontology.github.io/latest/modules/ccore/ccore.ttl
< Content-Length: 270
< Content-Type: text/html; charset=iso-8859-1
...
```

The response above shows that `http://localhost:8080/crima-ontology/ccore` (that is, `http://w3id.org/crima-ontology/ccore` in production) will be redirected to `https://crima-ontology.github.io/latest/modules/ccore/ccore.ttl`, which is the correct location of the Turtle serialization (the default one) of the `ccore` module.


## Expected Redirections

The following `curl` commands provide a *test suite* covering different cases of redirections that should be supported by `crima-ontology/.htaccess`.

* Extension-based routing without version (which becomes `latest`), testing for extension normalization (e.g., `.owl` -> `.rdf`):
  ```bash
  curl -v http://localhost:8080/crima-ontology/ccore.n3      # -> http://localhost:8080/crima-ontology/latest/modules/ccore/ccore.ttl
  curl -v http://localhost:8080/crima-ontology/ccore.owl     # -> http://localhost:8080/crima-ontology/latest/modules/ccore/ccore.rdf
  curl -v http://localhost:8080/crima-ontology/ccore.jsonld  # -> http://localhost:8080/crima-ontology/latest/modules/ccore/ccore.json
  curl -v http://localhost:8080/crima-ontology/ccore.html    # -> http://localhost:8080/crima-ontology/latest/modules/ccore/ccore.html
  ```

* Extension-based routing with version, testing for extension normalization (e.g., `.owl` -> `.rdf`):
  ```bash
  curl -v http://localhost:8080/crima-ontology/1.0/ccore.n3      # -> http://localhost:8080/crima-ontology/1.0/modules/ccore/ccore.ttl
  curl -v http://localhost:8080/crima-ontology/1.0/ccore.owl     # -> http://localhost:8080/crima-ontology/1.0/modules/ccore/ccore.rdf
  curl -v http://localhost:8080/crima-ontology/1.0/ccore.jsonld  # -> http://localhost:8080/crima-ontology/1.0/modules/ccore/ccore.json
  curl -v http://localhost:8080/crima-ontology/1.0/ccore.html    # -> http://localhost:8080/crima-ontology/1.0/modules/ccore/ccore.html
  ```

* Content negotiation without version (which becomes `latest`), extension from `Accept` header with fallback to `.ttl`:
  ```bash
  curl -v -H "Accept: text/html" http://localhost:8080/crima-ontology/ccore              # -> http://localhost:8080/crima-ontology/latest/modules/ccore/ccore.html
  curl -v -H "Accept: application/rdf+xml" http://localhost:8080/crima-ontology/ccore    # -> http://localhost:8080/crima-ontology/latest/modules/ccore/ccore.rdf
  curl -v -H "Accept: application/ld+json" http://localhost:8080/crima-ontology/ccore    # -> http://localhost:8080/crima-ontology/latest/modules/ccore/ccore.json
  curl -v -H "Accept: application/n-triples" http://localhost:8080/crima-ontology/ccore  # -> http://localhost:8080/crima-ontology/latest/modules/ccore/ccore.nt
  curl -v -H "Accept: */*" http://localhost:8080/crima-ontology/ccore                    # -> http://localhost:8080/crima-ontology/latest/modules/ccore/ccore.ttl
  ```

* Content negotiation with version, extension from `Accept` header with fallback to `.ttl`:
  ```bash
  curl -v -H "Accept: text/html" http://localhost:8080/crima-ontology/1.0/ccore              # -> http://localhost:8080/crima-ontology/latest/modules/ccore/ccore.html
  curl -v -H "Accept: application/rdf+xml" http://localhost:8080/crima-ontology/1.0/ccore    # -> http://localhost:8080/crima-ontology/latest/modules/ccore/ccore.rdf
  curl -v -H "Accept: application/ld+json" http://localhost:8080/crima-ontology/1.0/ccore    # -> http://localhost:8080/crima-ontology/latest/modules/ccore/ccore.json
  curl -v -H "Accept: application/n-triples" http://localhost:8080/crima-ontology/1.0/ccore  # -> http://localhost:8080/crima-ontology/latest/modules/ccore/ccore.nt
  curl -v -H "Accept: */*" http://localhost:8080/crima-ontology/1.0/ccore                    # -> http://localhost:8080/crima-ontology/latest/modules/ccore/ccore.ttl
  ```