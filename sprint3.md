# Sprint 3 - Despliegue y operacion de API

## Objetivo
Desplegar la API de inferencia en entorno remoto y dejar un flujo operativo estable.

## Duracion sugerida
1 semana

## Paso a paso
1. Preparar aplicacion de serving
- Revisar deployment/app/main.py.
- Confirmar carga de artefactos desde la ultima release de GitHub.
- Validar endpoints /health, /predict y /metrics.

2. Configurar dependencias de despliegue
- Revisar deployment/requirements.txt.
- Verificar compatibilidad de versiones con artefactos entrenados.

3. Configurar workflow de despliegue
- Revisar .github/workflows/deploy.yml.
- Configurar secreto RENDER_DEPLOY_HOOK.
- Ejecutar deploy manual por workflow_dispatch.

4. Validar despliegue en entorno remoto
- Verificar estado de servicio con GET /health.
- Ejecutar prediccion de prueba con POST /predict.
- Revisar logs y metricas basicas.

5. Definir runbook operativo
- Actualizar README.md con orden operativo: Integration, Build, Deploy.
- Incluir procedimiento de rollback (release anterior).
- Incluir checklist de verificacion post-deploy.

## Criterios de aceptacion (DoD)
- Servicio desplegado y accesible.
- Endpoint /predict responde correctamente con payload valido.
- Equipo tiene documentado el proceso de despliegue y rollback.

## Riesgos y mitigacion
- Riesgo: deploy sin artefactos actualizados.
- Mitigacion: ejecutar siempre Build exitoso antes de Deploy.

- Riesgo: fallo en carga de release durante arranque.
- Mitigacion: monitoreo de logs y versionado claro de releases.
