# Inicio de la aplicación

La usuaria abre la aplicación.

Al iniciarse, el sistema verifica si existe una sesión activa.

---

## Verificación de sesión

- **Si la usuaria ya tiene sesión iniciada:**
  - La aplicación carga su perfil completo:
    - Contactos
    - Configuración
    - Modo sigiloso
    - Preferencias

- **Si no hay sesión iniciada:**
  - La aplicación puede funcionar completamente en modo invitado.
  - Algunas funciones avanzadas pueden estar limitadas:
    - Respaldo en la nube
    - Sincronización institucional
  - Iniciar sesión o crear una cuenta puede hacerse después dentro de la aplicación en la página de perfil.

---

## Registro o inicio de sesión (opcional)

Si la usuaria elige registrarse o iniciar sesión:

1. La aplicación solicita credenciales (correo y contraseña).
2. Envía los datos al servidor.
3. El servidor valida la información.
4. Si es correcto, devuelve un token de sesión.
5. La aplicación guarda la sesión y carga el perfil.

Si decide continuar como invitada:

- La aplicación crea una sesión local temporal.
- Los datos se almacenan únicamente en el dispositivo.

---

## Carga de configuración

Una vez definida la sesión (registrada o invitada), la aplicación carga:

- Modo sigiloso (ON/OFF)
- Contactos de emergencia (locales o sincronizados)
- Preferencias de grabación
- Activar o desactivar cancelación por clave
- Preferencia de cancelación

---

## Evaluación del modo de apertura

- Si el modo sigiloso está activado → muestra interfaz encubierta.
- Si está desactivado → muestra pantalla principal con niveles de criticidad.

---

# 🚦 Flujo de niveles de criticidad

---

## 🟢 Nivel 1 – Prevención (Iniciar trayecto)

### La usuaria:

- Selecciona destino
- Establece tiempo estimado
- Selecciona contactos

### El sistema:

- Obtiene ubicación
- Comparte ubicación en tiempo real
- Programa verificación de llegada

Cuando la usuaria llega, la app muestra botón de **“Confirmar llegada segura”**.

Al presionarlo:

- Se cambia estado a "finalizado"
- Se cancela verificación
- Se notifica a contactos seleccionados:  
  > “Llegó segura a su destino”

Si llega el tiempo de verificación y sigue activo:

- El sistema envía notificación a la usuaria:  
  > “¿Llegaste segura?”
- Da 1–2 minutos para responder.
- Si no responde:
  - Se envía alerta a contactos:  
    > “No confirmó llegada. Última ubicación: …”

⚠️ No se notifica a seguridad institucional, solo a contactos personales.

---

## 🟡 Nivel 2 – Riesgo

La usuaria selecciona **“Me siento en riesgo”**.

### El sistema:

- Obtiene ubicación
- Inicia grabación automática de audio
- Envía SMS a contactos
- Comparte ubicación en tiempo real
- Permite escalar a emergencia con un toque

⚠️ No notifica aún a seguridad institucional.

---

## 🔴 Nivel 3 – Emergencia real

La usuaria activa **“Ayuda ahora”** (desde pantalla principal o modo sigiloso).

### El sistema:

- Obtiene ubicación en tiempo real
- Inicia grabación automática de audio
- Envía alerta al servidor
- El servidor notifica a seguridad
- Envía SMS a contactos
- Continúa enviando actualizaciones periódicas

Si no hay internet:

- Guarda evidencia local
- Reintenta envío cuando haya conexión
- El SMS sigue funcionando si hay red celular

---

# 🔐 Cancelación de emergencia

La usuaria puede:

- Ingresar código de cancelación
- Usar biometría si olvidó el código
- Solicitar validación institucional (llama a seguridad de la institución)

El servidor actualiza el estado y notifica a seguridad.

---

# 🏁 Resolución institucional

Seguridad marca incidente como resuelto.

La aplicación:

- Detiene grabación
- Finaliza seguimiento
- Vuelve al estado normal


<br>
<br>
<br>
<br>

 # 📱 Diagrama secuencial
 (instalar la extension Markdown Preview Mermaid Support en Visual Studio)
 <br>

