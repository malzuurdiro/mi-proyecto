
## 1. Instalación del entorno (Ubuntu 22.04 + ROS2 Humble)
 
### 1.1. Instalar ROS2 Humble
 
```bash
sudo apt update && sudo apt install -y software-properties-common curl
sudo add-apt-repository universe
sudo curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
 
# OJO: se fija "jammy" explícitamente (codename de Ubuntu 22.04) en vez de
# depender de $UBUNTU_CODENAME, que en algunos shells no se sustituye bien
# dentro del echo y deja el fichero de repositorio vacío o mal formado.
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu jammy main" | sudo tee /etc/apt/sources.list.d/ros2.list
```
 
**Verificación antes de continuar** (si algo de esto falla, NO sigas al
siguiente paso, revisa el error primero):
 
```bash
lsb_release -cs                        # debe imprimir: jammy
cat /etc/apt/sources.list.d/ros2.list  # debe mostrar una línea "deb [...] jammy main", sin variables sin sustituir
ls /usr/share/keyrings/ros-archive-keyring.gpg   # debe existir
```
 
Si `cat /etc/apt/sources.list.d/ros2.list` te sale vacío o con `$UBUNTU_CODENAME`
literal sin sustituir, es la causa del error `Unable to locate package
ros-humble-desktop`: bórralo y repite el `echo ... | sudo tee ...` de
arriba (con "jammy" ya fijo, no debería volver a pasar).
 
```bash
sudo apt update
sudo apt install -y ros-humble-desktop python3-argcomplete
```
 
 
### 1.2. Instalar TurtleBot3 y su simulación en Gazebo
 
```bash
sudo apt install -y ros-humble-turtlebot3 ros-humble-turtlebot3-simulations ros-humble-gazebo-ros-pkgs
 
# Modelo de TurtleBot3 a usar (burger es el más simple y habitual para docencia)
echo "export TURTLEBOT3_MODEL=burger" >> ~/.bashrc
 
# Cargar el entorno en cada terminal nueva
echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc
source ~/.bashrc
```
 
## 2. Crear el workspace del PFG y compilar los paquetes
 
Este bloque ya viene como **dos paquetes ROS2 reales** (no scripts
sueltos): `pfg_tb3_interfaces` (define la action personalizada `Rotate`)
y `pfg_tb3_examples` (los nodos de ejemplo), empaquetados juntos en
`pfg_dia1_paquetes_ros2.zip`. Como trabajas en WSL, las descargas hechas
desde el navegador de Windows aparecen en Linux dentro de
`/mnt/c/Users/name/Downloads/` — descomprime el zip directamente desde
ahí a tu workspace:
 
```bash
mkdir -p ~/pfg_ws/src
cd ~/pfg_ws/src
unzip /mnt/c/Users/name/Downloads/pfg_dia1_paquetes_ros2.zip
cd ~/pfg_ws
colcon build
echo "source ~/pfg_ws/install/setup.bash" >> ~/.bashrc
source ~/.bashrc
```
 
Si `colcon build` da un error de dependencias, instala los paquetes que
falten con `rosdep`:
 
```bash
sudo rosdep init   # solo la primera vez que se usa rosdep en el sistema
rosdep update
rosdep install --from-paths src --ignore-src -r -y
```
 
## 3. Lanzar la simulación
 
```bash
# Terminal 1 (dejar corriendo siempre)
source /opt/ros/humble/setup.bash
export TURTLEBOT3_MODEL=burger
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
```
 
## 4. Ejecutar los ejemplos
 
- **Nodo + Topic (publisher):**
  `ros2 run pfg_tb3_examples publisher_node` → el robot empieza a moverse
  en círculo.
- **Nodo + Topic (subscriber):**
  `ros2 run pfg_tb3_examples subscriber_node` → muestra x, y, yaw en
  tiempo real (ejecutar junto con el publisher para ver datos cambiando).
- **Servicio:**
  `ros2 run pfg_tb3_examples service_client_node` → reinicia la
  simulación de Gazebo a su estado inicial (servicio `/reset_simulation`).
- **Action (servidor + cliente):**
```bash
  # Terminal A
  ros2 run pfg_tb3_examples rotate_action_server
  # Terminal B
  ros2 run pfg_tb3_examples rotate_action_client
```
  El robot gira 90° mostrando el ángulo restante como feedback por
  consola en ambas terminales.
 
