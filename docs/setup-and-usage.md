# Configuración y uso local

Esta guía concentra los pasos operativos para levantar el ecosistema local, inicializar datos de ejemplo y ejecutar una prueba integrada. El README principal mantiene la visión del caso; este documento conserva los detalles largos de instalación y uso.

## Ruta recomendada para estudiantes

Use esta ruta antes de responder la Parte A de PC03. El objetivo no es memorizar comandos, sino entender el flujo real del sistema y reunir evidencia mínima.

1. Leer el `README.md` para comprender el caso **Tutorías Universitarias**.
2. Preparar archivos `.env` desde sus plantillas.
3. Levantar servicios con Docker Compose.
4. Inicializar bases de datos PostgreSQL con los scripts de esta guía.
5. Abrir cliente, dashboard, RabbitMQ Management, Prometheus y Grafana.
6. Obtener un token, ejecutar una solicitud de tutoría confirmada y revisar trazabilidad.
7. Probar o inspeccionar un conflicto `409` y registrar qué evidencia lo demuestra.

Los comandos usan `docker compose`, que es la sintaxis moderna integrada a Docker. Si su instalación solo reconoce el binario anterior, use `docker-compose` como fallback manteniendo los mismos argumentos.

## Prerrequisitos

Antes de levantar el proyecto, instalar y verificar estas herramientas base. No avance a Docker Compose si alguna verificación falla: primero corrija la instalación, porque después los errores serán más difíciles de diagnosticar.

| Herramienta | Para qué se usa en este proyecto | Instalación recomendada | Verificación mínima |
| --- | --- | --- | --- |
| Git | Clonar el repositorio y revisar cambios locales. | Instalar desde <https://git-scm.com/downloads> o mediante el gestor del sistema operativo. | `git --version` |
| Docker Desktop | Ejecutar microservicios, PostgreSQL, RabbitMQ, Toxiproxy, Prometheus y Grafana. | Instalar desde <https://www.docker.com/products/docker-desktop/> y abrir la aplicación antes de ejecutar comandos. | `docker --version` y `docker compose version` |
| Node.js 18+ | Ejecutar servicios, clientes o pruebas fuera de Docker cuando sea necesario. | Instalar Node.js LTS desde <https://nodejs.org/>. Si ya usa `nvm`, instalar una versión LTS compatible. | `node --version` y `npm --version` |
| Cliente SQL | Crear tablas, insertar datos seed y revisar registros en PostgreSQL. | Usar DBeaver, pgAdmin o el cliente de línea de comandos `psql`. | Conectarse a `localhost:5432`, `localhost:5433` o `localhost:5434` después de levantar Docker Compose. |
| Cliente HTTP | Probar endpoints y repetir solicitudes con headers específicos. | Usar Postman, Insomnia, Thunder Client, REST Client de VS Code o `curl`. | Poder enviar una solicitud HTTP local a un puerto expuesto. |
| Navegador web moderno | Abrir cliente simulado, dashboard, RabbitMQ Management, Prometheus y Grafana. | Usar Chrome, Edge, Firefox o equivalente actualizado. | Abrir `http://localhost:8080` después del arranque. |
| Terminal | Ejecutar comandos desde la raíz del repositorio. | macOS/Linux: terminal del sistema. Windows: PowerShell, Windows Terminal o Git Bash. | Ubicarse en la carpeta raíz y ejecutar `pwd` o `cd`. |

### Pautas de instalación por herramienta

#### 1. Git

1. Instalar Git.
2. Cerrar y volver a abrir la terminal.
3. Verificar:

   ```bash
   git --version
   ```

4. Clonar o actualizar el repositorio según las indicaciones del curso.

#### 2. Docker Desktop y Docker Compose

1. Instalar Docker Desktop.
2. Abrir Docker Desktop y esperar a que indique estado listo.
3. Verificar desde una terminal:

   ```bash
   docker --version
   docker compose version
   ```

4. Si `docker compose` no existe pero `docker-compose` sí, use `docker-compose` como fallback en los comandos de esta guía.
5. En Windows, mantener Docker Desktop usando backend WSL 2 cuando esté disponible. Evite mezclar rutas de Windows y Linux dentro del mismo comando.

#### 3. Node.js y npm

El camino principal del curso usa Docker Compose, pero Node.js permite ejecutar servicios, clientes o pruebas de forma individual cuando el docente lo solicite.

1. Instalar Node.js LTS 18 o superior.
2. Verificar:

   ```bash
   node --version
   npm --version
   ```

3. Si se ejecuta un servicio fuera de Docker, entrar a la carpeta del servicio y ejecutar `npm install` antes de iniciar scripts locales.
4. No ejecutar `npm install` desde la raíz esperando instalar todo el monorepo: cada microservicio o cliente mantiene su propio `package.json`.

