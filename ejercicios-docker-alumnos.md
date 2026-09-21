# 🐳 Ejercicios Prácticos de Docker

**Temas:** Imágenes y Contenedores · Volúmenes · Redes
**Nivel:** Básico → Intermedio
**Duración estimada:** 2–3 horas

> **Requisitos:** Docker instalado y corriendo (`docker --version`).
> Verifica con: `docker run hello-world`

---

## 📦 Bloque 1: Imágenes y Contenedores

### Ejercicio 1.1 — Tu primer contenedor
1. Descarga la imagen oficial de **nginx** usando `docker pull`.
2. Lista las imágenes locales y verifica que nginx aparece.
3. Ejecuta un contenedor llamado `mi-nginx` en segundo plano (`-d`), mapeando el puerto **8181** del host al **80** del contenedor.
4. Abre `http://localhost:8181` en el navegador .
4.EXTRA Intenta acceder por curl desde otro contendor.
5. Comprueba que el contenedor está corriendo con `docker ps`.



### Ejercicio 1.2 — Explorando el ciclo de vida
1. Detén el contenedor `mi-nginx` con `docker stop`.
2. Verifica que ya no aparece en `docker ps` pero sí en `docker ps -a`.
3. Vuelve a arrancarlo con `docker start` (sin crear uno nuevo).
4. Reinícialo con `docker restart` y luego elimínalo con `docker rm` (deberás detenerlo antes, salvo que uses `-f`).
5. Elimina la imagen de nginx con `docker rmi`.



### Ejercicio 1.3 — Inspeccionando contenedores
1. Crea un contenedor interactivo con la imagen `alpine` en modo terminal:
   `docker run -it --name mi-alpine alpine sh`
2. Dentro, ejecuta `cat /etc/os-release` y `hostname`, luego sal con `exit`.
3. ¿Sigue el contenedor corriendo? ¿Por qué? 
4. Vuelve a entrar con `docker start -ai mi-alpine` (-ai interactivo mas conectar consola) y comprueba que el sistema de archivos se conserva (crea un archivo: `echo hola > /tmp/prueba.txt`).
5. ¿hace falta algo mas? Sale y usa `docker exec mi-alpine cat /tmp/prueba.txt` mientras está corriendo… ¿funciona? ¿Qué necesitas hacer para que funcione?



### Ejercicio 1.4 — Logs y depuración
1. Corre: `docker run -d --name web -p 8181:80 nginx`
2. Genera tráfico: `curl localhost:8181`, o con un navegador, varias veces.
3. Muestra los logs del contenedor con `docker logs web`.
4. Sigue los logs en tiempo real con la opción `-f`.
5. Usa `docker inspect web` y extrae solo la IP del contenedor con `--format` y mirando directamente todo el json. (docker inspect --format='{{.NetworkSettings.IPAddress}}' web)




---

## 💾 Bloque 2: Volúmenes y Persistencia

### Ejercicio 2.1 — Los datos se pierden sin volúmenes
1. Corre un contenedor de `alpine`: `docker run -it --name sin-volumen alpine sh`
2. Dentro crea un archivo: `echo "datos importantes" > /datos.txt` y sal.
3. Elimina el contenedor (`docker rm -f sin-volumen`) y crea otro nuevo.
4. Busca `/datos.txt` → ya no existe. Los datos desaparecen con el contenedor.

### Ejercicio 2.2 — Tu primer volumen nombrado
1. Crea un volumen llamado `datos-persistentes`.
2. Corre un contenedor `alpine` montando ese volumen en `/data`.
3. Dentro, escribe `echo "hola volumen" > /data/archivo.txt` y sal.
4. Elimina el contenedor y crea uno nuevo con el mismo volumen.
5. Verifica que `cat /data/archivo.txt` sigue funcionando.



### Ejercicio 2.3 — Volúmenes anónimos y bind mounts
1. Ejecuta `docker run -d --name nginx-vol -v /usr/share/nginx/html nginx` y explica qué tipo de volumen se creó (anónimo).
2. Lista los volúmenes con `docker volume ls` y ubica el anónimo.
3. Ahora usa un **bind mount** para servir tu propio HTML:
   ```bash
   mkdir -p ~/sitio-web && echo "<h1>Mi sitio local</h1>" > ~/sitio-web/index.html
   docker run -d --name nginx-local -p 8082:80 -v ~/sitio-web:/usr/share/nginx/html:ro nginx
   ```
