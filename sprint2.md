# Sprint 2 - CI y build de artefactos

## Objetivo
Automatizar validacion de calidad en PR y build del modelo con publicacion de artefactos versionados.

## Duracion sugerida
1 semana

## Paso a paso
1. Configurar workflow de integracion
- Revisar .github/workflows/integration.yml.
- Ejecutar pruebas unitarias con pytest.
- Incluir cobertura con pytest-cov.
- Guardar resultados de pruebas y cobertura.

2. Integrar feedback en Pull Request
- Publicar resumen de resultados como comentario en PR.
- Configurar fallo del job si los tests fallan.

3. Configurar workflow de build del modelo
- Revisar .github/workflows/build.yml.
- Descargar dataset Adult en data/raw.
- Ejecutar entrenamiento con python src/main.py.
- Ejecutar pruebas de modelo en model_tests/test_model.py.

4. Publicar artefactos de build
- Publicar models/model.pkl en Release.
- Publicar models/scaler.pkl en Release.
- Publicar models/encoders.pkl en Release.
- Versionar release por numero de ejecucion o tag.

5. Validar flujo extremo a extremo en rama principal
- Mergear PR validada por Integration.
- Confirmar ejecucion automatica de Build en main.
- Confirmar disponibilidad de artefactos en Releases.

## Criterios de aceptacion (DoD)
- Cada PR ejecuta pruebas y cobertura automaticamente.
- Build en main genera y publica artefactos sin intervencion manual.
- Tests unitarios y tests de modelo quedan en verde en CI.

## Riesgos y mitigacion
- Riesgo: tiempos altos de pipeline por descarga de dataset.
- Mitigacion: cache de dependencias y optimizacion de pasos.

- Riesgo: artefactos incompletos en release.
- Mitigacion: validacion explicita de archivos antes de publicar.