- **rviz2:**
```bash
  rviz2
```
  Al abrirse, rviz2 aparece vacío (sin nada visualizado) — hay que añadir
  manualmente cada elemento desde la interfaz:
 
  1. Arriba a la izquierda, en el panel **"Displays"**, pulsa el botón
     **"Add"** (abajo del todo de ese panel).
  2. Se abre una ventana con dos pestañas: **"By display type"** y
     **"By topic"**. Usa la pestaña **"By topic"** — es más fácil
     encontrar lo que buscas ahí.
  3. Añade estos tres elementos, uno por uno (repite el botón "Add" para
     cada uno):
     - **RobotModel**: en "By display type", busca "RobotModel" en la
       lista y pulsa Ok. Esto dibuja la forma 3D del robot.
     - **TF**: en "By display type", busca "TF" y pulsa Ok. Esto dibuja
       los ejes de coordenadas de cada parte del robot.
     - **LaserScan**: en "By topic", busca `/scan` → `LaserScan` y pulsa
       Ok. Esto dibuja los puntos detectados por el láser del TurtleBot3.
  4. Arriba del panel "Displays" hay un desplegable **"Fixed Frame"**
     (dentro de "Global Options") — cámbialo de `map` a **`odom`**.
  5. Si con esto no ves el robot, prueba `Ctrl+scroll` o el ratón para
     alejar/acercar la cámara — a veces la vista inicial no encuadra el
     modelo.
  **Si ves los ejes (TF) y el láser pero NO el robot en sí** (caso
  típico), revisa esto en orden:
 
  1. En el panel "Displays", pulsa sobre la entrada **"RobotModel"** para
     desplegar sus opciones. Fíjate si el nombre aparece en **rojo** o
     con un icono de aviso — eso indica un error concreto, normalmente
     "Could not load resource" o "package not found". Si ves un mensaje
     así, dínoslo tal cual y lo resolvemos (casi siempre es que el
     paquete `turtlebot3_description` no está instalado en el mismo
     entorno desde el que abres rviz2, o que no has hecho
     `source /opt/ros/humble/setup.bash` en esa terminal).
  2. Comprueba que la casilla junto a "RobotModel" está **marcada**
     (habilitada) — a veces se añade el display pero queda desactivado.
  3. Dentro de "RobotModel", busca el campo **"Description Topic"** (o
     "Description Source" en versiones más nuevas) y confirma que apunta
     a `/robot_description` — si está vacío o apunta a otro sitio, no
     encontrará el modelo aunque TF sí funcione (TF no depende del URDF,
     por eso puede verse igualmente).
  4. Comprueba que **"Alpha"** (dentro de RobotModel) no está a 0 — si
     está a 0 el modelo se carga pero es invisible.
  5. Si nada de esto lo soluciona, ejecuta en una terminal nueva
     `ros2 topic echo /robot_description --once` — si no devuelve nada,
     el problema no está en rviz2 sino en que el robot_state_publisher no
     está publicando el URDF (revisa que el launch de Gazebo del paso 3
     sigue corriendo sin errores).
 
## 5. Comandos de inspección útiles
 
```bash
ros2 node list
ros2 node info /circle_publisher
ros2 topic list
ros2 topic echo /odom
ros2 topic hz /cmd_vel
ros2 service list
ros2 action list
rqt_graph        # captura visual del grafo de nodos/topics
```
 
## 6. Visualizar el robot FANUC CRX-10iA/L en RViz (Bloque 4-5)
 
Esta sección es independiente de todo lo anterior (TurtleBot3) — es la
puesta en marcha de `fanuc_driver` que corresponde al Capítulo 5 de la
memoria. **Importante: cerrar antes cualquier simulación de TurtleBot3**
(Terminal 1 del paso 3) y cualquier rviz2 que tengas abierto — si se
quedan corriendo de fondo, rviz2 seguirá mostrando el TurtleBot3 en vez
del FANUC, porque ambos compiten por el mismo topic `/robot_description`.
 
```bash
# Comprobación rápida de que no queda nada de TurtleBot3/Gazebo corriendo
ros2 node list
# Si aparece algo relacionado con turtlebot3 o gazebo, ciérralo primero
# (Ctrl+C en su terminal) antes de continuar.
```
 
### 6.1. Instalar y compilar fanuc_description + fanuc_driver
 
Además de los dos repositorios oficiales de FANUC, descomprime también
el paquete `pfg_dia2_paquete_control_fanuc.zip` (los ejemplos de
control del Capítulo 6) desde tu carpeta de descargas de Windows:
 
```bash
mkdir -p ~/fanuc_ws/src && cd ~/fanuc_ws/src
unzip /mnt/c/Users/name/Downloads/pfg_dia2_paquete_control_fanuc.zip
git clone https://github.com/FANUC-CORPORATION/fanuc_description.git
git clone --branch humble --recurse-submodules https://github.com/FANUC-CORPORATION/fanuc_driver.git
 
cd ~/fanuc_ws
rosdep update
rosdep install --ignore-src --from-paths src -y
colcon build --symlink-install
echo "source ~/fanuc_ws/install/setup.bash" >> ~/.bashrc
source ~/.bashrc
```
 
