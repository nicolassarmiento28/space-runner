# SPACE RUNNER

Juego que combina endless runner y shooter espacial con temática de espacio profundo.

## 🎮 Cómo jugar

### Controles PC
- **W / S** o **Flechas ↑↓** - Mover entre carriles
- **ESPACIO** - Disparar (3 direcciones) / Iniciar juego

### Controles Táctil (Móvil/Tablet)
- **Tocar arriba** - Mover hacia arriba
- **Tocar abajo** - Mover hacia abajo  
- **Tocar centro** - Disparar

## 🎯 Objetivo

- Destruye enemigos:
  - Enemigos normales: +15 puntos
  - Enemigos shooters: +30 puntos
- Recolecta powerups: +40 puntos
  - **(+)** = Agrega 1 vida
  - **(S)** = Escudos
- **GANA** al llegar a **1500 puntos**
- **PIERDE** al perder todas sus **3 vidas**

## 🌌 Características del Juego

- Fondo espacial con estrellas, planetas y asteroides
- Música procedural generada en tiempo real (Web Audio API)
- Efectos de partículas al destruir enemigos
- Sistema de puntuación con record guardado (localStorage)
- Controles responsivos con cooldown para mejor experiencia
- Funciona en PC y dispositivos móviles

## 🛠️ Tecnologías Usadas

- **HTML5 Canvas** - Renderizado del juego
- **JavaScript ES6+** - Lógica del juego sin librerías
- **Web Audio API** - Música y efectos de sonido procedurales
- **localStorage** - Guardado de puntuación más alta

## 📁 Estructura del Proyecto

```
space-runner/
├── index.html    # Juego completo (todo en un archivo)
├── README.md     # Este archivo
└── AGENTS.md     # Guía para agentes de código
```

## 🚀 Cómo Ejecutar

### Opción 1: Abrir directamente
Simplemente abre el archivo `index.html` en cualquier navegador moderno.

### Opción 2: Servidor local
Si prefieres usar un servidor local:

```bash
# Python 3
python -m http.server 8000

# Node.js
npx http-server

# PHP
php -S localhost:8000
```

Luego abre `http://localhost:8000` en tu navegador.

### Opción 3: Despliegue en la nube
El juego está listo para desplegar en:
- **Vercel** - Arrastra la carpeta o conecta tu repositorio Git
- **Netlify** - Arrastra la carpeta o conecta tu repositorio
- **GitHub Pages** - Haz push a tu repositorio y habilita Pages

## 🔧 Solución de Problemas

### No se escucha la música
- El audio requiere interacción del usuario para funcionar
- Intenta presionar una tecla o tocar la pantalla antes de jugar

### Los controles no responden
- Verifica que el juego esté en estado "playing"
- Espera un momento entre pulsaciones (hay cooldown de movimiento)

### La pantalla está en blanco
- Verifica que tu navegador soporte HTML5 Canvas
- Prueba con Chrome, Firefox, Safari o Edge

## 📝 Notas

- Resolución base: 600x800 píxeles
- Se adapta automáticamente a diferentes tamaños de pantalla
- No requiere instalación ni dependencias
- Funciona sin conexión a internet (excepto por la fuente de letra)

---

Desarrollado por Nicolás Sarmiento