# Minecraft Server (Fabric 26.3)

Servidor de Minecraft Java Edition corriendo en la Raspberry Pi 4B con Docker, usando la imagen [`itzg/minecraft-server`](https://docker-minecraft-server.readthedocs.io). La imagen descarga Fabric y los mods automáticamente.

## Mods del servidor

| Mod | Para qué |
|---|---|
| Fabric API | Librería base que necesitan casi todos los mods de Fabric |
| Lithium | Optimiza la lógica del juego (ticks, IA de mobs, físicas) |
| FerriteCore | Reduce el uso de RAM |
| Chunky | Pregenera el mundo para no generar chunks mientras se juega |

Los mods del cliente (Sodium, Iris, Sodium Extra, ImmediatelyFast, Entity Culling, Mod Menu) y los shaders **no** van aquí: solo afectan el renderizado. Como ninguno de los mods del servidor cambia el gameplay, los jugadores pueden entrar con Minecraft **26.3 vanilla**.

## Instalación

```fish
# En la Pi
mkdir -p /mnt/datos/apps/minecraft
cd /mnt/datos/apps/minecraft
# copiar aquí compose.yaml, README.md y .gitignore

docker compose up -d
docker compose logs -f   # esperar a "Done (...)! For help, type "help""
```

El primer arranque tarda varios minutos: descarga el servidor, Fabric y los mods, y genera el mundo.

## Pregenerar el mundo (una sola vez)

Generar chunks es lo más pesado para la Pi, sobre todo con el mundo en el HDD por USB. Hacerlo antes de jugar evita lag:

```fish
docker exec -i minecraft rcon-cli
> chunky radius 1000
> chunky start
> chunky progress     # para ver el avance
```

Un radio de 1000 bloques puede tardar horas en la Pi. Se puede dejar corriendo; si se reinicia el servidor, `chunky continue` lo retoma.

## Uso diario

```fish
docker exec -i minecraft rcon-cli        # consola del servidor (Ctrl+D para salir)
docker compose logs -f                   # ver logs
docker compose restart                   # reiniciar
docker compose down                      # apagar (guarda el mundo antes)
```

Comandos útiles en la consola: `op <jugador>`, `whitelist add <jugador>`, `whitelist on`, `save-all`, `list`.

## Acceso para los jugadores

Por **Tailscale**, sin abrir puertos en el router:

1. Compartir la Pi con cada jugador desde la consola de administración de Tailscale.
2. Cada jugador instala Tailscale y acepta la invitación.
3. En Minecraft: **Multijugador → Agregar servidor** → `<IP-tailscale-de-la-pi>:25565` (o el nombre MagicDNS de la Pi).

### Modo offline

El servidor corre con `ONLINE_MODE=FALSE` para aceptar cuentas sin premium. Eso significa que **no verifica identidades**: cualquiera que llegue al puerto puede entrar con cualquier nombre de usuario, incluido el de un operador, y la whitelist solo compara nombres.

La seguridad depende de que el puerto **nunca** se exponga a internet (nada de port forwarding en el router ni Tailscale Funnel para este servicio). Solo deben poder llegar los dispositivos de la tailnet y de la red de la casa.

## Respaldos

El mundo vive en `data/world/`. Para un respaldo consistente:

```fish
docker exec minecraft rcon-cli save-off
docker exec minecraft rcon-cli save-all flush
tar -czf ~/backups/minecraft-(date +%F).tar.gz -C data world
docker exec minecraft rcon-cli save-on
```

## Actualizar

Para una versión nueva de Minecraft, cambiar `VERSION` en `compose.yaml` y verificar antes en Modrinth que los cuatro mods ya la soporten. Luego:

```fish
docker compose pull
docker compose up -d
```

Hacer un respaldo antes: un mundo abierto en una versión nueva no se puede volver a abrir en la anterior.