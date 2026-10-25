# Patient Administrator: versión con Zustand y React Hook Form

[Volver al README principal](../README.md) · [Ver la versión con estado local](../patients/README.md)

Segunda implementación del administrador de pacientes de una veterinaria. Está escrita en TypeScript y reemplaza el estado local de la primera versión por un store global de Zustand, los inputs controlados por React Hook Form y los mensajes en pantalla por notificaciones de React Toastify.

## Demo

[Probar el sitio en vivo](https://patient-administrator-zustand.netlify.app/)

El deploy de Netlify corresponde a esta versión.

## Tecnologías

| Tecnología | Versión declarada | Uso |
| --- | --- | --- |
| React y React DOM | `^19.3.1` | Interfaz |
| TypeScript | `~6.0.3` | Tipado estático |
| Vite | `^8.3.4` | Servidor de desarrollo y build |
| `@vitejs/plugin-react` | `^6.1.2` | Soporte de React en Vite |
| Tailwind CSS y `@tailwindcss/vite` | `^4.3.3` | Estilos |
| Zustand | `^5.0.16` | Estado global, persistencia y DevTools |
| React Hook Form | `^7.90.0` | Formulario y validaciones |
| React Toastify | `^11.1.0` | Notificaciones |
| uuid | `^14.0.3` | Identificadores de pacientes |
| ESLint, `typescript-eslint`, `eslint-plugin-react-hooks`, `eslint-plugin-react-refresh` | ver `package.json` | Lint |

## Funcionalidades

- Registro de pacientes con nombre, propietario, email, fecha de alta y síntomas.
- Listado de pacientes, con un mensaje cuando todavía no hay ninguno.
- Edición de un paciente: sus datos se cargan en el formulario y al guardar se actualiza el registro.
- Eliminación de pacientes.
- Mientras un paciente está en edición, los botones Editar y Eliminar de su tarjeta quedan deshabilitados.
- Validación por campo con mensajes de error debajo de cada input.
- Notificaciones al registrar, actualizar y eliminar.
- Persistencia en `localStorage` bajo la clave `patient-storage`.
- Inspección del store con Redux DevTools.

## Estructura

```
patients-zustand/
├── index.html
├── package.json
├── eslint.config.js
├── tsconfig.json
├── tsconfig.app.json
├── tsconfig.node.json
├── vite.config.ts
└── src/
    ├── main.tsx
    ├── App.tsx
    ├── store.ts
    ├── index.css
    ├── types/
    │   └── index.ts
    └── components/
        ├── Error.tsx
        ├── PatientDetailItem.tsx
        ├── PatientDetails.tsx
        ├── PatientForm.tsx
        └── PatientsList.tsx
```

| Archivo | Responsabilidad |
| --- | --- |
| `store.ts` | Store de Zustand: estado, acciones, persistencia y DevTools |
| `types/index.ts` | Tipos `Patient` y `DraftPatient` |
| `App.tsx` | Título, composición de formulario y listado, y `ToastContainer` |
| `PatientForm.tsx` | Formulario con React Hook Form, validación y alta o edición |
| `PatientsList.tsx` | Lee `patients` del store y renderiza la lista o el estado vacío |
| `PatientDetails.tsx` | Tarjeta de un paciente con los botones Editar y Eliminar, que se deshabilitan si ese paciente está en edición |
| `PatientDetailItem.tsx` | Fila de etiqueta y valor reutilizable |
| `Error.tsx` | Mensaje de error de un campo |

## Modelo de datos

```ts
export type Patient = {
    id: string
    name: string
    caretaker: string
    email: string
    date: Date
    symptoms: string
}

export type DraftPatient = Omit<Patient, 'id'>
```

`DraftPatient` representa los datos del formulario, antes de que se les asigne un `id`.

## Estado global con Zustand

El store se define en `src/store.ts` con `create` y se expone como el hook `usePatientStore`.

**Estado**

- `patients`: lista de pacientes.
- `activeId`: `id` del paciente que se está editando, o cadena vacía si no hay edición.

**Acciones**

| Acción | Efecto |
| --- | --- |
| `addPatient(data)` | Crea el paciente con un `id` generado por `uuid` (v4) y lo agrega a `patients` |
| `deletePatient(id)` | Quita de `patients` al paciente con ese `id` |
| `getPatientById(id)` | Guarda el `id` en `activeId`; el formulario usa ese valor para pasar a modo edición |
| `updatePatient(data)` | Reemplaza el paciente cuyo `id` es `activeId` y vuelve `activeId` a `''` |

**Middleware**

El store se compone como `devtools(persist(...))`:

- `persist` guarda el estado en `localStorage` con la clave `patient-storage`, usando `createJSONStorage`.
- `devtools` permite inspeccionar el store y sus acciones con Redux DevTools.

**Selectores**

Los componentes se suscriben a una porción del estado, con un selector por cada valor o acción:

```ts
const patients = usePatientStore(state => state.patients)
const deletePatient = usePatientStore(state => state.deletePatient)
```

El store se consume en `PatientForm` (`patients`, `activeId`, `addPatient`, `updatePatient`), `PatientsList` (`patients`) y `PatientDetails` (`activeId`, `deletePatient`, `getPatientById`). Ningún componente recibe datos del store por props, salvo `PatientDetails`, que recibe el `patient` a renderizar.

## Formularios con React Hook Form

`PatientForm` usa `useForm<DraftPatient>()` y aprovecha estas partes de la API:

- `register`: conecta cada input con el formulario y define sus reglas de validación.
- `handleSubmit`: ejecuta la función de envío solo si la validación pasa.
- `formState.errors`: expone los errores de cada campo, que se muestran con el componente `Error`.
- `setValue`: carga los datos del paciente seleccionado en los campos al editar.
- `reset`: limpia el formulario tras guardar.

El `<form>` define `noValidate`, por lo que la validación del navegador queda desactivada y los mensajes provienen de React Hook Form.

### Validaciones implementadas

| Campo | Regla | Mensaje |
| --- | --- | --- |
| `name` | Obligatorio | El Nombre del Paciente es Obligatorio |
| `caretaker` | Obligatorio | El Propetario es obligatorio |
| `email` | Obligatorio y coincidencia con una expresión regular de email | El Email es Obligatorio / Email No Válido |
| `date` | Obligatorio | La Fecha de Alta es Obligatorio |
| `symptoms` | Obligatorio | Los Sintomas son Obligatorios |

## Notificaciones con React Toastify

`ToastContainer` se monta una sola vez en `App.tsx`, y las notificaciones se disparan con `toast` desde los componentes que ejecutan la acción:

| Evento | Notificación | Ubicación |
| --- | --- | --- |
| Alta de paciente | `toast.success('Paciente Registrado Correctamente')` | `PatientForm.tsx` |
| Actualización de paciente | `toast.success('Paciente Actualizado Correctamente')` | `PatientForm.tsx` |
| Eliminación de paciente | `toast.error('Paciente Eliminado')` | `PatientDetails.tsx` |

## Flujo de funcionamiento

1. Al cargar, `persist` rehidrata `patients` desde `localStorage`.
2. `PatientsList` renderiza un `PatientDetails` por cada paciente, o un mensaje si la lista está vacía.
3. El usuario completa el formulario. Al enviar, React Hook Form valida y, si hay errores, los muestra debajo de cada campo.
4. Si la validación pasa, `PatientForm` decide según `activeId`:
   - Sin `activeId`: llama a `addPatient` y muestra el toast de registro.
   - Con `activeId`: llama a `updatePatient` y muestra el toast de actualización.
5. Luego se llama a `reset()` para limpiar el formulario.
6. Al pulsar Editar, `PatientDetails` llama a `getPatientById(patient.id)`. El `useEffect` de `PatientForm`, que depende de `activeId`, busca ese paciente y completa los campos con `setValue`. Mientras dura la edición, `PatientDetails` compara `activeId` con el `id` de su paciente y deshabilita los botones Editar y Eliminar de esa tarjeta; se reactivan cuando `updatePatient` vuelve `activeId` a `''`.
7. Al pulsar Eliminar en un paciente que no está en edición, `PatientDetails` llama a `deletePatient` y muestra el toast de eliminación. No se pide confirmación previa.

## Instalación y ejecución

Requiere Node.js 20.19 o superior, o 22.12 o superior, según el motor declarado por Vite 8. ESLint 10 requiere 20.19, 22.13 o superior. Los comandos se ejecutan desde la carpeta `patients-zustand/`.

```bash
cd patients-zustand
npm install
npm run dev
```

### Scripts disponibles

| Comando | Descripción |
| --- | --- |
| `npm run dev` | Inicia el servidor de desarrollo de Vite |
| `npm run build` | Verifica los tipos con `tsc -b` y genera el build de producción en `dist/` |
| `npm run lint` | Ejecuta ESLint sobre el proyecto |
| `npm run preview` | Sirve localmente el build de producción |

No hay tests automatizados configurados.

## Decisiones técnicas y aprendizajes

- **Store único y acciones centralizadas.** Alta, edición y eliminación se definen en un solo archivo, en lugar de repartirse entre `App` y los componentes.
- **Selectores individuales.** Cada componente toma solo lo que necesita del store.
- **Edición mediante `activeId`.** El store guarda qué paciente se edita, y el formulario reacciona a ese valor para rellenarse.
- **Bloqueo durante la edición.** Deshabilitar los botones del paciente activo evita eliminarlo mientras el formulario lo tiene cargado. Se resuelve en la UI con `disabled` y las variantes `enabled:` y `disabled:` de Tailwind, sin agregar estado nuevo al store.
- **Persistencia declarativa.** El middleware `persist` reemplaza el `useEffect` manual de la primera versión.
- **Formulario declarativo.** Las reglas de validación se declaran junto a cada input, y los errores por campo vienen de `formState.errors`.
- **Tipado del dominio.** `Patient` y `DraftPatient` documentan la forma de los datos y los comparten el store y el formulario.
- **Componentes de presentación reutilizables.** `PatientDetailItem` y `Error` evitan repetir markup.

## Limitaciones

- Los datos solo existen en el `localStorage` del navegador: no hay backend ni sincronización entre dispositivos.
- La eliminación no pide confirmación.
- No hay una acción para cancelar una edición. Además, el botón Editar de las demás tarjetas sigue habilitado y reemplaza los datos cargados en el formulario sin avisar.
- No hay tests automatizados.
- `npm run lint` informa una advertencia de `react-hooks/exhaustive-deps` en el `useEffect` de `PatientForm.tsx`, cuyo arreglo de dependencias contiene solo `activeId`. No es un error y no bloquea el build.
