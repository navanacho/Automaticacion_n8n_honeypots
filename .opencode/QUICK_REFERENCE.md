# 🚀 Desarrollo - Quick Reference

## Skills Instaladas Localmente

| Skill | Cuándo Usarla | Comando |
|-------|---------------|---------|
| **openspec-design** | Crear/actualizar diseño técnico | \/opsx:design <change>\ |
| **openspec-spec** | Escribir especificaciones | \/opsx:spec <capability>\ |
| **openspec-tasks** | Generar checklist de tareas | \/opsx:tasks <change>\ |
| **openspec-apply-change** | Empezar implementación | \/opsx-apply\ |
| **openspec-verify** | Validar que implementación = specs | \/opsx:verify <change>\ |

## Estado del Proyecto

### ✅ Completado
- ✓ Proposal: \openspec/changes/honeypot-dashboard-interface/proposal.md\
- ✓ Design: \openspec/changes/honeypot-dashboard-interface/design.md\
- ✓ Specs (7): \openspec/changes/honeypot-dashboard-interface/specs/*/spec.md\
- ✓ Tasks (247 tareas): \openspec/changes/honeypot-dashboard-interface/tasks.md\
- ✓ Agents: \.opencode/agents.md\ (230 líneas, listo)
- ✓ Skills: 8 skills instaladas localmente

### 📋 Próximo Paso
Ejecuta: \\\ash
/opsx-apply
\\\

Esto inicia la Fase 1: Database + Auth

---

## Conventions Rápidas

**TypeScript**: \interface\ para APIs, \	ype\ para internos  
**React**: Funcional components, custom hooks, \component.tsx\  
**Express**: RESTful, \{ error, code, details }\, JWT requerido  
**Database**: \snake_case\ tablas y columnas  
**Git**: \eat(scope): description\ (conventional commits)  

Ver más: \.opencode/agents.md\

