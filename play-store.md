# Ficha y formularios de Google Play

Respuestas listas para transcribir a la Consola, más el texto de la ficha.
Todo esto describe la aplicación **tal como está hoy**; cuando entre el cobro
por Google Play hay que volver sobre las secciones marcadas.

> Las categorías y preguntas exactas del formulario de seguridad de los datos
> **cambian cada cierto tiempo**. Lo de abajo es el contenido correcto; si una
> pregunta no coincide literalmente con lo que ves en la Consola, guíate por
> el sentido y no por la redacción.

---

## 1. Borrado de cuenta: a medio camino

Google exige que toda aplicación que permita **crear una cuenta** ofrezca
también una forma de **pedir que se borre**, y además un enlace web que se
pueda abrir sin instalar la app.

**Hecho:** el lado del servidor, en la migración `20261109010000`. Las
funciones `delete_my_account()`, `transfer_farm_ownership()` y
`farms_blocking_account_deletion()` implementan la regla: una finca
compartida nunca se destruye por decisión de una sola persona, así que el
dueño tiene que transferirla antes de poder borrarse. Esa migración también
corrige dos defectos que hacían que borrar un usuario no borrara sus datos
—o fallara con un error— y están explicados en su encabezado.

**Hecho también:** la pantalla en la app (menú **⋮ → Borrar mi cuenta**, en
las pantallas principal, de bienvenida y de plan vencido), y la página
`docs/borrar-cuenta.md` que Google exige aparte.

**Falta:** publicar esa página en una URL accesible sin instalar la app ni
iniciar sesión, y pegarla en el campo **URL de eliminación de datos** de la
Consola. Va junto a la política de privacidad.

**Probado** el 9 de noviembre de 2026 contra el proyecto real, con una
cuenta desechable que era dueña de una finca compartida con un mayordomo.
Las dos ramas se comportaron como se esperaba:

- Siendo dueño con otro miembro, el borrado **se negó** con
  `OWNER_MUST_TRANSFER: Loma alta`.
- Tras transferir la finca al mayordomo, el borrado **sí procedió**: el
  usuario desapareció de `auth.users`, la finca siguió viva, y el mayordomo
  quedó con rol `owner`.

Quedó despejada la duda que más preocupaba: Supabase **sí** le permite a una
función `security definer` borrar de `auth.users`. No hace falta mover ese
paso a una función de servidor con la *service role key*.

Como no hay pantalla todavía, la prueba se hizo desde el SQL Editor
suplantando al usuario con
`set_config('request.jwt.claims', json_build_object('sub', '<uuid>')::text, true)`
antes de llamar a la función, porque `auth.uid()` es nulo en el editor.

---

## 2. Ficha de la tienda

**Nombre de la aplicación** (máx. 30 caracteres)

```
Finca Ganado
```

**Descripción corta** (máx. 80 caracteres)

```
Lleve el control de su ganado, sus lotes y sus insumos, incluso sin señal.
```

**Descripción completa** (máx. 4000 caracteres)

```
Finca Ganado es una aplicación para llevar el día a día de una finca
ganadera desde el celular, hecha para usarse en el potrero y no en una
oficina.

TRABAJA SIN SEÑAL
Todo lo que registre se guarda de una vez en su teléfono, tenga o no
cobertura. Cuando vuelve la señal, se sincroniza solo. No pierde un
registro por estar lejos de la antena.

EL DUEÑO Y EL MAYORDOMO, EN LA MISMA PÁGINA
Invite a su mayordomo con un código y ambos verán la misma información en
sus propios celulares, actualizada sola.

LO QUE PUEDE LLEVAR
• Animales: número, raza, sexo, fechas y de quién es hijo cada uno.
• Vacunas, con el aviso de la próxima dosis.
• Tratamientos, con los días de retiro de carne y de leche calculados.
• Partos y montas, con la fecha estimada del próximo parto.
• Compras de insumos, con abonos parciales y lo que queda debiendo.
• Lotes y fumigaciones: qué se aplicó, cuánto se gastó de la compra y desde
  cuándo se puede volver a entrar.

LE AVISA A TIEMPO
La aplicación le recuerda las vacunas que vienen, cuándo se cumple el
retiro de un tratamiento y cuándo vuelve a estar libre un lote fumigado.

DICTE EN VEZ DE ESCRIBIR
Con las manos ocupadas o sucias, toque el micrófono y dicte la nota. La
aplicación no graba su voz: solo escribe lo que usted dice.
```

