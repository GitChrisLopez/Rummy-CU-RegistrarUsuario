# Arquitectura de Software - Módulo Registrar Usuario (MVC)

Este módulo implementa el Caso de Uso Registrar Usuario del juego Rummy. Se ha diseñado bajo una arquitectura MVC (Modelo-Vista-Controlador) estricta, integrando el patrón Observer para la comunicación asíncrona entre la lógica de red y la interfaz gráfica.

Release: 08/12/2025
---

## 🛠️ Tecnologías Utilizadas

*   **Java**: Lenguaje de programación principal.
*   **Java Swing / JavaFX**: Utilizada para la interfaz de usuario.
*   **Maven / Gradle**: Para la gestión de dependencias y la construcción del proyecto.

---

## 🚀 Cómo Empezar

### 📋 Prerrequisitos

Asegúrate de tener instalado el JDK (Java Development Kit) en tu máquina.

### ⚙️ Instalación

1.  **Clonar el repositorio**:
    ```bash
    git clone https://github.com/GitChrisLopez/Rummy-CU-RegistrarUsuario
    ```
2.  **Navegar al directorio del proyecto**:
    ```bash
    cd proyecto-rummy
    ```
3.  **Hacer clean y build**:
    > [!NOTE]
    > Es crucial realizar un `clean and build with dependencies` antes de ejecutar. 

### ▶️ Ejecutar el Programa

> [!TIP]
> Para iniciar el juego, ejecuta el servidor Blackboard y despues clase `EnsambladoresMVC` ubicada en el paquete `Ensambladores`.

1.  **Ejecutar la clase principal**:
    Despues de clean and build correr la clase main.

## 🏗️ Arquitectura del Modulo

El flujo de registro separa la validación de datos, la comunicación de red y la presentación visual en tres capas distintas:

# 1. Módulo Pantalla Inicial (Lobby)
Este módulo es el punto de entrada. Su responsabilidad principal es establecer la intención del usuario (Host vs. Guest) y coordinar la navegación global hacia los otros casos de uso.

Vista (Interfaz Gráfica)

VistaLobby (JFrame):Presenta las opciones principales: "Crear Partida" y "Unirse a Partida".
Actúa como Observador (ObservadorLobby) del modelo. Reacciona a eventos como ACCESO_DENEGADO (si la partida está llena) o CREAR_PARTIDA (confirmación para avanzar al registro).

Controlador (Orquestador)

ControlCUPrincipal: Hub de Navegación: Es el controlador "padre". Mantiene referencias a los controladores de los subsistemas (Registro, Sala de Espera, Juego) y decide qué pantalla mostrar según el estado del modelo.
Gestión de Flujo: Recibe la intención de la vista (ej. "Quiero crear partida") y delega la petición al modelo. Si la respuesta es positiva, este controlador se encarga de cerrar el Lobby y abrir el módulo de Registrar Usuario.

Modelo (Lógica de Sesión)

ModeloCUPrincipal:Negociación de Red: Envía los comandos iniciales al servidor (SOLICITAR_CREACION o SOLICITAR_UNIRSE) incluyendo la IP y puerto local del cliente.
Recepción de Eventos: Escucha respuestas críticas del servidor (TCP) mediante PropertyChangeListener. Por ejemplo, si recibe PARTIDA-EXISTENTE, notifica a la vista para informar al usuario que no puede ser Host.

# 2. Módulo Registrar Usuario    
El flujo de registro separa la validación de datos, la comunicación de red y la presentación visual en tres capas distintas:

Vista (Interfaz Gráfica)
Responsable de capturar la interacción del usuario y mostrar el estado del registro.

RegistrarUsuario (JFrame): Ventana principal que gestiona la entrada del Nickname, la selección visual del Avatar y actúa como contenedor principal. Implementa la interfaz ObservadorRegistro para reaccionar a eventos del modelo (como REGISTRO_EXITOSO o NOMBRE_REPETIDO).
leccionColores (JFrame): Sub-ventana auxiliar que permite al usuario personalizar los colores de sus 4 sets de fichas, validando que no existan colores duplicados antes de confirmar.

Controlador (Coordinación)
Actúa como intermediario, procesando las acciones de la vista y delegando la lógica al modelo o la navegación.

ControladorRegistro:Validación: Verifica que el nickname no esté vacío y convierte los objetos de la vista (como Color o índices de avatar) en datos primitivos manejables.
Interacción: Recibe el evento del botón "Registrar" y llama al método registrarUsuario del modelo.
Navegación: Coordina el cambio de pantalla hacia la "Sala de Espera" (entrarSalaEspera) una vez que el registro es exitoso.

Modelo (Lógica y Red)
Encapsula la lógica de negocio y la comunicación con el servidor (Blackboard).

ModeloRegistro: Protocolo de Red: Serializa los datos del usuario (Nickname, ID Avatar, Colores RGB) en una trama de texto bajo el comando ACTUALIZAR_PERFIL y la envía a través del iDespachador.
Gestión de Eventos: Escucha las respuestas del servidor (via sockets) y notifica a la Vista mediante el patrón Observer si el registro fue aprobado o si hubo un error (ej. nombre duplicado).
SesionUsuario: Clase de utilidad (estática) que almacena localmente los datos del jugador (perfil y configuración de colores) una vez que el registro se ha completado.

# 3. Módulo Sala de Espera
Una vez registrado el usuario, este módulo se encarga de la sincronización y espera activa hasta que todos los jugadores estén listos para comenzar.

Vista (Interfaz Gráfica)

VistaSalaEspera (JFrame): Visualización Dinámica: Muestra en tiempo real quiénes están conectados, renderizando sus nombres y avatares (decodificados desde los strings del protocolo).
Interacción de Estado: Contiene el botón "Iniciar Partida" (o "Estoy Listo"). Este botón se habilita/deshabilita dinámicamente dependiendo de si hay suficientes jugadores conectados (mínimo 2).

Controlador (Gestión de Voto)

ControlSalaDeEspera: Procesamiento de Acción: Captura el clic del usuario en "Estoy Listo" y ordena al modelo emitir el voto.
Inyección de Dependencias: Configura el modelo con la información del cliente (ID, conexiones) recibida de las etapas anteriores.

Modelo (Sincronización)

ModeloSalaDeEspera: Protocolo de Votación: Envía el comando ESTOY_LISTO al servidor cuando el usuario confirma.
Parsing de Datos: Recibe cadenas de texto "sucias" del servidor (ej. ID1,Juan$Avatar1;ID2,Pepe$Avatar2) a través del evento ACTUALIZAR_SALA. Procesa esta información y notifica a la vista (ACTUALIZAR_DATOS_JUGADORES) para que repinte la lista de jugadores.

## CONTENIDO DE MVCEjercerTurno/MVCJuego:
*   **`Vista`**: Responsable de toda la logica de presentación, pintar objetos y repintar los objetos de presentación.
(Obtiene los datos gracias a la implementacion de el patron observer)
*   **`Controlador`**: Responsable de atender las llamadas de la vista, dirigiendolas hacia el modelo.
*   **`Modelo`**: Responsable de la logica principal de el juego y dirigir llamadas a entidades necesarias para validar las reglas de el juego. (Se comunica con la vista a traves de segregar una interfaz con metodos para obtener datos, pasandoselo como notificacion a los observadores de esta misma). 

---

## 👨‍💻 Alumno
- Chris Fitch 00000252379

