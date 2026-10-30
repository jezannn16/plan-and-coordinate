![Plan and Coordinate cover](assets/cover.png)

# Plan and Coordinate

[English](README.md) · [Русский](README.ru.md) · [Español](README.es.md)

**A roadmap is not a delivery plan. Make it executable.**

A reusable agent skill for turning phased roadmaps into task-level implementation plans and coordinating people or agents around a shared board.

## What it does

Expand release plans; create one plan per defined phase; assign owners and track dependencies, blockers and progress.

## How it works

Moves from scope and release boundaries to concrete work, then supports a shared AGENTS-STATUS.md board when coordination is requested.

## Requirements

Some planning workflows reference Superpowers skills. The coordination protocol uses Git commits, pushes and an active wait loop; review it before use.

## Install

```bash
mkdir -p ~/.agents/skills
git clone https://github.com/jezannn16/plan-and-coordinate.git ~/.agents/skills/plan-and-coordinate
```

## Try it

> Turn this roadmap into release-by-release implementation plans, then propose a coordination board.

See [SKILL.md](SKILL.md) and [references/](references/) for the complete workflow.
