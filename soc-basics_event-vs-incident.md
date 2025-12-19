# SOC Basics – Event vs Incident

> Notas personales de estudio. Conceptos entendidos desde la práctica y el aprendizaje como SOC junior.

## Definición (en mis palabras)

**Evento de seguridad (event):**  
Para mí, un evento es básicamente cualquier cosa que queda registrada en un sistema.  
Un login, un antivirus que avisa algo, una conexión rara, un cambio de permisos, etc.

Un evento **no es necesariamente algo malo**. Muchas veces es solo información que hay que mirar con contexto antes de sacar conclusiones.

**Incidente de seguridad (incident):**  
Un incidente es cuando uno o varios eventos ya muestran que **hay un problema real** o un intento claro de ataque.  
Acá ya no es solo “mirar”, sino **actuar o escalar**, porque puede haber impacto o riesgo real para la organización.

Dicho simple:
- Evento: *“pasó algo”*
- Incidente: *“pasó algo que importa y hay que responder”*


## Ejemplo realista (pensado como SOC junior)

**Evento:**  
El SIEM muestra muchos intentos fallidos de login para un usuario corporativo desde una IP extranjera.

En este punto, para mí todavía es solo un evento porque:
- puede ser un error del usuario,
- puede ser alguien viajando,
- o puede ser un ataque automático.

**Cuando se vuelve incidente:**  
Revisando un poco más veo que:
- después de muchos fallos, hay un **login exitoso**,
- el usuario nunca se conectó desde ese país,
- y aparecen acciones raras en la cuenta (por ejemplo reglas de reenvío de mail).

Ahí ya no lo veo como algo normal.  
Con esa evidencia, lo consideraría un **incidente de compromiso de cuenta** y lo escalaría.


## Cómo lo priorizaría como SOC junior

No intento ser perfecto ni adivinar todo. Mi foco sería **no dejar pasar algo grave**.

### 1. Primero: entender el contexto
Me preguntaría cosas simples:
- ¿Este usuario suele conectarse desde esa ubicación?
- ¿Hubo login exitoso o solo intentos fallidos?
- ¿Hay acciones posteriores sospechosas?
- ¿El usuario es crítico (IT, finanzas, admin)?

### 2. Prioridad aproximada
- **Baja:** eventos aislados, sin éxito, sin patrón claro.
- **Media:** muchos intentos, comportamiento raro, pero sin confirmación total.
- **Alta:** login exitoso extraño + acciones sospechosas claras.

Si dudo entre media y alta, **prefiero escalar**.

### 3. Qué haría como primer paso
Como junior, no intento resolver todo solo:
- Documento bien lo que veo (logs, horarios, IPs).
- Escalo con la información clara a L2 o al responsable.
- Sigo el playbook si existe, o pregunto antes de tocar nada crítico.

Mi idea es aprender a **detectar, priorizar y escalar bien**.