---

 ```mermaid
sequenceDiagram
    actor U as Usuaria
    participant A as App
    participant S as Servidor
    participant W as Worker Backend
    participant C as Contactos
    participant I as Seguridad Institucional

    %% =======================
    %% INICIO DE APLICACIÓN
    %% =======================

    U->>A: Abre aplicación
    A->>A: Verifica sesión local

    alt Sesión existente
        A->>S: Validar token
        S-->>A: Perfil + configuración
        A->>A: Cargar contactos y preferencias
    else No existe sesión
        A->>A: Crear sesión local temporal (modo invitado)
        Note right of A: Funciona completo\nLimitaciones: nube e institucional
    end

    %% =======================
    %% LOGIN OPCIONAL DESDE PERFIL
    %% =======================

    U->>A: Ir a Perfil
    U->>A: Elegir Iniciar sesión / Registrarse
    A->>S: Enviar credenciales
    S->>S: Validar información
    S-->>A: Token de sesión
    A->>A: Guardar token y cargar perfil

    %% =======================
    %% CARGA DE CONFIGURACIÓN
    %% =======================

    A->>A: Cargar modo sigiloso
    A->>A: Cargar contactos
    A->>A: Cargar preferencias grabación
    A->>A: Cargar método cancelación

    alt Modo sigiloso ON
        A-->>U: Mostrar interfaz encubierta
    else Modo sigiloso OFF
        A-->>U: Mostrar pantalla principal
    end

    %% =======================
    %% NIVEL 1 - PREVENCIÓN
    %% =======================

    U->>A: Iniciar trayecto
    A->>A: Obtener GPS
    A->>S: Crear sesión de trayecto
    S->>W: Programar verificación
    A->>C: Enviar link seguimiento

    loop Cada 30s
        A->>S: Actualizar ubicación
    end

    alt Confirma llegada
        U->>A: Confirmar llegada segura
        A->>S: Estado = finalizado
        S-->>C: Notificar llegada segura
    else No confirma llegada
        W->>S: Detectar tiempo vencido
        S->>A: Notificación ¿Llegaste segura?
        alt No responde
            S-->>C: Alerta + última ubicación
        end
    end

    %% =======================
    %% NIVEL 2 - RIESGO
    %% =======================

    U->>A: Me siento en riesgo
    A->>A: Obtener GPS
    A->>A: Iniciar grabación audio
    A->>C: Enviar SMS alerta
    A->>S: Registrar evento riesgo

    loop Monitoreo activo
        A->>S: Actualización ubicación
    end

    alt Escalar a emergencia
        U->>A: Escalar
        A->>S: Estado = emergencia
    end

    %% =======================
    %% NIVEL 3 - EMERGENCIA
    %% =======================

    U->>A: Ayuda ahora
    A->>A: Obtener GPS
    A->>A: Iniciar grabación audio
    A->>S: Enviar alerta emergencia

    S->>I: Notificar incidente
    S->>C: Enviar SMS emergencia

    loop Seguimiento activo
        A->>S: Actualización ubicación
        S->>I: Actualizar panel institucional
    end

    %% =======================
    %% MODO SIN INTERNET
    %% =======================

    alt Sin internet
        A->>A: Guardar evidencia local
        A->>A: Reintentar envío automático
        A->>C: SMS si hay red celular
    end

    %% =======================
    %% CANCELACIÓN
    %% =======================

    U->>A: Cancelar emergencia
    A->>S: Validar código / biometría

    alt Cancelación válida
        S->>I: Notificar cancelación
        S->>C: Notificar cancelación
        S-->>A: Estado = cancelado
    else Validación institucional
        S->>I: Solicitar verificación manual
    end

    %% =======================
    %% RESOLUCIÓN
    %% =======================

    I->>S: Marcar incidente resuelto
    S-->>A: Estado = resuelto
    A->>A: Detener grabación
    A->>A: Finalizar seguimiento
    A-->>U: Volver a estado normal

 ```

<br>

<br>

<br>

<br>
<br>
<br>
<br>

 # 📱 bosquejo de interfaz


![Pantalla principal](/interfaz/pantalla-principal.png)
