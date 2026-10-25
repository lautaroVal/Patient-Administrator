# Patient Administrator

Aplicación web para administrar pacientes de una veterinaria, desarrollada con React. Permite registrar pacientes, listarlos, editarlos y eliminarlos, con los datos persistidos en el navegador.

Este repositorio contiene dos implementaciones de la misma aplicación. Ambas resuelven la misma funcionalidad, pero con distintos enfoques para el estado y los formularios, lo que permite comparar las decisiones técnicas de cada una.

## Demo

- Versión con Zustand: [patient-administrator-zustand.netlify.app](https://patient-administrator-zustand.netlify.app/)

El deploy corresponde a la segunda versión (`patients-zustand/`). La primera versión tiene su propio enlace de demo en [su README](./patients/README.md). El repositorio no incluye archivos de configuración de Netlify; la configuración de build se define en la plataforma.

## 📸 Capturas

![Vista principal](/patients/public/Screenshot%202026-10-25%20030454.png)

## Versiones

### 1. React con estado local

Implementación inicial en JavaScript (JSX). El estado vive en `useState` dentro de los componentes: `App` guarda la lista de pacientes y el paciente en edición, y se los pasa a los hijos por props. Cada campo del formulario es un input controlado con su propio estado.

- Código: [`patients/`](./patients/)
- Documentación: [`patients/README.md`](./patients/README.md)

### 2. React con Zustand y React Hook Form ✅💎

Segunda implementación en TypeScript. El estado compartido se centraliza en un store de Zustand, los formularios se gestionan con React Hook Form y las notificaciones se muestran con React Toastify.

- Código: [`patients-zustand/`](./patients-zustand/)
- Documentación: [`patients-zustand/README.md`](./patients-zustand/README.md)

## Funcionalidades

Presentes en ambas versiones:

- Alta de pacientes con nombre, propietario, email, fecha de alta y síntomas.
- Listado de pacientes con mensaje de estado vacío.
- Edición de un paciente reutilizando el mismo formulario.
- Eliminación de pacientes.
- Persistencia en `localStorage`.
- Diseño responsive con Tailwind CSS.

No hay backend, API, base de datos, autenticación ni tests automatizados en ninguna de las dos versiones. Los datos viven solo en el `localStorage` del navegador.

## Comparación

| Aspecto | `patients/` | `patients-zustand/` |
| --- | --- | --- |
| Lenguaje | JavaScript (JSX) | TypeScript (TSX) |
| React | `^18.2.0` | `^19.3.1` |
| Bundler | Vite `^4.2.0` | Vite `^8.3.4` |
| Estilos | Tailwind CSS `^3.3.1` con PostCSS | Tailwind CSS `^4.3.3` con `@tailwindcss/vite` |
| Estado compartido | `useState` en `App` y props | Store de Zustand |
| Persistencia | Manual: lectura inicial con `JSON.parse` y `useEffect` que escribe en `localStorage` | Middleware `persist` (clave `patient-storage`) |
| Formulario | Un `useState` por campo (inputs controlados) | React Hook Form (`register`, `handleSubmit`, `setValue`, `reset`) |
| Validación | Chequeo de campos vacíos con un único mensaje | Mensajes por campo, más patrón de email |
| Feedback al usuario | Mensaje de error en el formulario y `confirm()` al eliminar | Toasts al registrar, actualizar y eliminar |
| Identificadores | Función propia (`Date.now` y `Math.random` en base 36) | `uuid` v4 |
| Tipado | No aplica | Tipos `Patient` y `DraftPatient` |
| Herramientas de desarrollo | Sin script de lint | Script de lint con ESLint y type-check en el build; Redux DevTools mediante el middleware `devtools` |
| Dependencias de runtime | `react`, `react-dom` | `react`, `react-dom`, `zustand`, `react-hook-form`, `react-toastify`, `uuid` |

### Qué aporta cada enfoque

**Estado local (versión 1).** Es la opción más simple: no requiere dependencias adicionales y el flujo de datos se puede seguir leyendo `App.jsx`. Con pocos componentes y un solo nivel de anidación, pasar props resulta razonable. El costo aparece al crecer: cada componente que necesita un dato o una acción debe recibirlo por props, y `App` concentra la lógica de eliminación mientras que `Formulario` concentra la de alta y edición.

**Zustand (versión 2).** Los componentes leen y modifican el estado directamente desde el store mediante selectores, sin pasar props intermedias, y las acciones quedan definidas en un solo archivo. Persistencia y DevTools se agregan como middleware. A cambio, suma una dependencia y un concepto más, que para una aplicación de este tamaño no es estrictamente necesario.

**Formularios.** Con estado manual, el código es explícito pero repetitivo: un estado, un `value` y un `onChange` por campo, y el reinicio campo por campo. React Hook Form reduce ese código y permite declarar las reglas de cada campo junto al input. La versión manual, en cambio, deja visible cómo funcionan los inputs controlados, que es el objetivo de aprendizaje de la primera implementación.

**Experiencia de usuario.** La versión 2 notifica cada operación con toasts y muestra errores por campo. La versión 1 pide confirmación antes de eliminar con `confirm()`, algo que la versión 2 no hace.

**Mantenibilidad y escalabilidad.** La versión 2 escala mejor si se agregan pantallas o más entidades: el estado compartido no depende de la jerarquía de componentes y el tipado ayuda a detectar errores de datos. Para una aplicación de una sola pantalla como esta, las diferencias prácticas son menores.

## Estructura del repositorio

```
Patient-Administrator/
├── README.md
├── patients/                  # Versión 1: estado local (JSX)
│   ├── README.md
│   ├── index.html
│   ├── package.json
│   ├── tailwind.config.js
│   ├── postcss.config.js
│   ├── vite.config.js
│   └── src/
│       ├── main.jsx
│       ├── App.jsx
│       ├── index.css
│       └── components/
│           ├── Error.jsx
│           ├── Formulario.jsx
│           ├── Header.jsx
│           ├── ListadoPaciente.jsx
│           └── Paciente.jsx
└── patients-zustand/          # Versión 2: Zustand + React Hook Form (TSX)
    ├── README.md
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

## Instalación y ejecución

Cada versión es una aplicación independiente, con su propio `package.json` y su propio `package-lock.json`. Las dependencias no se comparten, así que hay que instalar dentro de la carpeta que se quiere ejecutar. Ambas usan `npm`.

Requisitos de Node.js según las versiones de Vite declaradas en cada lockfile:

- `patients/`: Node.js 14.18 o superior (excepto 15), o 16 o superior.
- `patients-zustand/`: Node.js 20.19 o superior, o 22.12 o superior. ESLint requiere además 20.19, 22.13 o superior.

```bash
git clone https://github.com/lautaroVal/Patient-Administrator.git
cd Patient-Administrator
```

Versión 1:

```bash
cd patients
npm install
npm run dev
```

Versión 2:

```bash
cd patients-zustand
npm install
npm run dev
```

Los scripts de cada versión están detallados en sus README.

## Aspectos técnicos y aprendizajes

- Comparación directa de dos formas de manejar el mismo estado: elevado a `App` con props frente a un store global con selectores.
- Inputs controlados frente a inputs registrados con React Hook Form, y su impacto en la cantidad de código y en la validación.
- Persistencia manual con `useEffect` y `localStorage` frente al middleware `persist` de Zustand.
- Tipado del dominio con TypeScript (`Patient` y `DraftPatient`) en la segunda versión.
- Notificaciones desacopladas de los componentes que las disparan mediante `react-toastify`.
- El historial de commits sigue el orden en que se incorporó cada herramienta: React Hook Form, validaciones, Zustand, edición y eliminación, Toastify y persistencia.
