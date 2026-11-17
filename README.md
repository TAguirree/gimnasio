Iron Peak — Gestión de gimnasios
Aplicación web full stack para administrar gimnasios: los socios reservan clases y ven la tienda, los administradores gestionan clases, productos y socios de su sede, y el superadmin supervisa todo el sistema.
<img width="1892" height="923" alt="image" src="https://github.com/user-attachments/assets/47e4a642-3357-42c4-96b5-438bc9c995a9" /> 
<img width="1856" height="897" alt="image" src="https://github.com/user-attachments/assets/22e981e0-f0b5-4544-9cd3-1700fffbd054" />
<img width="1882" height="916" alt="image" src="https://github.com/user-attachments/assets/fb04c180-bbac-494f-8c6a-6e3dc3753513" />
<img width="1847" height="880" alt="image" src="https://github.com/user-attachments/assets/13db21d5-24a5-4a62-8bc9-cfb9733cbef0" />
<img width="1860" height="851" alt="image" src="https://github.com/user-attachments/assets/45ffb55b-207f-4037-9638-9a2434735fd1" />
<img width="1823" height="877" alt="image" src="https://github.com/user-attachments/assets/9f7167cf-b681-4555-92a9-986672405b4b" />

Funcionalidades

La aplicación maneja tres roles, cada uno con sus permisos:

👤 Usuario (socio)
Ver el calendario de clases de su gimnasio y reservarlas o cancelarlas.
Ver sus reservas filtradas por estado: activas, completadas y canceladas.
Consultar su membresía.
Navegar la tienda: búsqueda de productos, indicador de stock y carrito de compras.
🛠️ Admin (dueño del gimnasio)
ABM de clases: alta, modificación, consulta y baja, asignando un profesor y controlando la capacidad.
ABM de productos de la tienda.
ABM de miembros: alta, modificación y baja de los socios de su sede.
Gestión de profesores y dashboard de administración.
🔑 Superadmin
Acceso a todas las funciones anteriores.
Gestión de gimnasios y administradores, con visibilidad de todas las sedes.
Además: validación en todos los formularios con avisos al usuario, rutas protegidas por rol y diseño responsive con tema oscuro.

Stack tecnológico
Capa	Tecnologías
Frontend	React, Vite, React Router, Context API, hooks (useState, useEffect, useContext)
Backend	Node.js, Express, API REST
Base de datos	MySQL 8 (mysql2, pool de conexiones)
Autenticación	JWT (jsonwebtoken) y hash de contraseñas con bcryptjs
Herramientas	Git y GitHub (ramas y Pull Requests), Postman, MySQL Workbench

Arquitectura
El frontend nunca se conecta a la base de datos: solo habla con la API. El backend valida los datos, aplica los permisos por rol y recién después consulta MySQL.

Backend en tres capas

Cada entidad (usuarios, clases, productos, reservas…) se organiza igual:

Capa:	Carpeta	Responsabilidad
Rutas:	backend/routes	Definen el método y la URL, y aplican los middlewares de autenticación y rol
Controladores:	backend/controllers	Leen la petición, validan, llaman al modelo y responden con el código HTTP correcto
Modelos:	backend/models	Ejecutan las consultas SQL (parametrizadas)

backend/
├── index.js              # Punto de entrada: middlewares, rutas y manejo de errores
├── db.js                 # Pool de conexiones a MySQL
├── generar.js            # Script que genera el CRUD base de una tabla
├── middleware/auth.js    # verificarToken y permitirRoles
├── routes/
├── controllers/
└── models/

src/
├── components/   # Componentes reutilizables (UI, layout, CRUD genérico)
├── features/     # Lógica por funcionalidad (auth, reservas, tienda)
├── hooks/        # Hooks propios (por ejemplo useFetch)
├── services/     # Llamadas a la API
├── lib/          # Utilidades (formatos, roles, filtros por gimnasio)
└── pages/        # Pantallas: public/, user/, admin/, superadmin/

Seguridad y decisiones técnicas
- Contraseñas hasheadas con bcrypt (con sal); nunca se guardan ni se devuelven en texto plano.
- Autenticación con JWT (vence a las 8 h) que lleva el id, el rol y el gimnasio del usuario.
- Autorización en el servidor: los middlewares verificarToken (401) y permitirRoles (403) protegen las rutas. Las rutas protegidas del frontend (RequireRole) mejoran la experiencia, pero la seguridad real está en la API.
- El rol nunca se toma del body en el registro: nadie puede crearse como admin.
- Mensaje único en el login ante credenciales incorrectas, para no revelar qué correos existen.
- Aislamiento por gimnasio: un admin solo ve y modifica los datos de su sede; el filtro está en el SQL.
- Consultas parametrizadas contra inyección SQL.
- Transacciones en operaciones de varios pasos, como la baja de un usuario con sus reservas y membresías (todo o nada).
- Control de cupo y duplicados en las reservas; cancelar cambia el estado y conserva el historial.
- Manejo centralizado de errores: los errores de MySQL (claves foráneas, duplicados) se traducen a respuestas claras (400, 409).
- Validación en tres niveles: formulario, controlador y restricciones de la base de datos.
- SQL directo en lugar de un ORM: se eligió para controlar consultas puntuales (joins para el cupo, transacciones) y mantener el proyecto simple.
- Proxy de Vite: el frontend usa rutas relativas (/api) sin URLs fijas ni problemas de CORS en desarrollo.

API
Todas las rutas cuelgan de /api. Las que requieren sesión esperan el header Authorization: Bearer <token>.

Recurso	Endpoints	Acceso
Auth:	POST /auth/registro · POST /auth/login · GET /auth/perfil	Público (perfil requiere token)
Clases:	GET /clases · GET /clases/:id	Usuario logueado
	POST /clases · PUT /clases/:id · DELETE /clases/:id	Admin, superadmin
Usuarios:	GET · GET /:id · POST · PUT /:id · DELETE /:id en /usuarios	Admin (solo socios de su sede), superadmin
Productos:	GET /productos · GET /productos/:id	Usuario logueado
	POST · PUT /:id · DELETE /:id en /productos	Admin, superadmin
Reservas:	GET /reservas · POST /reservas · PATCH /reservas/:id/cancelar	Usuario logueado
	DELETE /reservas/:id	Admin, superadmin

También existen endpoints para profesores, membresías, gimnasios y pedidos.

Códigos de respuesta usados: 200 OK · 201 creado · 400 datos inválidos · 401 sin sesión · 403 sin permiso · 404 no encontrado · 409 conflicto (correo repetido, baja con relaciones) · 500 error del servidor.

Contacto

Tatiana Aguirre

GitHub: @TAguirree
LinkedIn: [www.linkedin.com/in/tatiana-aguirre-42341732a]
Correo: [tatianaaguirre940@gmail.com]
