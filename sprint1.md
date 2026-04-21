# Sprint 1 - Base tecnica y entrenamiento local

## Objetivo
Tener el pipeline de entrenamiento funcionando en local con pruebas unitarias estables.

## Duracion sugerida
1 semana

## Paso a paso
1. Preparar entorno de desarrollo
- Crear y activar entorno virtual.
- Instalar dependencias desde requirements.txt.
- Verificar version de Python y librerias clave.

2. Implementar pipeline de datos y entrenamiento
- Validar carga de datos en src/data_loader.py.
- Validar preprocesado (encoding + escalado).
- Entrenar modelo en src/model.py.
- Orquestar flujo completo en src/main.py.

3. Implementar evaluacion
- Ejecutar evaluacion de metricas en src/evaluate.py.
- Confirmar que accuracy y reporte de clasificacion se imprimen correctamente.

4. Asegurar generacion de artefactos
- Ejecutar pipeline local: python src/main.py.
- Confirmar creacion de models/model.pkl.
- Confirmar creacion de models/scaler.pkl y models/encoders.pkl.

5. Crear y ejecutar pruebas unitarias
- Ejecutar tests en unit_tests/test_data_loader.py.
- Ejecutar tests en unit_tests/test_model.py.
- Ejecutar tests en unit_tests/test_evaluate.py.
- Corregir fallos hasta tener suite en verde.

6. Documentar minima operativa
- Actualizar README.md con instrucciones de setup local.
- Incluir comandos para entrenamiento y tests.

## Criterios de aceptacion (DoD)
- El entrenamiento corre de inicio a fin sin errores.
- Se generan artefactos en la carpeta models.
- Todas las pruebas unitarias pasan en local.
- README documenta los comandos base.

## Riesgos y mitigacion
- Riesgo: inconsistencias en datos de entrada.
- Mitigacion: validaciones tempranas en carga y preprocesado.

- Riesgo: dependencias no reproducibles.
- Mitigacion: fijar versiones en requirements.txt.
