---
tags: [ai-901, azure, ia, laboratorio, foundry]
módulo: Laboratorios
duración: ~30 min
---

# Lab 01 · Explorar Microsoft Foundry

## Objetivo

Crear un **proyecto** de Microsoft Foundry y recorrer el portal para identificar dónde vive cada cosa: modelos, agentes, herramientas, conocimiento, guardrails y administración. Es la base de todos los demás laboratorios.

## Pasos

1. **Crear el proyecto.** Abre `https://ai.azure.com`, inicia sesión y activa la opción **New Foundry** si no lo está. Crea un proyecto con un nombre único; en **Advanced options** indica el **recurso de Foundry**, la suscripción, el grupo de recursos y una **región recomendada**. Espera unos minutos.
2. **Ver la jerarquía.** En la barra superior selecciona el nombre del proyecto → **View all resources**. Observa que cada proyecto tiene un **recurso padre de Microsoft Foundry** que agrupa servicios, usuarios, conexiones y modelos, y que puede tener varios proyectos hijos.
3. **Home.** Localiza la **API key**, el **Project endpoint** y el **Azure OpenAI endpoint**. Los necesitarás en el código.
4. **Discover.** Explora el catálogo de **modelos** y servicios, y los puntos de partida para empezar una solución.
5. **Build.** Recorre las secciones: **Agents**, **Workflows**, **Models** (despliegues), **Fine-tuning**, **Tools**, **Knowledge** (Foundry IQ), **Guardrails**, **Memory**, **Data**, **Evaluations** y **Services** (los playgrounds de Foundry Tools).
6. **Operate.** Mira **assets**, **compliance**, **quota** y las tareas de administración.
7. **Limpieza.** Si no vas a seguir, elimina el grupo de recursos desde el portal de Azure.

## Qué está ocurriendo

Un proyecto de Foundry no es un recurso de Azure independiente: es una unidad lógica dentro del **recurso de Foundry**, que es el que aporta los servicios en la nube. Los endpoints que aparecen en Home son dos puertas distintas: la de **modelos** (compatible con OpenAI, admite clave) y la del **proyecto** (para el SDK y los agentes, solo con identidad de Entra).

## Qué debo aprender para AI-901

- La jerarquía **recurso → proyecto** y qué contiene cada uno.
- La navegación **Discover / Build / Operate** y qué se hace en cada página.
- Que existen dos endpoints y que el del proyecto **no admite clave**.
- Que los despliegues dependen de **región y cuota**.

## Relacionado

- [[Microsoft Foundry]] · [[Foundry Tools (servicios de IA de Azure)]]
- [[Lab 02 - Desplegar un modelo y crear un agente]]

← Volver al índice: [[00 - Índice - Laboratorios]]