#### 4. Cliente SQL

El proyecto crea tres bases PostgreSQL separadas para sostener la arquitectura por servicios. Necesita un cliente SQL para ejecutar los scripts seed de esta guía.

Opciones válidas:

- DBeaver o pgAdmin si prefiere interfaz gráfica.
- `psql` si prefiere terminal.

Después de levantar Docker Compose, deberá poder conectarse a:

| Base de datos | Host | Puerto local | Usuario |
| --- | --- | ---: | --- |
| `db_usuarios` | `localhost` | `5432` | `user_usuarios` |
| `db_agenda` | `localhost` | `5433` | `user_agenda` |
| `db_tutorias` | `localhost` | `5434` | `user_tutorias` |

Las contraseñas y nombres completos de base aparecen en la sección [Inicializar bases de datos](#inicializar-bases-de-datos).

#### 5. Cliente HTTP y navegador

Para la prueba integrada basta con el cliente web local, pero un cliente HTTP externo ayuda a repetir casos, agregar headers y capturar evidencia.

Instale o tenga disponible al menos una opción:

- Postman, Insomnia, Thunder Client o REST Client de VS Code.
- `curl` desde terminal.

También use un navegador actualizado para acceder a las interfaces web locales.

### Checklist antes de continuar

- [ ] `git --version` responde correctamente.
- [ ] Docker Desktop está abierto y listo.
- [ ] `docker compose version` o `docker-compose --version` responde correctamente.
- [ ] `node --version` muestra una versión 18 o superior si se trabajará fuera de Docker.
- [ ] Tiene instalado un cliente SQL o sabe usar `psql`.
- [ ] Tiene un cliente HTTP o puede usar `curl`.
- [ ] Está ubicado en la raíz del repositorio antes de ejecutar los comandos siguientes.

## Configurar variables de entorno

Desde la raíz del repositorio, crear los archivos `.env` locales a partir de las plantillas:

```bash
cp ms-auth/.env.example ms-auth/.env
cp ms-usuarios/.env.example ms-usuarios/.env
cp ms-agenda/.env.example ms-agenda/.env
cp ms-tutorias/.env.example ms-tutorias/.env
cp ms-notificaciones/.env.example ms-notificaciones/.env
cp client-mobile-sim/.env.example client-mobile-sim/.env
cp tracking-dashboard/.env.example tracking-dashboard/.env
```

> Los valores de ejemplo están pensados para desarrollo local. No deben usarse como secretos productivos.

## Levantar el ecosistema

```bash
docker compose up --build
```

Esperar a que los logs indiquen conexiones exitosas a PostgreSQL y RabbitMQ. El entorno local expone servicios de aplicación, bases de datos, RabbitMQ, Toxiproxy, Prometheus y Grafana según `docker-compose.yml`.

Resultado esperado después del arranque:

| Servicio o recurso | Puerto local | Resultado esperado |
| --- | ---: | --- |
| `ms-tutorias` | `3000` | API orquestadora disponible para crear solicitudes de tutoría. |
| `ms-usuarios` | `3001` | API de usuarios disponible para validar estudiantes y tutores. |
| `ms-agenda` | `3002` | API de agenda disponible para verificar y bloquear horarios. |
| `ms-notificaciones` | `3003` | Consumidor de notificaciones conectado a RabbitMQ. |
| `ms-auth` | `4000` | Servicio de autenticación disponible para emitir token. |
| Cliente web | `8080` | Formulario local para simular la solicitud de tutoría. |
| Dashboard de tracking | `9000` | Vista local para observar eventos del flujo. |
| RabbitMQ Management | `15672` | Panel disponible con usuario/contraseña local `rabbit` / `rabbit`. |
| Prometheus | `9091` | Interfaz de consultas disponible si el contenedor está activo. |
| Grafana | `3005` | Login local disponible; contraseña por defecto `local-dev-admin`. |

> Si un servicio no aparece disponible, no avance a la prueba integrada todavía. Primero revise logs, puertos ocupados, variables `.env` y estado de RabbitMQ/PostgreSQL.

## Inicializar bases de datos

Los contenedores PostgreSQL se crean vacíos. Ejecutar los siguientes scripts en cada base de datos.

### `db_usuarios` en puerto `5432`

Conexión:

| Campo | Valor |
| --- | --- |
| Host | `localhost` |
| Puerto | `5432` |
| Base de datos | `db_usuarios` |
| Usuario | `user_usuarios` |
| Contraseña | `password_usuarios` |

```sql
CREATE TABLE estudiantes (
    id VARCHAR(50) PRIMARY KEY,
    nombreCompleto VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    carrera VARCHAR(255)
);

CREATE TABLE tutores (
    id VARCHAR(50) PRIMARY KEY,
    nombreCompleto VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    especialidad VARCHAR(255)
);

INSERT INTO estudiantes (id, nombreCompleto, email, carrera) VALUES
('e12345', 'Ana Torres', 'ana.torres@universidad.edu', 'Ingeniería de Software'),
('e67890', 'Luis Garcia', 'luis.garcia@universidad.edu', 'Medicina');

INSERT INTO tutores (id, nombreCompleto, email, especialidad) VALUES
('t54321', 'Dr. Carlos Rojas', 'carlos.rojas@universidad.edu', 'Bases de Datos Avanzadas'),
('t09876', 'Dra. Elena Solano', 'elena.solano@universidad.edu', 'Cálculo Multivariable');
```

### `db_agenda` en puerto `5433`

Conexión:

| Campo | Valor |
| --- | --- |
| Host | `localhost` |
| Puerto | `5433` |
| Base de datos | `db_agenda` |
| Usuario | `user_agenda` |
| Contraseña | `password_agenda` |

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;

CREATE TABLE bloqueos (
    idBloqueo UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    idTutor VARCHAR(50) NOT NULL,
    fechaInicio TIMESTAMPTZ NOT NULL,
    duracionMinutos INTEGER NOT NULL,
    idEstudiante VARCHAR(50) NOT NULL,
    createdAt TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT uq_bloqueos_tutor_fecha_inicio UNIQUE (idTutor, fechaInicio)
);

CREATE INDEX idx_bloqueos_idTutor ON bloqueos(idTutor);

INSERT INTO bloqueos (idTutor, fechaInicio, duracionMinutos, idEstudiante) VALUES
('t54321', '2025-10-22T10:00:00.000Z', 60, 'e12345');
```

### `db_tutorias` en puerto `5434`

Conexión:

| Campo | Valor |
| --- | --- |
| Host | `localhost` |
| Puerto | `5434` |
| Base de datos | `db_tutorias` |
| Usuario | `user_tutorias` |
| Contraseña | `password_tutorias` |

```sql
CREATE EXTENSION IF NOT EXISTS pgcrypto;

CREATE TABLE tutorias (
    idTutoria UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    idEstudiante VARCHAR(50) NOT NULL,
    idTutor VARCHAR(50) NOT NULL,
    materia VARCHAR(255),
    fecha TIMESTAMPTZ NOT NULL,
    estado VARCHAR(50) NOT NULL CHECK (estado IN ('PENDIENTE', 'CONFIRMADA', 'FALLIDA', 'CANCELADA')),
    createdAt TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    updatedAt TIMESTAMPTZ DEFAULT CURRENT_TIMESTAMP,
    error VARCHAR(500)
);

CREATE INDEX idx_tutorias_idEstudiante ON tutorias(idEstudiante);
CREATE INDEX idx_tutorias_idTutor ON tutorias(idTutor);
CREATE INDEX idx_tutorias_estado ON tutorias(estado);

CREATE OR REPLACE FUNCTION trigger_set_timestamp()
RETURNS TRIGGER AS $$
BEGIN
    NEW.updatedAt = NOW();
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER set_timestamp
BEFORE UPDATE ON tutorias
FOR EACH ROW
EXECUTE PROCEDURE trigger_set_timestamp();
```

## Verificar accesos

| Recurso | URL | Resultado esperado |
| --- | --- | --- |
| Cliente web | <http://localhost:8080> | Formulario para solicitar tutorías. |
| Dashboard de tracking | <http://localhost:9000> | Carriles/listado de eventos listo para recibir trazas. |
| RabbitMQ Management | <http://localhost:15672> | Panel local disponible con usuario/contraseña por defecto `rabbit` / `rabbit`, sobreescribible por entorno. |
| Prometheus | <http://localhost:9091> | Interfaz local de consultas disponible si el contenedor está activo. |
| Grafana | <http://localhost:3005> | Login local disponible; contraseña por defecto `local-dev-admin`, sobreescribible por entorno. |

En RabbitMQ Management, revisar que la cola `notificaciones_email_queue` tenga consumidor activo cuando `ms-notificaciones` esté corriendo.

## Prueba mínima guiada

1. Abrir el cliente web en `http://localhost:8080`.
2. Abrir el dashboard en `http://localhost:9000`.
3. Obtener un token desde el cliente o desde el endpoint de autenticación expuesto por `ms-auth`.
4. Solicitar una tutoría con datos existentes en la base seed, por ejemplo:
   - estudiante: `e12345`;
   - tutor: `t54321`;
   - materia: una materia relacionada con el tutor;
   - fecha: un horario disponible distinto del bloqueo seed `2025-10-22T10:00:00.000Z`.
5. Verificar que el cliente recibe una respuesta con estado `CONFIRMADA`.
6. Revisar el dashboard para seguir los eventos entre servicios.
7. Revisar RabbitMQ Management y confirmar que la cola `notificaciones_email_queue` tenga consumidor activo o actividad asociada al flujo.
8. Revisar logs de los servicios relevantes para identificar la secuencia: autenticación, validación de usuario/tutor, bloqueo de agenda, registro de tutoría y notificación.
9. Opcional: si se usa un cliente HTTP externo, enviar un `X-Correlation-ID` propio para rastrear la solicitud en logs y eventos.

## Casos de comprobación local

| Caso | Acción sugerida | Resultado esperado |
| --- | --- | --- |
| Token | Obtener token mediante cliente o `ms-auth`. | Existe un JWT para invocar operaciones protegidas. |
| Happy path | Solicitar tutoría con estudiante y tutor seed en un horario disponible. | Respuesta `CONFIRMADA`, registro en `db_tutorias` y bloqueo nuevo en `db_agenda`. |
| Conflicto `409` | Intentar reservar el horario seed ya bloqueado para `t54321` en `2025-10-22T10:00:00.000Z` o repetir una reserva ya ocupada. | Respuesta de conflicto `409`; no debe duplicarse el bloqueo. |
| RabbitMQ | Observar `notificaciones_email_queue` y logs de `ms-notificaciones`. | Consumidor activo y evidencia de procesamiento de notificación/evento. |
| Métricas | Abrir Prometheus o consultar endpoints de métricas cuando estén disponibles. | Señales operativas visibles para sustentar observabilidad local. |

Estos casos no son un solucionario de PC03. Sirven para llegar al examen con evidencias concretas y evitar respuestas genéricas desconectadas del sistema.

## Pruebas de resiliencia

Para pruebas de conflicto `409`, DLQ, Circuit Breaker y Saga compensation, no improvisar fallas manuales. La evidencia académica se consolida en [`docs/pc03.md`](./pc03.md). Los detalles operativos se distribuyen en documentos especializados:

- [`docs/event-contracts.md`](./event-contracts.md) para colas, eventos y DLQ.
- [`docs/observability-runbook.md`](./observability-runbook.md) para diagnóstico, logs, métricas y fallas inducidas.
- [`docs/audit-compliance-matrix.md`](./audit-compliance-matrix.md) para criterios de evidencia, cierre y brechas.

La prueba de Saga compensation requiere activar explícitamente `ENABLE_DEMO_FAULT_INJECTION=true` y enviar el header `X-Demo-Fail-After-Bloqueo: true`. Sin ambas condiciones, el flujo normal no debe alterarse.

## Problemas frecuentes

| Problema | Señal típica | Qué revisar o corregir |
| --- | --- | --- |
| Docker no está ejecutándose | `docker compose` no conecta con el daemon. | Abrir Docker Desktop y esperar a que indique estado listo. |
| Puertos ocupados | Algún contenedor no arranca o aparece error de bind. | Liberar los puertos usados por esta guía o ajustar temporalmente el mapeo local. |
| Base de datos vacía o no inicializada | Estudiantes/tutores no existen o la tutoría falla por datos ausentes. | Ejecutar los scripts SQL de `db_usuarios`, `db_agenda` y `db_tutorias` en la base correcta. |
| RabbitMQ aún no está listo | Servicios reportan errores de conexión AMQP al inicio. | Esperar unos segundos, revisar logs y confirmar RabbitMQ Management en `http://localhost:15672`. |
| Token faltante o expirado | La API responde error de autenticación/autorización. | Obtener un token nuevo desde el cliente o `ms-auth` y reenviar la solicitud. |
| Toxiproxy o fault injection quedó activo | Fallas inesperadas al consultar usuarios o completar la saga. | Revisar configuración de Toxiproxy y desactivar headers/variables de inyección de fallas usados en pruebas. |
| Volúmenes con datos obsoletos | Los datos no coinciden con los scripts o persisten conflictos anteriores. | Usar `docker compose down -v` solo de forma intencional, sabiendo que elimina datos persistidos. Luego reinicializar las bases. |

## Limpieza básica

Para detener contenedores sin eliminar volúmenes:

```bash
docker compose down
```

Para eliminar también datos persistidos de PostgreSQL y RabbitMQ:

```bash
docker compose down -v
```

Usar `down -v` solo cuando se quiera reiniciar completamente el entorno local.
