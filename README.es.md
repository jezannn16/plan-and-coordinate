![Plan and Coordinate cover](assets/cover.png)

# Plan and Coordinate

[English](README.md) · [Русский](README.ru.md) · [Español](README.es.md)

**Convierte una hoja de ruta en trabajo ejecutable.**

Una habilidad para agentes que desglosa entregas en tareas y coordina personas o agentes mediante un tablero compartido.

## Qué hace

Amplía planes de lanzamiento, crea un plan por fase y asigna tareas con dependencias y bloqueos.

## Cómo funciona

Parte del alcance y las fases, define trabajo concreto y, si se solicita coordinación, utiliza un tablero AGENTS-STATUS.md.

## Requisitos

Algunos modos requieren habilidades de Superpowers. El protocolo de coordinación usa commits, push y espera activa; revísalo antes de ejecutarlo.

## Instalación

```bash
mkdir -p ~/.agents/skills
git clone https://github.com/jezannn16/plan-and-coordinate.git ~/.agents/skills/plan-and-coordinate
```

## Ejemplo

> Desglosa esta hoja de ruta en planes por lanzamiento y propone un tablero de coordinación.

Consulta [SKILL.md](SKILL.md) y [references/](references/) para ver el flujo completo.