**Categoría sugerida:** Productividad, o Negocios.

**Justificación del permiso de micrófono** (si la Consola la pide)

```
El micrófono se usa únicamente para una función de dictado por voz en los
campos de texto libre (notas, nombres de productos), pensada para usuarios
que trabajan en el campo con las manos ocupadas. Se activa solo mientras el
usuario mantiene pulsado el botón de micrófono y se apaga al soltarlo.

La aplicación no graba ni almacena audio, y no lo transmite a servidores
propios: usa el servicio de reconocimiento de voz del sistema operativo y
recibe de vuelta únicamente el texto transcrito. No hay escucha en segundo
plano ni ninguna otra función que use el micrófono.
```

---

## 3. Formulario de seguridad de los datos

### ¿La app recoge o comparte datos de usuario? **Sí**

### Tipos de datos

| Tipo | ¿Se recoge? | ¿Se comparte? | ¿Obligatorio? | Para qué |
|---|---|---|---|---|
| Información personal → Dirección de correo | Sí | No | Obligatorio | Gestión de la cuenta |
| Información personal → Identificadores de usuario | Sí | No | Obligatorio | Gestión de la cuenta |

**Todo lo demás: no.** En particular, responde **No** a ubicación,
información financiera, salud, mensajes, fotos y videos, **archivos de
audio**, archivos y documentos, calendario, contactos, actividad de
navegación, rendimiento de la app, y otros identificadores.

> **Sobre el audio, que es donde más se equivoca la gente:** la app declara
> el permiso `RECORD_AUDIO`, pero **no recoge audio**. El reconocimiento lo
> hace el servicio del sistema y la app solo recibe texto. Declarar que sí
> se recoge audio sería incorrecto y además dispararía revisiones que no
> corresponden.

> **Sobre los registros de la finca** (animales, vacunas, gastos): no
> encajan en ninguna de las categorías del formulario, que están pensadas
> para datos sobre la persona y no para el contenido que la persona crea.
> Revisa la lista vigente por si apareció una categoría aplicable; si no la
> hay, no se declaran.

### ¿Se comparten con terceros?

**No.** Supabase y PowerSync son proveedores que tratan los datos por cuenta
nuestra para prestar el servicio, lo que Google no considera "compartir".

### Prácticas de seguridad

| Pregunta | Respuesta |
|---|---|
| ¿Los datos van cifrados en tránsito? | **Sí** (todo va por HTTPS) |
| ¿El usuario puede pedir que se borren sus datos? | **Sí** — requiere la página del punto 1 |
| ¿La app se sometió a una revisión de seguridad independiente? | No |
| ¿Hay un mecanismo para que el usuario pida una copia de sus datos? | Sí, por solicitud al correo de soporte |

### Cuando entre el cobro por Google Play

Habrá que volver aquí y declarar **Información financiera → Historial de
compras**, porque el estado de la suscripción se guarda del lado del
servidor.

---

## 4. Otros campos de la Consola

- **URL de la política de privacidad:** la de `docs/politica-de-privacidad.md`
  publicada en algún sitio accesible sin iniciar sesión. GitHub Pages sirve y
  es gratis.
- **Correo de contacto:** es obligatorio y queda **público** en la ficha.
- **Clasificación de contenido:** cuestionario aparte. Esta app no tiene
  contenido sensible; se responde que no a todo.
- **App de anuncios:** No.
- **Público objetivo:** mayores de 18. No está dirigida a menores.
