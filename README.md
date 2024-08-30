# **MICROPROYECTO_1_COMP**

## Descripción
Este microproyecto consiste en la instalación y configuración de Consul en un entorno con tres servidores: HAPROXY, appServer1 y appServer2. Además, se desplegará un servicio de Node.js en los servidores web para ilustrar cómo interactúan los componentes mediante balanceo de carga.

---

## **Requisitos Previos**
- Tres servidores configurados: HAPROXY, appServer1 y appServer2.
- Acceso de administrador a los servidores.
- Git instalado en appServer1 y appServer2.
- Node.js instalado en appServer1 y appServer2.

---

## **1. Instalación de Consul en los Servidores**
Consul debe instalarse en los tres servidores para permitir la comunicación y el descubrimiento de servicios.

### Para arquitectura **AMD**:
```bash
curl -O https://releases.hashicorp.com/consul/1.11.3/consul_1.11.3_linux_amd64.zip
```

### Para arquitectura **ARM**:
```bash
wget https://releases.hashicorp.com/consul/1.14.1/consul_1.14.1_linux_arm64.zip
```

---

## **2. Configuración de Consul en HAPROXY**
Configura Consul como servidor en HAPROXY para gestionar la comunicación y supervisión del clúster.

```bash
cat <<EOF | sudo tee /etc/consul.d/consul.hcl
data_dir = "/opt/consul"
server = true
bootstrap_expect = 1
bind_addr = "192.168.50.15"
client_addr = "0.0.0.0"
EOF
```

---

## **3. Configuración de Consul como Agente en appServer1 y appServer2**
Los servidores appServer1 y appServer2 actuarán como agentes y se unirán al servidor Consul en HAPROXY.

```bash
cat <<EOF | sudo tee /etc/consul.d/consul.hcl
data_dir = "/opt/consul"
retry_join = ["192.168.50.15"]
EOF
```

---

## **4. Despliegue del Servicio Node.js en appServer1 y appServer2**
Clona el repositorio del servicio en ambos servidores y realiza las configuraciones necesarias.

```bash
git clone https://github.com/omondragon/consulService
cd consulService/app
```

### Modifica el archivo `index.js`:
- **Para appServer1**:
  Cambia:
  ```js
  const HOST='192.168.100.3';
  ```
  por:
  ```js
  const HOST='192.168.50.13';
  ```

- **Para appServer2**:
  Cambia:
  ```js
  const HOST='192.168.100.3';
  ```
  por:
  ```js
  const HOST='192.168.50.14';
  ```

---

## **5. Iniciar Consul como Servicio**

### En **HAPROXY**:
```bash
nohup consul agent -server -bind=192.168.50.15 -data-dir=/opt/consul -config-dir=/etc/consul.d/ -ui &
```

### En **appServer1**:
```bash
nohup consul agent -node=appServer1 -bind=192.168.50.13 -data-dir=/opt/consul -config-dir=/etc/consul.d/ -join=192.168.50.15 -ui &
```

### En **appServer2**:
```bash
nohup consul agent -node=appServer2 -bind=192.168.50.14 -data-dir=/opt/consul -config-dir=/etc/consul.d/ -join=192.168.50.15 -ui &
```

---

## **6. Arrancar Servidores Node en appServer1 y appServer2**
Dirígete a la ubicación del archivo `index.js` en el repositorio clonado y ejecuta las instancias de Node.js.

```bash
cd consulService/app
nohup node index.js 3000 &
nohup node index.js 3001 &
nohup node index.js 3002 &
```

---

## **7. Apagar Servicios de Node.js**
Para detener las instancias de Node.js en ejecución:

1. Encuentra los procesos activos:
   ```bash
   pes aux | grep nod
   ```

2. Termina los procesos:
   ```bash
   sudo kill -9 <PID_NUMBER>
   ```

---

## **8. Apagar Servicios de Apache2**
Para detener los servicios de Apache de appServer1 y appServer2

1. Ejecutar para Apagar:
   ```bash
   sudo systemctl stop apache2
   ```

2. Ejecutar para Reinciar:
   ```bash
   sudo systemctl start apache2
   ```

---

## **URLs Importantes**
- **Consul UI**: [http://192.168.50.15:8500/ui](http://192.168.50.15:8500/ui)
- **HAProxy Stats**: [http://192.168.50.15/haproxy?stats](http://192.168.50.15/haproxy?stats)
- **Peticiones Balanceadas**: [http://192.168.50.15](http://192.168.50.15)