# Blog API con Prisma

Aprendiendo a usar schema.prisma

## 📖 Explicación del Código

Este proyecto es una guía para aprender a crear una API de Blog utilizando **Prisma** como ORM (Object-Relational Mapping).

### ¿Qué es Prisma?

Prisma es un ORM moderno para Node.js y TypeScript que facilita el acceso a bases de datos. Proporciona:

- **Prisma Client**: Un cliente de base de datos auto-generado y type-safe
- **Prisma Migrate**: Un sistema de migraciones declarativo
- **Prisma Studio**: Una interfaz visual para explorar y editar datos

### 📁 Estructura del Proyecto

Un proyecto típico de Blog API con Prisma tendría la siguiente estructura:

```
blog-api-ivan/
├── prisma/
│   └── schema.prisma    # Definición del esquema de base de datos
├── src/
│   ├── index.ts         # Punto de entrada de la aplicación
│   └── routes/          # Rutas de la API
├── package.json         # Dependencias del proyecto
└── README.md            # Documentación
```

### 📝 Schema de Prisma (schema.prisma)

El archivo `schema.prisma` es el corazón de Prisma. Define:

1. **Datasource**: La conexión a la base de datos
2. **Generator**: Configuración del cliente Prisma
3. **Models**: Las tablas de la base de datos

#### Ejemplo de Schema para un Blog:

```prisma
// Configuración de la base de datos
datasource db {
  provider = "postgresql"  // Puede ser: mysql, sqlite, sqlserver, mongodb
  url      = env("DATABASE_URL")
}

// Generador del cliente Prisma
generator client {
  provider = "prisma-client-js"
}

// Modelo de Usuario
model User {
  id        Int      @id @default(autoincrement())  // Llave primaria auto-incremental
  email     String   @unique                         // Campo único
  name      String?                                  // Campo opcional (puede ser null)
  posts     Post[]                                   // Relación uno-a-muchos con Post
  createdAt DateTime @default(now())                 // Fecha de creación automática
  updatedAt DateTime @updatedAt                      // Fecha de actualización automática
}

// Modelo de Publicación (Post)
model Post {
  id        Int       @id @default(autoincrement())
  title     String                                   // Título del post
  content   String?                                  // Contenido (opcional)
  published Boolean   @default(false)                // Estado de publicación
  author    User      @relation(fields: [authorId], references: [id], onDelete: Cascade)
  authorId  Int                                      // Llave foránea hacia User
  comments  Comment[]                                // Relación con comentarios
  createdAt DateTime  @default(now())
  updatedAt DateTime  @updatedAt
}

// Modelo de Comentario
model Comment {
  id        Int      @id @default(autoincrement())
  content   String                                   // Contenido del comentario
  post      Post     @relation(fields: [postId], references: [id], onDelete: Cascade)
  postId    Int                                      // Llave foránea hacia Post
  createdAt DateTime @default(now())
}
```

### 🔍 Explicación de los Decoradores

| Decorador | Descripción |
|-----------|-------------|
| `@id` | Define la llave primaria de la tabla |
| `@default()` | Establece un valor por defecto |
| `@unique` | Asegura que el valor sea único en la tabla |
| `@relation()` | Define relaciones entre modelos. Soporta opciones `onDelete` y `onUpdate` para cascadas |
| `@updatedAt` | Actualiza automáticamente con la fecha actual |
| `autoincrement()` | Genera un número secuencial automático |
| `now()` | Obtiene la fecha/hora actual |

#### Opciones de Cascada (onDelete/onUpdate)

| Opción | Descripción |
|--------|-------------|
| `Cascade` | Elimina/actualiza registros relacionados automáticamente |
| `Restrict` | Previene la eliminación si hay registros relacionados |
| `SetNull` | Establece la llave foránea a null |
| `NoAction` | Similar a Restrict (depende de la base de datos) |
| `SetDefault` | Establece la llave foránea al valor por defecto |

### 💻 Uso del Prisma Client

#### Instalación

```bash
# Instalar Prisma como dependencia de desarrollo
npm install prisma --save-dev

# Instalar el cliente de Prisma
npm install @prisma/client

# Inicializar Prisma en el proyecto
npx prisma init

# Generar el cliente después de cambios en el schema
npx prisma generate

# Ejecutar migraciones
npx prisma migrate dev --name init
```

#### Ejemplos de Operaciones CRUD

```typescript
import { PrismaClient } from '@prisma/client'

// Crear una instancia del cliente (usar patrón singleton en producción)
const prisma = new PrismaClient()

// IMPORTANTE: Cerrar la conexión cuando termine la aplicación
// prisma.$disconnect()

// CREATE - Crear un nuevo usuario
async function createUser() {
  const user = await prisma.user.create({
    data: {
      email: 'usuario@ejemplo.com',
      name: 'Juan Pérez'
    }
  })
  return user
}

// READ - Obtener todos los posts publicados
async function getPublishedPosts() {
  const posts = await prisma.post.findMany({
    where: { published: true },
    include: { 
      author: true,      // Incluir datos del autor
      comments: true     // Incluir comentarios
    }
  })
  return posts
}

// UPDATE - Actualizar un post
async function publishPost(postId: number) {
  const post = await prisma.post.update({
    where: { id: postId },
    data: { published: true }
  })
  return post
}

// DELETE - Eliminar un post
async function deletePost(postId: number) {
  await prisma.post.delete({
    where: { id: postId }
  })
}

// Crear un post con autor (relación)
async function createPost(authorId: number) {
  const post = await prisma.post.create({
    data: {
      title: 'Mi primer post',
      content: 'Contenido del post...',
      author: {
        connect: { id: authorId }  // Conectar con usuario existente
      }
    }
  })
  return post
}
```

### 🔗 Tipos de Relaciones

1. **Uno a Uno (1:1)**: Un usuario tiene un perfil único
2. **Uno a Muchos (1:N)**: Un usuario tiene muchos posts
3. **Muchos a Muchos (M:N)**: Posts pueden tener múltiples categorías

### 📚 Recursos Adicionales

- [Documentación oficial de Prisma](https://www.prisma.io/docs)
- [Prisma Schema Reference](https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference)
- [Ejemplos de Prisma](https://github.com/prisma/prisma-examples)

## 🚀 Próximos Pasos

1. Configurar la base de datos
2. Crear el schema.prisma
3. Ejecutar las migraciones
4. Implementar las rutas de la API
5. Probar con Prisma Studio (`npx prisma studio`)
