1. Configuración del Proyecto

La primera etapa consistió en configurar el entorno de desarrollo. Utilizamos Node.js como nuestro entorno de ejecución y Visual Studio Code como editor de código. Para inicializar el proyecto, ejecutamos:

npm init -y

Esto generó un archivo package.json, que contiene información del proyecto y las dependencias necesarias.

Luego instalamos los paquetes clave:

npm install express body-parser cors mongoose jsonwebtoken bcrypt

Express: Framework para construir aplicaciones web y APIs.

Body-parser: Procesa los datos enviados en el cuerpo de las solicitudes.

CORS: Habilita el acceso de clientes desde dominios diferentes.

Mongoose: Conecta nuestra API con MongoDB.

JSON Web Tokens (JWT): Permite manejar autenticación basada en tokens.

Bcrypt: Utilizado para encriptar contraseñas.

2. Estructura de Archivos

Organizamos el proyecto de manera modular:

app.js: Punto de entrada principal que configura y ejecuta el servidor.

models/todoModel.js: Define el esquema de datos para nuestras tareas (To-Do).

models/userModel.js: Define el esquema de datos para los usuarios.

routes/todoRoutes.js: Contiene las rutas para manejar las solicitudes CRUD.

routes/userRoutes.js: Contiene las rutas para autenticación y gestión de usuarios.

controllers/todoController.js: Implementa la lógica de negocio para cada operación CRUD.

controllers/userController.js: Implementa la lógica de negocio para registro e inicio de sesión.

middleware/authMiddleware.js: Middleware para proteger rutas con autenticación JWT.

3. Implementación de la API

La API que desarrollamos permite manejar tareas a través de las operaciones CRUD y gestiona la autenticación de usuarios.

3.1 Implementación de Autenticación

Se implementó un sistema de autenticación básico utilizando JWT:

Registro de Usuario (POST /users/register):

const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');

exports.registerUser = async (req, res) => {
    try {
        const hashedPassword = await bcrypt.hash(req.body.password, 10);
        const user = await User.create({ ...req.body, password: hashedPassword });
        res.status(201).json({ message: 'Usuario registrado exitosamente' });
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
};

Inicio de Sesión (POST /users/login):

exports.loginUser = async (req, res) => {
    try {
        const user = await User.findOne({ email: req.body.email });
        if (!user || !(await bcrypt.compare(req.body.password, user.password))) {
            return res.status(401).json({ error: 'Credenciales inválidas' });
        }
        const token = jwt.sign({ id: user._id }, process.env.JWT_SECRET, { expiresIn: '1h' });
        res.json({ token });
    } catch (error) {
        res.status(400).json({ error: error.message });
    }
};

Protección de Rutas:
Creamos un middleware para verificar tokens:

const jwt = require('jsonwebtoken');

exports.protect = (req, res, next) => {
    const token = req.headers.authorization?.split(' ')[1];
    if (!token) return res.status(401).json({ error: 'No autorizado' });

    jwt.verify(token, process.env.JWT_SECRET, (err, decoded) => {
        if (err) return res.status(401).json({ error: 'Token inválido' });
        req.userId = decoded.id;
        next();
    });
};

3.2 CRUD Protegido

Todas las rutas CRUD de tareas están protegidas, permitiendo que solo usuarios autenticados puedan acceder:

router.route('/todos').get(protect, getTodos).post(protect, createTodo);
router.route('/todos/:id').put(protect, updateTodo).delete(protect, deleteTodo);

3.3 Filtrado de Datos

Se agregaron funcionalidades para filtrar tareas por creador o estado de completado. Los filtros se implementan mediante parámetros en la URL:

exports.getFilteredTodos = async (req, res) => {
    const { creator, completed } = req.query;
    const query = {};
    if (creator) query.creator = creator;
    if (completed !== undefined) query.completed = completed === 'true';

    const todos = await Todo.find(query);
    res.json(todos);
};

4. Pruebas con Postman

Para asegurar el correcto funcionamiento de la API, utilizamos Postman, una herramienta que facilita la prueba de endpoints. Configuramos solicitudes para:

Registro e inicio de sesión de usuarios.

Operaciones CRUD protegidas mediante tokens JWT.

Filtrado de tareas por creador y estado de completado.

5. Documentación y Entrega Final

Documentamos el proyecto en un archivo README.md, detallando:

Instalación y configuración: Iniciar los servidores backend y frontend.

Pruebas: Instrucciones para probar la API con Postman.

Endpoints disponibles: Lista de rutas, métodos y parámetros aceptados.

El código se subió a un repositorio de GitHub, asegurándonos de que el historial de commits refleje cada etapa del desarrollo e integración. Esto facilita la colaboración y el mantenimiento del proyecto.