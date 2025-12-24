# 🏠 Docker Homelab Config

Configuración completa de Docker para home lab con Portainer, WireGuard VPN y Nginx Proxy Manager. Optimizado para WSL2 en disco F: y red local 192.168.18.0/24.

## 📦 Servicios Incluidos

### Portainer
- **Puerto**: 9000 (HTTP), 9443 (HTTPS)
- **Función**: Gestión visual de Docker con interfaz web
- **Acceso**: `http://localhost:9000` o `http://192.168.18.XXX:9000`

### WireGuard (wg-easy)
- **Puerto VPN**: 51820/UDP
- **Puerto Web UI**: 51821
- **Función**: VPN segura para acceso remoto a tu red
- **Acceso**: `http://localhost:51821`

### Nginx Proxy Manager (opcional)
- **Puertos**: 80 (HTTP), 443 (HTTPS), 81 (Admin UI)
- **Función**: Reverse proxy con certificados SSL automáticos
- **Acceso**: `http://localhost:81`

## 🚀 Instalación Rápida

### Prerrequisitos

1. **WSL2 con Docker instalado**
2. **Docker Compose** (incluido con Docker Desktop)
3. **Disco F:** configurado para WSL2

### Paso 1: Clonar el repositorio

```bash
# En WSL2 (disco F:)
cd /mnt/f
mkdir homelab && cd homelab
git clone https://github.com/Gasnokamae/docker-homelab-config.git
cd docker-homelab-config
```

### Paso 2: Configurar variables de entorno

Crea un archivo `.env` en el mismo directorio:

```bash
nano .env
```

Añade:

```env
WG_HOST=tu-dns.duckdns.org
WG_PASSWORD=TuPasswordSegura123!
```

### Paso 3: Levantar los servicios

```bash
# Levantar SOLO Portainer y WireGuard
docker-compose up -d portainer wireguard

# O levantar TODOS los servicios (incluye Nginx)
docker-compose up -d
```

### Paso 4: Verificar que están corriendo

```bash
docker-compose ps
```

## 🔧 Configuración Post-Instalación

### Portainer (Primera vez)

1. Abre `http://localhost:9000`
2. Crea un usuario admin con contraseña segura
3. Selecciona "Docker" como environment local
4. Ya puedes gestionar contenedores desde la UI

### WireGuard VPN

1. Abre `http://localhost:51821`
2. Login con la contraseña del archivo `.env`
3. Crea un nuevo cliente VPN (botón "+")
4. Escanea el código QR con la app WireGuard (móvil) o descarga el archivo .conf (PC)

### Configurar Router (para acceso externo)

**Solo si quieres acceso VPN desde fuera de casa:**

1. Abre la configuración de tu router
2. Ve a **Port Forwarding** / **Reenvío de puertos**
3. Añade una regla:
   - **Puerto externo**: 51820
   - **Puerto interno**: 51820
   - **Protocolo**: UDP
   - **IP interna**: IP de tu PC con WSL2 (ej: 192.168.18.101)

## 📡 Añadir Otros PCs a Portainer

### En el PC remoto (Windows con Docker Desktop)

1. Activa el Docker daemon en modo remoto:
   - Docker Desktop → Settings → General
   - Marca "Expose daemon on tcp://localhost:2375 without TLS"

2. En WSL del PC remoto, edita `/etc/docker/daemon.json`:

```json
{
  "hosts": ["tcp://0.0.0.0:2375", "unix:///var/run/docker.sock"]
}
```

3. Reinicia Docker:

```bash
sudo systemctl restart docker
```

### En Portainer (PC principal)

1. Ve a `http://localhost:9000`
2. Settings → Environments → Add environment
3. Selecciona "Docker"
4. Nombre: `PC-2` (o el que quieras)
5. URL: `192.168.18.102:2375` (IP del PC remoto)
6. Click en "Add environment"

¡Ahora puedes gestionar Docker en todos tus PCs desde un solo Portainer!

## 🛡️ Acceso Remoto Seguro

### Opción 1: Solo VPN (Recomendado)

1. Configura WireGuard como se indica arriba
2. Conéctate a la VPN desde fuera
3. Accede a Portainer como si estuvieras en casa: `http://192.168.18.101:9000`

### Opción 2: VPN + Nginx Proxy Manager

1. Levanta Nginx Proxy Manager: `docker-compose up -d nginx-proxy`
2. Configura dominios con SSL en Nginx (ej: `portainer.tu-dominio.com`)
3. Accede desde cualquier lugar con HTTPS

## 📂 Estructura de Carpetas

```
/mnt/f/homelab/docker-homelab-config/
├── docker-compose.yml      # Configuración de servicios
├── .env                    # Variables de entorno (CREAR ESTE)
└── README.md              # Esta guía
```

## 🔍 Comandos Útiles

```bash
# Ver logs de un servicio
docker-compose logs portainer
docker-compose logs wireguard

# Reiniciar un servicio
docker-compose restart portainer

# Parar todos los servicios
docker-compose down

# Actualizar imágenes
docker-compose pull
docker-compose up -d

# Ver IP de tu PC
ip addr show eth0 | grep inet
```

## 🆘 Troubleshooting

### Portainer no arranca

```bash
# Verifica que Docker esté corriendo
docker ps

# Revisa los logs
docker-compose logs portainer
```

### WireGuard no permite conexiones

1. Verifica que el puerto 51820/UDP esté abierto en el router
2. Comprueba que `WG_HOST` en `.env` apunte a tu IP pública o DNS
3. Revisa logs:

```bash
docker-compose logs wireguard
```

### No puedo acceder desde otros PCs en la red

- Verifica que el firewall de Windows permite conexiones en el puerto 9000
- Comprueba la IP con `ip addr show eth0 | grep inet`

## 📝 Notas

- **Zona horaria**: Configurada como `Europe/Madrid`, cámbiala en `docker-compose.yml` si es necesario
- **Red interna**: 172.20.0.0/16 (no debería colisionar con tu LAN)
- **Persistencia**: Todos los datos se guardan en volúmenes Docker persistentes

## 🔗 Enlaces Útiles

- [Documentación de Portainer](https://docs.portainer.io/)
- [Guía de WireGuard](https://github.com/wg-easy/wg-easy)
- [Nginx Proxy Manager](https://nginxproxymanager.com/)

## 📜 Licencia

MIT License - Úsalo libremente para tu homelab

---

**¿Problemas o sugerencias?** Abre un Issue en GitHub 🚀
