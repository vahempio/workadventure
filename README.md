# Work Adventure

Work Adventure is a web-based collaborative workspace for small to medium teams (2-100 people) presented in the form of a 16-bit video game.

In Work Adventure, you can move around your office and talk to your colleagues (using a video-chat feature that is triggered when you move next to a colleague).

All initial work is here: [https://workadventu.re/](https://workadventu.re/).

# How to start my fork
## Minimum requirement
With my tests, here is the minimal requirement for the host server:
- RAM: 2GB
- HDD: 6GB (including Ubuntu)

If you use AWS EC2, I recommend a **t3a.medium** instance as a starting point.
Capabilities: 2 vCPU - 4 GiB RAM - EBS Storage - 5 Gbit/s network capable speed. The cost is 0,0425 USD/h in eu-west-3 (Paris) for on-demand instances.

This is enough for 2-50 people at the same time.

### Create your server instance
I recommend the following settings:
- HDD = 10GB
- OS = Ubuntu 20.04 (or newer LTS)
- Assign an elastic IP, for static route

### Configure your firewall
Ports that should be opened for income traffic:
- 22/tcp (for SSH, or other if you want)
- 80/tcp (for HTTP->HTTPS redirection + Let's Encrypt challenge/response)
- 443/tcp (for HTTPS)
- 3478/udp (for TURN server)
- 5349/udp (for TURN server, cyphered link)

### Prepare your server
The following softwares must be installed on the server:
- git
- docker
- docker-compose
- certbot

### Prepare your DNS
For example, if we use the `foo.bar.com` domain, you should have these 2 reccords in your DNS zone, at least:
- `A` reccord type from your elasticIP to your domain `foo.bar.com`
- `CNAME` reccord type form the wildcard `*.foo.bar.com` to `foo.bar.com`

Of course, you are free to have another setup here, according to the docker-compose configuration, this is only the minimal example.

### Deploy & configure the server
On your server:
```
git clone https://github.com/vahempio/workadventure.git
```
You must edit the configuration to fit with your configuration. The minimum changes are the following:
- Put a valid email in `./traefik_files/config/traefik_static.yml` for a valid usage of Let's Encrypt
- Set your elasticIP value in `EXTERNAL_IP` field into `./docker.env` file for a correct TURN server behavior
- Set the `ROOT_DOMAIN` field into `./docker.env` file (example here: `foo.bar.com`)

Other fields should be also changed when you are confortable with this software, but it is not mandatory for testing.

### First test 
Run the reverse proxy Traefik with:
```
docker-compose --env docker.env up traefik
```
and connect a browser to https://traefik.foo.bar.com

If the traefik docker network does not exist, follow the solution which is present in your output console for creating this the first time.

If everything is correct, you should have a reverse proxy dashboard, with a valid certificate from Let's Encrypt, protected by a BasicAuth method. Log in with `admin:WAtraefik`. Remember to change the default password in `./traefik_files/.htpasswd` with the following command:
```
htpasswd -B -C 10 ./traefik_files/.htpasswd admin
```

If it is not the case, check your docker-compose console, the error decription will indicate you the issue.

### TURN server certificate
For using the secured connection with the embedded TURN server, a certificate must be provided. Today, it is not managed dynamically, here is the manual operation:
- Ask Let's Encrypt for a manual certificate request with `sudo certbot certonly --standalone -d coturn.YOUR_ROOT_DOMAIN`
- Move the obtained certificate full chain & privatekey in these respective destinations:
  - `./turn_cert/fullchain.pem`
  - `./turn_cert/privkey.pem`

Be carreful: this step should be automated (or extracted from traefik certificate database). Without this action, your certificate will expire in 3 months. If your email address is correctly fed, you will be notified by Let's Encrypt.

### Full game
Run the full game with:
```
docker-compose --env docker.env up -d
```
Wait 30 seconds, and connect a browser to https://play.foo.bar.com. In case of problems, check the logs with `docker-compose logs -f`.

Enjoy!

