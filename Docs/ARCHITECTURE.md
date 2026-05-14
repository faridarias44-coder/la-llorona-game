# Arquitectura Técnica del Juego

## 🏗️ Diagrama General

```
┌─────────────────────────────────────────────────┐
│           LA LLORONA - JUEGO DE TERROR          │
├─────────────────────────────────────────────────┤
│                                                 │
│  ┌────────────┐    ┌──────────┐    ┌────────┐  │
│  │   PLAYER   │    │  ENEMY   │    │  TIME  │  │
│  │ Controller │◄──►│ AI Logic │◄──►│Manager │  │
│  └────────────┘    └──────────┘    └────────┘  │
│       │                   │              │      │
│       ▼                   ▼              ▼      │
│  ┌────────────┐    ┌──────────┐    ┌────────┐  │
│  │  CAMERA    │    │   DOGS   │    │AMBIENT │  │
│  │ MANAGEMENT │    │  WARNING │    │MANAGER │  │
│  └────────────┘    └──────────┘    └────────┘  │
│                                                 │
│  ┌─────────────────────────────────────────┐   │
│  │         GAME MANAGER (MAESTRO)          │   │
│  │  - Controla flujo general del juego     │   │
│  │  - Detecta condiciones de victoria      │   │
│  │  - Pausa/Resume                         │   │
│  └─────────────────────────────────────────┘   │
│                                                 │
│  ┌─────────────────────────────────────────┐   │
│  │         AUDIO MANAGER (SINGLETON)       │   │
│  │  - Música de fondo                      │   │
│  │  - Efectos de sonido                    │   │
│  │  - Sonidos ambientes                    │   │
│  └─────────────────────────────────────────┘   │
│                                                 │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   │
│  │   HUD    │   │   MENUS  │   │   INPUT  │   │
│  │ MANAGER  │   │ MANAGER  │   │ HANDLER  │   │
│  └──────────┘   └──────────┘   └──────────┘   │
│                                                 │
└─────────────────────────────────────────────────┘
```

---

## 📚 Estructura de Clases

### Core Systems

#### **GameManager** (Singleton)
```csharp
Responsabilidades:
- Flujo del juego (inicio, juego, fin)
- Detección de victoria (llegar a casa)
- Detección de derrota (atrapado)
- Control de pausa
- Transición de escenas

Métodos Públicos:
- WinGame()
- LoseGame()
- IsGameOver()
- HasPlayerWon()
```

#### **TimeManager** (Evento)
```csharp
Responsabilidades:
- Simulación de tiempo real
- Ciclo día/noche (22:00 a 6:00)
- Evento OnTimeChanged
- Control de iluminación

Métodos Públicos:
- GetCurrentTime()
- IsNight()
```

#### **AudioManager** (Singleton)
```csharp
Responsabilidades:
- Reproducción de música
- Reproducción de SFX
- Carga de audio en batch
- Control de volumen

Métodos Públicos:
- PlayMusic(string)
- PlaySFX(string)
- GetSoundEffect(string)
- StopMusic()
```

---

### Player Systems

#### **PlayerController**
```csharp
Responsabilidades:
- Input del jugador (WASD)
- Física de movimiento
- Velocidad (caminar/correr)
- Saltos
- Detección de colisiones

Componentes Requeridos:
- Rigidbody
- Capsule Collider
```

#### **CameraManager**
```csharp
Responsabilidades:
- Manejo de dos cámaras (1ª/3ª persona)
- Rotación con mouse
- Interpolación de cámara
- Límites de rotación

Input:
- Mouse X/Y (rotación)
- C (cambiar vista)
```

#### **PlayerAnimator**
```csharp
Responsabilidades:
- Control de animaciones
- Estados (parado, caminando, corriendo)
- Sincronización con movimiento

Parámetros Animator:
- isMoving (bool)
- isRunning (bool)
- isGrounded (bool)
```

---

### Enemy Systems

#### **LloranaAI**
```csharp
Responsabilidades:
- Patrulla automática
- Detección del jugador (50m)
- Persecución inteligente
- Animaciones y sonidos
- Ataque cuando está cerca

Estados:
- Patrullando
- Persiguiendo
- Atacando

Componentes Requeridos:
- NavMeshAgent
- Animator
- AudioSource
```

#### **DogWarningSystem**
```csharp
Responsabilidades:
- Monitorear posiciones de perros
- Detectar cuándo La Llorona persigue
- Disponer ladridos cuando hay peligro
- Radio de detección dinámico
```

---

### Environment Systems

#### **EnvironmentManager**
```csharp
Responsabilidades:
- Cambiar skybox (día/noche)
- Ajustar niebla
- Control de luces ambientales
- Efectos de iluminación

Listener:
- OnTimeChanged (TimeManager)
```

