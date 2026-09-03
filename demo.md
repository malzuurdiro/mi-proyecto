# PFG Prácticas docentes ROS2 + FANUC CRX-10iA/L

 
**Antes de empezar (WSL):** en cada terminal nueva para TB3:
 
```bash
source /opt/ros/humble/setup.bash
source ~/pfg_ws/install/setup.bash
```

---
 
## Parte 1 — Fundamentos de ROS2 con TurtleBot3 (capitulo 3)
 
**Terminal 1** — arrancar la simulación:
```bash
export TURTLEBOT3_MODEL=burger
ros2 launch turtlebot3_gazebo turtlebot3_world.launch.py
```
 
**Terminal 2** — visualización en rviz2:
```bash
rviz2
```
 
**Terminal 3** — nodo publisher (mueve el robot en círculo):
```bash
ros2 run pfg_tb3_examples publisher_node
```
 
**Terminal 4** — nodo subscriber (posición en tiempo real):
```bash
ros2 run pfg_tb3_examples subscriber_node
```
 
**Terminal 5** — servicio (reinicia la simulación):
```bash
ros2 run pfg_tb3_examples service_client_node
```
 
**Terminal 6 y 7** — action con feedback (servidor + cliente):
```bash
# Terminal 6 -> ejecuta tarea, espera goals, envía feedback, devuelve resultado
ros2 run pfg_tb3_examples rotate_action_server
# Terminal 7 -> pide acción, recibe feedback, recibe result, puede cancelar
ros2 run pfg_tb3_examples rotate_action_client
```
 
---
 
## Parte 2 — Robot FANUC CRX-10iA/L: visualización y planificación (Capítulo 5)
 
Cerrar las terminales de la Parte 1 antes de seguir (TurtleBot3 y el
FANUC no deben correr a la vez, comparten `/robot_description`).

**Antes de empezar (WSL):** en cada terminal nueva para FANUC: 
 
```bash
source /opt/ros/humble/setup.bash
source ~/fanuc_ws/install/setup.bash
```
 
**Terminal 1** — solo visualización con sliders:
```bash
ros2 launch fanuc_crx_description view_crx.launch.py robot_model:=crx10ia_l
```
 
**Terminal 1 (cerrar la anterior y relanza)** — RViz + MoveIt2 con
mock hardware, para planificar y ejecutar movimientos (dejar corriendo):
```bash
ros2 launch fanuc_moveit_config fanuc_moveit.launch.py robot_model:=crx10ia_l use_mock:=true
```
En rviz2: arrastra el marcador naranja del extremo del robot a una
pose nueva → pestaña "Planning" → **"Plan & Execute"**.
 
---
 
## Parte 3 — Control del robot (Capítulo 6)
 
Con el Terminal 1 de la Parte 2 (MoveIt2) todavía corriendo:
 
**Terminal 2** — trayectoria de 2 puntos (FollowJointTrajectory):
```bash
ros2 run pfg_fanuc_control send_trajectory_goal
```

---
 
## Parte 4 — Verificación OPC-UA con UaExpert (Capítulo 8)
 
Con MoveIt2 (Parte 2, Terminal 1) todavía corriendo:
 
**Terminal 4** — puente ROS2 → OPC-UA (déjalo corriendo el resto de la demo):
```bash
ros2 run pfg_fanuc_control joint_state_opcua_bridge
```
Debe salir: `Puente ROS2-OPC-UA listo, sirviendo en opc.tcp://0.0.0.0:4840/fanuc_bridge/`
 
**En Windows — UaExpert:** conectar al endpoint
`opc.tcp://localhost:4840/fanuc_bridge/` y muestra el árbol
`Objects → CRX10iAL → J1...J6` con los valores en tiempo real,
estado "Good".
 
---
 
## Parte 5 — Gemelo digital en RoboDK, sincronizado en tiempo real (Capítulo 8.5)
 
**Requisito previo:** Terminal 1 (MoveIt2) y Terminal 4 (puente OPC-UA)
de las partes anteriores, todavía corriendo en WSL.
 
**En Windows:**
 
1. Abrir RoboDK con la escena que tiene cargado el robot
   **"Fanuc CRX-10iA/L"** (nombre exacto, tal cual aparece en el árbol
   de la izquierda).
2. Abrir una terminal de Windows (PowerShell) y ejecutar:
```powershell
   C:\RoboDK\Python-Embedded\python.exe -u C:\Users\marta\opcua_to_robodk_bridge.py
```
 
3. Debe aparecer en la consola:
```
   RoboDK conectado. Robot encontrado: "Fanuc CRX-10iA/L"
   OPC-UA conectado correctamente.
   Nodos J1-J6 preparados.
   ============================
   GEMELO DIGITAL ACTIVO
   OPC-UA -> RoboDK
   Pulsa Ctrl+C para detener
```
 
4. En WSL, ejecuta de nuevo `send_trajectory_goal` (Parte 3)
   — **el robot de RoboDK debe moverse solo**,
   en tiempo real, siguiendo los mismos valores que ves en
   rviz2 y en UaExpert.
---
 
## Orden recomendado de apertura de terminales
 
Estado final con todo corriendo a la vez (Partes 2 a 5):
 
- **WSL Terminal 1**: MoveIt2 (`fanuc_moveit.launch.py`)
- **WSL Terminal 4**: puente OPC-UA (`joint_state_opcua_bridge`)
- **Windows**: RoboDK abierto + `opcua_to_robodk_bridge.py` corriendo
- **Windows**: UaExpert conectado
- **WSL Terminal 2 o 3**: para lanzar `send_trajectory_goal` /
  `send_position_goal` cuando quieras que todo se mueva a la vez
