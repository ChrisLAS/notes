The basic idea is to create a tailscale service container, that runs along side the main app container. You then set the main app container to use the tailscale container for networking.

I followed this guide: [Using Tailscale with Docker](https://rnorth.org/tailscale-docker/).  Tailnets created after October 20, 2022 have MagicDNS on by default and will automatically register unique hostnames on the network.  Older tailnets must [enable MagicDNS in the DNS page of the admin console](https://tailscale.com/kb/1081/magicdns).

So first you add the tailscale service to the docker-compose.yml file:

```
services:

tailscale:
hostname: CHANGEME # This will become the tailscale device name
image: tailscale/tailscale

volumes:

- ./tailscale:/var/lib/tailscale # State data will be stored in this directory"/dev/net/tun:/dev/net/tun" # Required for tailscale to work

cap_add: # Required for tailscale to work

- net_admin
- sys_module

command: tailscaled
```

Then change the main app networking to use the tailscale service container:
```
  audiobookshelf:
    image: ghcr.io/advplyr/audiobookshelf:latest
    network_mode: service:tailscale
    volumes:
      - /cargo/Audiobooks:/audiobooks
      - /cargo/Podcasts:/podcasts
      - ./config:/config
      - ./metadata>:/metadata
```

Once the containers are up, you exec into the tailscale container and authenticate the tailscale session:

```
docker compose exec tailscale tailscale up
```
