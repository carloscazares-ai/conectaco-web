CONECTACO

conecTaco es una plataforma digital que conecta a turistas con lugares, actividades y negocios de la Ciudad de México mediante una experiencia personalizada y gamificada. El sistema contará con dos tipos de usuarios: turistas, quienes podrán seleccionar sus intereses, recibir recomendaciones, explorar lugares mediante un mapa interactivo, participar en actividades y obtener recompensas que les permitan adquirir experiencia y personalizar una mascota virtual; y negocios, quienes podrán registrar y promocionar sus establecimientos, crear actividades, promociones y recompensas para atraer visitantes. De esta manera, conecTaco busca incentivar la exploración de CDMX y, al mismo tiempo, brindar a los negocios una herramienta de promoción e interacción con potenciales clientes. 

BASE DE DATOS

![conectaco.png](docs/conectaco.png)

---

## Cómo correr el proyecto

### Lo que necesitas
- **IntelliJ IDEA**
- **Java 25** (IntelliJ lo puede descargar: File → Project Structure → SDK)
- **MySQL 8** y **MySQL Workbench**
- **Postman** (para probar los endpoints)

### 1. Clonar el repositorio
En IntelliJ: **File → New → Project from Version Control**, pega la URL del repositorio y dale **Clone**. Espera a que Maven termine de descargar las dependencias (barra de abajo).

### 2. Crear la base de datos
1. Abre **MySQL Workbench** y conéctate a tu servidor local.
2. Abre el archivo `database/conectaco.sql` (File → Open SQL Script).
3. Ejecútalo completo con el rayito ⚡. Se crea el esquema `conectaco` con todas sus tablas.

> Si el script cambia (alguien agrega o modifica tablas), tienes que volver a correrlo. Si ya tenías el esquema creado, primero bórralo con `DROP SCHEMA conectaco;` (se pierden tus datos de prueba).

### 3. Poner TU contraseña de MySQL (importante)
La contraseña **no** está en el repositorio: cada quien pone la suya en un archivo local que Git ignora.

1. En IntelliJ ve a `src/main/resources/`.
2. Copia `application-local.properties.example` y pégalo en la misma carpeta con el nombre **`application-local.properties`** (clic derecho → Copy, luego clic derecho en la carpeta → Paste y cambia el nombre).
3. Ábrelo y pon tu usuario y contraseña de MySQL:
   ```properties
   spring.datasource.username=root
   spring.datasource.password=la_contraseña_de_tu_mysql
   ```

Ese archivo **nunca se sube a GitHub** (está en `.gitignore`), así que no pisas la contraseña de nadie. **No pongas tu contraseña en `application.properties`**.

### 4. Arrancar la aplicación
Abre `src/main/java/com/conectaco/conectaco/ConectacoApplication.java` y dale al botón verde ▶️. Cuando en la consola aparezca `Started ConectacoApplication`, ya está corriendo en **http://localhost:8080**.

Errores comunes:
| Error | Solución |
|---|---|
| `Port 8080 was already in use` | Ya tienes otra copia corriendo. Detenla con el cuadro rojo ⏹️ y vuelve a darle ▶️. |
| `Access denied for user 'root'` | La contraseña de `application-local.properties` está mal o no creaste el archivo. |
| `Unknown database 'conectaco'` | No corriste el script `database/conectaco.sql`. |

### 5. Probar en Postman
En Postman: **Import** → elige `postman/conectaco.postman_collection.json`. Ahí vienen todas las peticiones listas; abre una y dale **Send**. También puedes correrlas todas: `...` de la colección → **Run collection**.

| Método | URL | Qué hace |
|---|---|---|
| GET | http://localhost:8080/test/hola | Prueba que el servidor responde |
| GET | http://localhost:8080/test/carlos | Endpoint de prueba de Carlos |
| GET | http://localhost:8080/v1/usuarios | Lista todos los usuarios |
| POST | http://localhost:8080/v1/usuarios | Crea un usuario |

Ejemplo de body para el POST (Body → raw → JSON):
```json
{
  "nombre": "Ana",
  "email": "ana@test.com",
  "password": "1234",
  "tipoUsuario": "Turista"
}
```
`tipoUsuario` solo acepta `Turista` o `Negocio`. El `email` no se puede repetir.

---

## Cómo trabajar en equipo

### Ramas
| Rama | Para qué |
|---|---|
| `dev` | **La rama principal.** Aquí se junta el trabajo de todos y es lo que se entrega |
| `tu-nombre` | Tu rama personal; aquí trabajas tú |

No hay rama `main`: **`dev` es la principal**.

**`dev` está protegida: nadie puede hacer push directo.** Todo entra a `dev` con un Pull Request, y otra persona del equipo (no quien lo abrió) tiene que aprobarlo antes del merge.

### Antes de empezar a trabajar (cada vez)
1. Cámbiate a tu rama (abajo a la derecha en IntelliJ, el nombre de la rama → tu rama → **Checkout**).
2. Trae lo último de `dev`: en el menú de ramas, clic en `origin/dev` → **Merge 'origin/dev' into tu-rama**. Así no se te acumulan conflictos.
3. Si cambió `database/conectaco.sql`, vuelve a correr el script.

La primera vez: tu rama ya existe en GitHub. Haz **Git → Fetch** y luego, en el menú de ramas, `origin/tu-nombre` → **Checkout**.

### Cuando terminas algo
1. **Git → Commit**: selecciona tus archivos, escribe qué hiciste (ej. "Se creo Lugar: entity, dto, mapper, repository, service y controller") y dale **Commit and Push**.
2. En GitHub entra al repo → **Pull requests → New pull request** → base: `dev` ← compare: `tu-rama` → **Create pull request**.
3. Avisa al equipo para que alguien lo revise y le dé **Merge**.

Revisa antes de hacer commit que **no** vaya incluido `application-local.properties` ni la carpeta `target/`.

### Cómo crear un módulo nuevo (ej. Lugar)
Usa Usuarios como plantilla. Por cada tabla se crean estos archivos dentro de `src/main/java/com/conectaco/conectaco/`:

| Carpeta | Archivo | Qué es |
|---|---|---|
| `entity` | `LugarEntity` | La tabla de MySQL en Java (`@Entity`, `@Table(name = "Lugar")`, una variable por columna) |
| `dto` | `LugarDTO` | Lo que viaja en el JSON (mismos campos, sin anotaciones) |
| `mapper` | `LugarMapper` + `impl/LugarMapperImpl` | Convierte Entity ↔ DTO |
| `repository` | `LugarRepository` | `extends JpaRepository<LugarEntity, Integer>` |
| `service` | `LugarService` + `impl/LugarServiceImpl` | La lógica (`save`, `findAll`) |
| `controller` | `LugarController` | Los endpoints (`@GetMapping`, `@PostMapping` en `/v1/lugares`) |

Al terminar, agrega tus peticiones a la colección de Postman (o pásaselas a quien lleve la documentación).