**Nota importante:** solo `fanuc_driver` necesita indicar la rama
`humble` (porque su rama `main` apunta a ROS2 Jazzy, incompatible con tu
Humble). `fanuc_description` no tiene una rama separada llamada
`humble` — se clona con su rama por defecto, que sirve igual para
Humble y Jazzy. Si pones `--branch humble` también en
`fanuc_description`, obtendrás el error `fatal: Remote branch humble
not found in upstream origin`.
 
**Otra nota importante:** `fanuc_driver` incluye una dependencia externa
(la librería `sockpp`) como submódulo git, por lo que el clonado
necesita `--recurse-submodules`. Si ya clonaste sin ese flag y
`colcon build` falla en el paquete `fanuc_libs` con un error de tipo
`fatal: repository '.../dependencies/sockpp' does not exist`, no hace
falta volver a clonar desde cero — entra en la carpeta ya clonada y
descarga los submódulos que faltan:
```bash
cd ~/fanuc_ws/src/fanuc_driver
git submodule update --init --recursive
cd ~/fanuc_ws
colcon build --symlink-install
```
 
Esto es lo mismo que el Código 2 de la memoria: descarga el paquete de
modelos 3D del robot (`fanuc_description`) y el driver ROS2
(`fanuc_driver`), ambos en la rama `humble` (la que corresponde a tu
versión de ROS2), y los compila dentro de un workspace propio
(`~/fanuc_ws`, separado de `~/pfg_ws` del TurtleBot3, para no mezclar
ambos proyectos), junto con el paquete `pfg_fanuc_control` con los
ejemplos de control (Código 5 y 6 de la memoria).
 
### 6.2. Ver solo el modelo del robot (con sliders articulares)
 
```bash
ros2 launch fanuc_crx_description view_crx.launch.py robot_model:=crx10ia_l
```
 
Este comando (Código 3 de la memoria) abre rviz2 ya con el `RobotModel`
y el `TF` del CRX-10iA/L añadidos automáticamente (no hace falta
añadirlos a mano como con el TurtleBot3), más un panel lateral con un
slider por cada articulación para moverlas manualmente una a una. Es la
forma más simple de comprobar que el modelo se ha cargado bien, sin
necesidad de arrancar ningún controlador.
 
### 6.3. Planificar y ejecutar movimientos con MoveIt2 (mock hardware)
 
```bash
ros2 launch fanuc_moveit_config fanuc_moveit.launch.py robot_model:=crx10ia_l use_mock:=true
```
 
Este es el Código 4 de la memoria. A diferencia del comando anterior
(que solo visualiza), este arranca `ros2_control` + MoveIt2 completos,
usando hardware simulado (`use_mock:=true`, sin física real, sin
Gazebo). En rviz2 verás además el plugin **"Motion Planning"** de
MoveIt2 en el panel de la izquierda:
 
1. En la vista 3D aparece un marcador naranja/interactivo (flechas y
   aros) en el extremo del robot — arrástralo para definir una pose
   objetivo.
2. En el panel "Motion Planning" (pestaña "Planning"), pulsa **"Plan &
   Execute"**.
3. Deberías ver primero una trayectoria previsualizada (línea/trail) y
   después el robot moviéndose realmente hasta la pose objetivo.

 
### 6.4. Ejecutar los ejemplos de control (Bloque 6)
 
Si aún no lo has hecho, primero descomprime el paquete `pfg_fanuc_control`
(contiene `send_trajectory_goal` y `send_position_goal`) dentro de tu
workspace del FANUC y compílalo:
 
```bash
cd ~/fanuc_ws/src
unzip /mnt/c/Users/name/Downloads/pfg_dia2_paquete_control_fanuc.zip
cd ~/fanuc_ws
colcon build --symlink-install
source ~/fanuc_ws/install/setup.bash
```
 
(Si ya lo descomprimiste antes y solo quieres asegurarte, comprueba con
`ls ~/fanuc_ws/src/pfg_fanuc_control` — si te lista archivos como
`package.xml` y `setup.py`, ya está hecho y puedes saltarte este bloque.)
 
Con MoveIt2/mock hardware ya lanzado (apartado 6.3), en una terminal
**nueva** con los dos `source` cargados (ROS2 + workspace, ver 6.5 si no
los tienes claros):
 
```bash
# Para qué sirve: envía una trayectoria de 2 puntos articulares al robot
# usando FollowJointTrajectory (Código 5 de la memoria). Verás por consola
# el feedback de posición articular mientras el robot se mueve en RViz.
ros2 run pfg_fanuc_control send_trajectory_goal
```
 
