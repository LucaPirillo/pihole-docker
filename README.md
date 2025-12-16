# pihole-docker
Instalacion y configuracion de Pi-Hole en Docker - Ubuntu 24.0 LTS
# Pi-hole en Docker (Linux) — homelab setup

Repo con mi setup simple para correr **Pi-hole** en un contenedor Docker y usarlo como DNS en mi PC / teléfono (y luego en el router). La idea es tener bloqueo de ads/trackers a nivel DNS y un panel web para ver consultas.

## Qué incluye
- `docker-compose.yml` listo para levantar Pi-hole.
- Persistencia con volúmenes para no perder configuración al recrear el contenedor.

## Requisitos
- Linux + Docker + Docker Compose.
- Acceso a tu red local (esto está pensado para uso hogareño).

## Importante (IP correcta)
Para que otros dispositivos usen Pi-hole, **se usa la IP de tu PC (host)**, no una IP interna del contenedor. Como el compose publica puertos al host, tu DNS va a ser `IP_DE_TU_PC` (ej: `192.168.1.50`).

## Levantar
Desde la carpeta del proyecto:

```bash
docker compose up -d
docker ps

Panel web:

    http://localhost/admin (misma PC)

    http://IP_DE_TU_PC/admin (desde otro dispositivo)

Logs:

docker logs -f pihole

Ejemplo de docker-compose.yml

services:
  pihole:
    container_name: pihole
    image: pihole/pihole:latest
    ports:
      - "53:53/tcp"
      - "53:53/udp"
      - "80:80/tcp"
      - "443:443/tcp"
    environment:
      FTLCONF_webserver_api_password: "CambiaEstaClave"
      TZ: "America/Argentina/Buenos_Aires"
      FTLCONF_dns_listeningMode: "ALL"
    volumes:
      - "./etc-pihole:/etc/pihole"
      - "./etc-dnsmasq.d:/etc/dnsmasq.d"
    restart: unless-stopped

Contraseña del panel (sin ponerla en el compose)

Setear/cambiar password desde el contenedor:

docker exec -it pihole pihole setpassword

Problema típico: el puerto 53 está ocupado

Si al levantar te tira failed to bind ... 0.0.0.0:53 ... address already in use, casi siempre es porque Linux ya tiene un servicio DNS escuchando (muy común: systemd-resolved).

Checklist rápido:

sudo lsof -i :53

Solución que me funcionó en Ubuntu:

    Editar /etc/systemd/resolved.conf y poner:

[Resolve]
DNSStubListener=no

    Reiniciar el servicio:

sudo systemctl restart systemd-resolved.service

    Re-levantar Pi-hole:

docker compose down
docker compose up -d

Usar Pi-hole como DNS en dispositivos

    Linux: poné como DNS la IP de tu PC donde corre Pi-hole.

    Android: ojo con “DNS privado” (DoT/DoH). Si está activado, puede saltearse Pi-hole. Para probar, dejalo en Automático/Desactivado y configurá DNS en la red Wi-Fi.