4. Abre `http://localhost:8082` y modifica el `index.html` en tu máquina: se refleja al instante.
5. ¿Qué significa el flag `:ro`? (read-only: el contenedor no puede escribir en ese montaje)

### Ejercicio 2.4 — Persistencia con MySQL (caso real)
1. Crea un volumen `mysql-data`.
2. Arranca MySQL 8 montando el volumen, con variables de entorno:
   ```bash
   docker run -d --name mysql-db \
     -v mysql-data:/var/lib/mysql \
     -e MYSQL_ROOT_PASSWORD=secret123 \
     -e MYSQL_DATABASE=prueba \
     -p 3306:3306 mysql:8
   ```
3. Espera a que inicie (`docker logs -f mysql-db`) y conéctate:
   ```bash
   docker exec -it mysql-db mysql -uroot -psecret123 -e "CREATE TABLE prueba.personas(id INT, nombre VARCHAR(50)); INSERT INTO prueba.personas VALUES (1, 'Ana');"
   ```
4. Elimina el contenedor, crea uno nuevo con el **mismo volumen** y verifica que los datos siguen ahí.
5. Inspecciona dónde está el volumen en el host con `docker volume inspect mysql-data`.



### Ejercicio 2.5 — Limpieza de volúmenes
1. Lista todos los volúmenes.
2. Elimina el volumen `datos-persistentes` (¿qué error da si hay un contenedor usándolo?).
3. Borra contenedores y volúmenes huérfanos de los ejercicios anteriores.

```bash
docker volume ls
docker rm -f c2                    # primero el contenedor que lo usa
docker volume rm datos-persistentes
docker volume prune                # elimina volúmenes sin uso
```
---

## 🌐 Bloque 3: Redes

### Ejercicio 3.1 — Redes por defecto
1. Lista las redes con `docker network ls` e identifica: `bridge`, `host`, `none`.
2. Corre dos contenedores `alpine` en la red `bridge` por defecto:
   ```bash
   docker run -dit --name host1 alpine sh
   docker run -dit --name host2 alpine sh
   ```
3. Obtén la IP de `host1` con `docker inspect`.
4. Desde `host2`, haz ping a esa IP: `docker exec host2 ping -c 3 <IP_HOST1>`.
5. Ahora intenta hacer ping **por nombre**: `docker exec host2 ping -c 3 host1` → ¿falla? ¿Por qué?

> ✅ **Conclusión:** la red bridge por defecto permite conectividad por IP, pero **no resuelve nombres de contenedores**.

### Ejercicio 3.2 — Red bridge personalizada con DNS interno
1. Crea una red propia: `docker network create mi-red`.
2. Inspecciónala (`docker network inspect mi-red`) y apunta su subred.
3. Arranca dos contenedores nuevos **en esa red** (`--network mi-red`).
4. Repite el ping por nombre → ahora **sí funciona**, porque las redes definidas por el usuario incluyen resolución DNS automática entre contenedores.



### Ejercicio 3.3 — Conectar y desconectar contenedores
1. Crea un contenedor `db` con alpine en `mi-red`.
2. Crea otro `cliente` **sin** red personalizada y comprueba que no puede hacer ping a `db`.
3. Conecta el cliente a la red: `docker network connect mi-red cliente`.
4. Repite el ping → ahora funciona.
5. Desconéctalo con `docker network disconnect mi-red cliente`.

### Ejercicio 3.4 — Caso real: Nginx + Nginx (comunicación por red)
1. User imagen nginx con el contenido cambiado
   ```bash
   docker run -d --name backend --network mi-red mi-app:1.0
   ```
2. Crea un `nginx.conf` que haga de proxy reverso hacia `backend:5000`:
   ```nginx
   server {
       listen 80;
       location / {
           proxy_pass http://backend:5000;
       }
   }
   ```
3. Corre nginx montando ese archivo y en la misma red:
   ```bash
   docker run -d --name proxy -p 8083:80 --network mi-red \
     -v $(pwd)/nginx.conf:/etc/nginx/conf.d/default.conf:ro nginx
   ```
4. Accede a `http://localhost:8083` → el proxy debería responder con el mensaje de Flask.
5. Prueba que el proxy deja de funcionar si haces `docker network disconnect mi-red proxy`.

### Ejercicio 3.6 — Limpieza final
```bash
docker rm -f $(docker ps -aq)        # elimina todos los contenedores
docker network prune                 # elimina redes sin uso
docker volume prune                  # elimina volúmenes sin uso
docker system df                     # muestra el espacio usado
```



