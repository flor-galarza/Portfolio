# sv

Everything you need to build a Svelte project, powered by [`sv`](https://github.com/sveltejs/cli).

## Creating a project

If you're seeing this, you've probably already done this step. Congrats!

```sh
# create a new project in the current directory
npx sv create

# create a new project in my-app
npx sv create my-app
```

## Developing

Once you've created a project and installed dependencies with `npm install` (or `pnpm install` or `yarn`), start a development server:

```sh
npm run dev

# or start the server and open the app in a new browser tab
npm run dev -- --open
```

## Building

To create a production version of your app:

```sh
npm run build



Genial 😄
Te dejo **un README.md listo para copiar/pegar**, escrito **nivel portfolio profesional**, explicando **qué hiciste, cómo y por qué**. Esto suma muchísimo en entrevistas.

---

# 📁 Portfolio – Project Management & IT Delivery

Este repositorio contiene mi **portfolio profesional** orientado a **Project Management / Analista de Proyectos TI**, desarrollado como un **producto real**, aplicando prácticas de **gestión ágil, trazabilidad y CI/CD**.

El objetivo no es solo mostrar el resultado visual, sino **cómo gestiono el trabajo**, **cómo integro herramientas** y **cómo aseguro entregas incrementales de valor**.

---

## 🎯 Objetivo del proyecto

Construir un portfolio web aplicando:

* Gestión por **Historias de Usuario**
* **Kanban** como modelo de flujo
* **Git Flow simplificado**
* Integración **Jira ↔ GitHub**
* Automatizaciones basadas en eventos reales del ciclo de vida del código

---

## 🧩 Metodología de trabajo

### 🟦 Modelo de gestión

* **Kanban** (flujo continuo)
* **Épicas = Releases**
* **Historias de Usuario = Features**

Cada release entrega un incremento funcional del portfolio.

---

## 🟣 Releases (Épicas)

### 🔹 Release v0.1.0 – Fundaciones

* Home page base
* Identidad visual
* Layout global
* Espacios definidos para biografía y proyectos (placeholders)

> Enfocado en sentar los **cimientos del producto**, sin contenido final.

---

## 🗂️ Flujo Kanban

Estados del tablero:

* **POR HACER**
* **EN CURSO**
* **EN REVISIÓN**
* **LISTO PARA RELEASE**
* **FINALIZADO**

---

## 🔄 Git Flow aplicado

Se utiliza un **Git Flow simplificado**:

* `feature/*` → desarrollo de historias
* `develop` → integración
* `release/*` → preparación de versiones
* `main` → producción

---

## 🔗 Trazabilidad Jira ↔ GitHub

Cada historia de usuario está vinculada automáticamente a:

* Ramas
* Commits
* Pull Requests
* Merges

Esto se logra utilizando la **GitHub App oficial de Jira Software** y una convención estricta de nombres:

```text
feature/KAN-11-descripcion
KAN-11: título del PR
feat(KAN-11): mensaje de commit
```

---

## ⚙️ Automatizaciones implementadas (Jira Automation)

El estado de las historias se actualiza automáticamente según eventos reales:

| Evento técnico        | Estado Jira        |
| --------------------- | ------------------ |
| Rama creada (KAN-XX)  | EN CURSO           |
| PR creado a develop   | EN REVISIÓN        |
| PR mergeado a develop | LISTO PARA RELEASE |
| PR mergeado a main    | FINALIZADO         |

Esto asegura que **el tablero refleja el estado real del código**, sin actualizaciones manuales.

---

## 🚀 CI/CD

* Validaciones automáticas en Pull Requests
* Protecciones de rama en `develop` y `main`
* Solo se permite merge si los checks pasan correctamente

> Un ítem en **FINALIZADO** implica que el cambio está **integrado y publicado**.

---

## 🛠️ Tecnologías y herramientas

* **Frontend:** Svelte / SvelteKit
* **Gestión:** Jira Software
* **Control de versiones:** Git & GitHub
* **Automatización:** Jira Automation
* **CI/CD:** GitHub Actions

---

## 🧠 Aprendizajes clave

* Cómo alinear **gestión de proyectos** con **eventos técnicos reales**
* Evitar burocracia innecesaria manteniendo trazabilidad
* Diseñar flujos defendibles en entornos profesionales
* Integrar herramientas como lo hacen equipos reales de TI

---

## 📌 Nota final

Este repositorio forma parte de un **portfolio vivo**, que seguirá evolucionando con nuevas releases y mejoras.

El foco está puesto tanto en el **producto** como en el **proceso**.

---


You can preview the production build with `npm run preview`.

> To deploy your app, you may need to install an [adapter](https://svelte.dev/docs/kit/adapters) for your target environment.