```bash
# Para qué sirve: envía una única consigna de posición directa al robot
# (sin trayectoria interpolada), usando forward_position_controller
# (Código 6 de la memoria).
ros2 run pfg_fanuc_control send_position_goal
```
 
### 6.5. Arranque rápido (cada vez que vuelvas a abrir Ubuntu/WSL)
 
Una vez instalado todo (apartados 6.1-6.3), **no hace falta repetir la
instalación cada día** — solo tienes que volver a cargar el entorno en
cada terminal nueva y relanzar. Estos son los únicos comandos que
necesitas a partir de ahora:
 
```bash
# En CADA terminal nueva que abras, antes de cualquier otro comando:
source /opt/ros/humble/setup.bash
source ~/fanuc_ws/install/setup.bash
```
 
Y luego, según lo que quieras abrir:
 
```bash
# Solo visualización con sliders (Código 3):
ros2 launch fanuc_crx_description view_crx.launch.py robot_model:=crx10ia_l
 
# RViz + MoveIt2 con mock hardware (Código 4), para planificar y ejecutar movimientos:
ros2 launch fanuc_moveit_config fanuc_moveit.launch.py robot_model:=crx10ia_l use_mock:=true
```
 
**Si el robot aparece pero no se mueve al pulsar "Plan & Execute"**
(típico tras reiniciar Ubuntu/WSL), la causa casi siempre es que en
alguna terminal falta el segundo `source` (el del workspace,
`~/fanuc_ws/install/setup.bash`) — sin él, rviz2 puede seguir mostrando
el modelo (lo carga una sola vez al lanzar) pero los controladores de
`ros2_control` no están realmente activos. Comprueba esto:
 
```bash
# En una terminal con AMBOS source ya cargados:
ros2 control list_controllers
```
Deberías ver `scaled_joint_trajectory_controller` en estado `active`. Si
sale `inactive`, o el comando falla, cierra todo (Ctrl+C en la terminal
donde lanzaste MoveIt2), abre una terminal nueva, haz los dos `source`
de arriba con cuidado de no saltarte ninguno, y vuelve a lanzar el
Código 4.
 
### 6.6. Bloque 8: puente ROS2 → OPC-UA (para RoboDK)
 
Este nodo (`joint_state_opcua_bridge`, ya incluido en el zip de
`pfg_fanuc_control` que descomprimiste en el apartado 6.1) necesita una
librería adicional que no viene con ROS2:
 
```bash
pip install asyncua --break-system-packages
```
 
Con MoveIt2/mock hardware ya lanzado (apartado 6.3 o 6.4), en una
terminal nueva con los dos `source` cargados:
 
```bash
ros2 run pfg_fanuc_control joint_state_opcua_bridge
```
 
Deberías ver el mensaje `Puente ROS2-OPC-UA listo, sirviendo en
opc.tcp://0.0.0.0:4840/fanuc_bridge/`. A partir de aquí, sigue el
apartado 8.4 de la memoria para verificarlo con UaExpert antes de
conectar RoboDK.
 
### 6.7. RoboDK: conectar como cliente OPC-UA y montar el gemelo digital
 
**Requisito previo**: deja el puente OPC-UA (apartado 6.6) y MoveIt2
corriendo en sus terminales de WSL — los necesitas activos durante
todo este apartado.
 
**Paso 1 — Añadir el robot CRX-10iA/L a la escena.** No hace falta
importar ningún archivo: el CRX-10iA/L ya está en la librería online
de RoboDK.
- En RoboDK, ve al panel de librería (o File → Add → Add robot) y
  busca "Fanuc CRX-10iA/L".
- Añádelo a una escena nueva o existente. Debería aparecer el modelo
  3D del robot en la vista.
**Paso 2 — Habilitar el complemento OPC-UA**:
- Menú Tools → Add-ins.
- Doble clic en "OPC-UA" para activarlo.
- Deberías ver una barra de herramientas nueva y una entrada de menú
  "OPC-UA".
**Paso 3 — Añadir el cliente OPC-UA:**
- Desde el menú/barra "OPC-UA", busca la opción "Add Client".
- Introduce la URL del endpoint de tu puente:
  `opc.tcp://localhost:4840/fanuc_bridge/`
- Pulsa "Connect".
- Si todo va bien, debería aparecer un mensaje tipo "Server variables
  retrieved" (variables del servidor recuperadas).
**Paso 4 — Comprobar los valores recibidos:**
- Clic derecho sobre el nombre de la estación (en el árbol de la
  izquierda) → "Station Parameters".
- Deberías ver ahí las 6 variables (J1...J6) con sus valores actuales,
  cambiando en tiempo real si mueves el robot en RViz/MoveIt2.
 
