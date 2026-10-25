# Patient Administrator: versión con estado local

[Volver al README principal](../README.md) · [Ver la versión con Zustand](../patients-zustand/README.md)

Primera implementación del administrador de pacientes de una veterinaria. Está escrita en JavaScript (JSX) y gestiona todo el estado con hooks básicos de React, sin librerías de estado ni de formularios.

## Tecnologías

| Tecnología | Versión declarada | Uso |
| --- | --- | --- |
| React y React DOM | `^18.2.0` | Interfaz y estado local |
| Vite | `^4.2.0` | Servidor de desarrollo y build |
| `@vitejs/plugin-react` | `^3.1.0` | Soporte de React en Vite |
| Tailwind CSS | `^3.3.1` | Estilos, con PostCSS y Autoprefixer |

No tiene dependencias de runtime fuera de React.

## Funcionalidades

- Registro de pacientes con nombre de la mascota, propietario, email, fecha de alta y síntomas.
- Listado de pacientes, con un mensaje cuando todavía no hay ninguno.
- Edición de un paciente: sus datos se cargan en el formulario y el botón cambia de "Agregar Paciente" a "Editar Paciente".
- Eliminación con confirmación mediante `confirm()`.
- Persistencia en `localStorage` bajo la clave `pacientes`.
- Diseño responsive: formulario y listado se muestran lado a lado desde el breakpoint `md`.

## Estructura

```
patients/
├── index.html
├── package.json
├── tailwind.config.js
├── postcss.config.js
├── vite.config.js
└── src/
    ├── main.jsx
    ├── App.jsx
    ├── index.css
    └── components/
        ├── Error.jsx
        ├── Formulario.jsx
        ├── Header.jsx
        ├── ListadoPaciente.jsx
        └── Paciente.jsx
```

| Componente | Responsabilidad |
| --- | --- |
| `App` | Mantiene el estado `pacientes` y `paciente`, persiste en `localStorage` y define `eliminarPaciente` |
| `Header` | Título de la aplicación |
| `Formulario` | Campos, validación, alta y edición |
| `ListadoPaciente` | Renderiza la lista o el mensaje de estado vacío |
| `Paciente` | Muestra los datos de un paciente y sus botones de editar y eliminar |
| `Error` | Contenedor de estilo para el mensaje de error |

## Gestión del estado

El estado compartido está elevado a `App`:

- `pacientes`: arreglo con todos los pacientes. Se inicializa leyendo `localStorage` con `JSON.parse`, y si no hay datos guardados, con un arreglo vacío (`?? []`).
- `paciente`: objeto del paciente que se está editando. Es `{}` cuando no hay edición.

`App` pasa por props `pacientes`, `setPacientes`, `paciente` y `setPaciente` a `Formulario`, y `pacientes`, `setPaciente` y `eliminarPaciente` a `ListadoPaciente`, que a su vez reenvía `setPaciente` y `eliminarPaciente` a cada `Paciente`.

`Formulario` mantiene además estado propio: un `useState` por campo (`nombre`, `propietario`, `email`, `fecha`, `sintomas`) y otro para el indicador de error.

## Hooks utilizados

- `useState`: lista de pacientes, paciente en edición, valores de cada campo del formulario y bandera de error.
- `useEffect` en `App`: con dependencia `[pacientes]`, guarda la lista en `localStorage` cada vez que cambia.
- `useEffect` en `Formulario`: con dependencia `[paciente]`, copia los datos del paciente seleccionado a los campos del formulario cuando se pulsa Editar.

## Flujo de funcionamiento

1. Al cargar, `App` lee los pacientes guardados en `localStorage`.
2. El usuario completa el formulario. Cada cambio actualiza el `useState` del campo (input controlado).
3. Al enviar, `handleSubmit` valida el formulario:
   - Si algún campo está vacío, activa el estado de error y muestra el mensaje "Todos los campos son obligatorios".
   - Si es válido, arma el objeto del paciente.
4. Si `paciente.id` existe, se está editando: se reemplaza el paciente con ese `id` mediante `map` y se vuelve a `{}` el paciente en edición. Si no existe, se genera un `id` con `generarId()` y se agrega el paciente al final de la lista.
5. Se reinician los campos del formulario.
6. El cambio en `pacientes` dispara el `useEffect` de `App`, que actualiza `localStorage`.
7. Al eliminar, `Paciente` pide confirmación y, si se acepta, `App` filtra la lista por `id`.

## Formularios y validación

Los formularios se manejan manualmente con inputs controlados (`value` y `onChange`). La validación consiste en verificar que ninguno de los cinco campos esté vacío; el resultado es un único mensaje general, sin detalle por campo. El campo email usa `type="email"`, por lo que el navegador aplica además su validación nativa de formato al enviar. El formulario no define `noValidate`, y no hay validaciones propias adicionales sobre el contenido.

## Identificadores

`generarId()` combina `Date.now()` y `Math.random()`, ambos convertidos a base 36. Genera un identificador suficiente para las claves de React y para ubicar pacientes dentro de esta aplicación.

## Instalación y ejecución

Requiere Node.js 14.18 o superior (excepto la serie 15), o 16 o superior, según el motor declarado por Vite 4. Los comandos se ejecutan desde la carpeta `patients/`.

```bash
cd patients
npm install
npm run dev
```

### Scripts disponibles

| Comando | Descripción |
| --- | --- |
| `npm run dev` | Inicia el servidor de desarrollo de Vite |
| `npm run build` | Genera el build de producción en `dist/` |
| `npm run preview` | Sirve localmente el build de producción |

Esta versión no define scripts de lint ni de tests.

## Aspectos técnicos y aprendizajes

- Elevación de estado (`lifting state up`): `App` es la fuente de verdad y los hijos reciben datos y funciones por props.
- Inputs controlados: el valor de cada campo vive en React y se sincroniza con `onChange`.
- Sincronización con `useEffect`: dos efectos con responsabilidades distintas, persistir datos y rellenar el formulario al editar.
- Un mismo formulario sirve para alta y edición, y se distingue por la presencia de `paciente.id`.
- Actualizaciones inmutables del estado con `map`, `filter` y el operador spread.
- Renderizado condicional con `&&` y operadores ternarios.

## Limitaciones

- Los datos solo existen en el `localStorage` del navegador: no hay backend ni sincronización entre dispositivos.
- La validación se limita a campos vacíos.
- No hay tests automatizados.