#### **HidingSpot**
```csharp
Responsabilidades:
- Detección de proximidad
- Mechánica de esconderse
- Conteo de tiempo de seguridad
- Efectos visuales al esconderse
```

---

### UI Systems

#### **HUDManager** (Singleton)
```csharp
Responsabilidades:
- Mostrar hora en tiempo real
- Panel de Game Over
- Botones (Reiniciar, Menú Principal)
- Actualización de UI

Canvas Components:
- TextMeshPro (Hora)
- Panel (Game Over)
- Botones interactuables
```

#### **MainMenuUI**
```csharp
Responsabilidades:
- Menú principal
- Botón Play (carga MainGame)
- Botón Settings
- Botón Exit
```

---

## 🔄 Flujo de Ejecución

### Inicio del Juego
```
1. Unity carga MainMenu.unity
2. MainMenuUI.Start()
   └─ AudioManager.PlayMusic("menu_music")
3. Usuario presiona Play
4. Carga MainGame.unity
```

### Durante el Juego
```
Loop principal:
┌──────────────────────────────────┐
│ Input (PlayerController.Update)  │
├──────────────────────────────────┤
│ Physics (FixedUpdate)            │
├──────────────────────────────────┤
│ Lógica (LloranaAI.Update)        │
├──────────────────────────────────┤
│ Tiempo (TimeManager.Update)      │
├──────────────────────────────────┤
│ UI (HUDManager.UpdateTimeDisplay)│
├──────────────────────────────────┤
│ Verificar Victoria/Derrota       │
└──────────────────────────────────┘
```

### Condiciones de Victoria
```
IF (distancia_jugador_a_casa < 5 metros)
    GameManager.WinGame()
    
OR

IF (hora_juego >= 6 AM)
    GameManager.WinGame()
```

### Condiciones de Derrota
```
IF (distancia_llorona_a_jugador < 3 metros)
    GameManager.LoseGame()
```

---

## 📡 Sistema de Comunicación

### Events (Observadores)
```
TimeManager.OnTimeChanged
    ↓
    - HUDManager (actualiza hora)
    - EnvironmentManager (cambia lighting)
    - PlayerAnimator (sincroniza animaciones)
```

### Tags & Layers
```
PlayerController.FindGameObjectWithTag("Player")
LloranaAI.FindGameObjectWithTag("Enemy")
GameManager.FindGameObjectWithTag("SafeHouse")
```

### Singletons
```
AudioManager.Instance.PlaySFX("jump")
GameManager.Instance.WinGame()
HUDManager.Instance.ShowGameOverScreen(true)
```

---

## 🎯 Performance Considerations

### Optimizaciones Implementadas
- NavMeshAgent en vez de raycasting constante
- AudioManager con precarga en batch
- Singleton pattern para evitar instancias múltiples
- Object pooling potencial (próxima versión)

### Próximas Optimizaciones
- LOD (Level of Detail) para modelos lejanos
- Shader compiling en background
- Frustum culling para objetos fuera de pantalla

---

## 🧪 Testing

### Casos de Prueba Recomendados
1. Victoria por llegar a casa
2. Victoria por sobrevivir hasta 6 AM
3. Derrota por ser atrapado
4. Cambio de cámara funcional
5. Perros ladran cuando La Llorona persigue
6. Audio carga correctamente

---

## 📊 Diagrama de Estados

```
GAME STATE MACHINE:

┌─────────┐
│  START  │
└────┬────┘
     │ (carga MainGame)
     ▼
┌─────────────────┐
│  PLAYING (HUD)  │◄──┐
└────┬─────┬──────┘   │
     │     │          │
     │     └─── PAUSE (futuro)
     │
     ├─ Jugador atrapado ──┐
     │                      ▼
     │              ┌──────────────┐
     │              │ GAME_OVER    │
     │              │ (Derrota)    │
     │              └──────────────┘
     │
     ├─ Llega a casa ──────┐
     │                      │
     ├─ Llega a 6 AM ───────┤
     │                      ▼
     │              ┌──────────────┐
     │              │ GAME_OVER    │
     └─────────────►│ (Victoria)   │
                    └──────────────┘
                           │
                           ▼
                    (Volver a menú)
```

---

## 🔐 Error Handling

Cada script verifica:
```csharp
if (player == null)
    Debug.LogError("Jugador no encontrado!");
    
if (llorona == null)
    Debug.LogWarning("Llorona no encontrada");
```

---

**Actualizado**: 2026-05-14  
**Versión**: 1.0 Prototipo Base
