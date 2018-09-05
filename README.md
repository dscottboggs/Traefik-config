<div style="text-align: center;" markdown="1">
<h1>Traefik:</h1>
application layer reverse proxy with tight docker integration
</div>

### Configuration:
Traefik is a web application that sits between your web applications and the
actual internet. In this configuration, Traefik is used to route all requests
received on the public interface, based on two sources of rules:


- **traefik.toml** <br/>
  traefik.toml is the primary configuration file for traefik's routing rules. In
  our case, it contains the following configuration values:
  - listens on ports 80 and 443 (the standard HTTP and HTTPS ports,
    respectively).
  - denies any requests on port 80, instead redirecting all traffic to port 443.
  - automatically check any docker containers in the network that Traefik is
    connected to for labels that contain additional configuration rules. See
    below.
  - automatically configures LetsEncrypt TLS certificates, using an ACME plugin.
    This means that all traffic coming and going between the public internet and
    the Traefik application can and will be encrypted and secured. TLS
    (Transport-Layer Security, the technology that enables HTTPS), is absolutely
    essential to have if anyone needs to sign into your site, and it looks good
    to have even if you don't need it.
  - Log any incoming traffic that results in an error code - for example:
    - when someone (or some*bot*) enters a URL that's not valid
    - when another web application that traefik is routing for has an error
    - when authentication is denied by a a sign-in request

- **docker container labels** <br/>
  Docker allows for the creation of private networks that are internal to the
  physical computer you're on, or span across servers that are linked with
  Docker's *swarm* technology. In our case, all of our containers which have
  public facing interfaces are connected to the docker network named "web", and
  in each container's configuration we specify that. This allows the containers
  to talk to the reverse proxy. In addition, it allows containers to be labelled.
  Traefik can parse labels that you tag to each container, and create rules for
  routing based on these rules. See individual applications for the rules they
  use.

All docker configurations are handled by docker-compose.yml files, at the root
of the application folder; however, labels can be applied in a Dockerfile,
through the `docker` command, or with one of the docker APIs.

---

## File structure
the traefik folder can serve as an example for each application's configuration.

```
-- docker
 | -- traefik
 |  | -- mounts             # mounted files or folders
 |  |  | -- acme.json       # the place where traefik stores the encryption keys
 |  |  | -- traefik.toml
 |  | -- docker-compose.yml # the application configuration
 |  | -- README.md          # a documentation file like this
 | -- other_application
 |  | -- appname_build      # a folder with the files needed to build the image
 |  |  | -- Dockerfile
 |  |  | ...
 |  | -- mounts             # mounted files or folders
 |  |  | ...
 |  | -- docker-compose.yml # the application configuration
 |  | -- README.md          # a documentation file like this
```
