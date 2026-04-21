# README Sprint 3 - Deploy y operacion de API

## Objetivo
Desplegar y operar la API de inferencia usando los artefactos publicados por Build, validando salud del servicio y calidad minima operativa.

## Alcance de Sprint 3
1. Preparar app de serving y dependencias.
2. Configurar y ejecutar despliegue manual con GitHub Actions.
3. Validar endpoints funcionales post-deploy.
4. Definir runbook de rollback y verificacion.

## Requisitos previos
- Build de modelo exitoso en main (Sprint 2).
- Release existente con estos assets:
  - model.pkl
  - scaler.pkl
  - encoders.pkl
- Secreto configurado en GitHub:
  - RENDER_DEPLOY_HOOK
- Acceso al servicio desplegado (URL publica).

## Archivos clave del Sprint
- .github/workflows/deploy.yml
- deployment/app/main.py
- deployment/requirements.txt

## 1) Verificaciones previas al deploy

### 1.1 Confirmar release con artefactos
En GitHub -> Releases, validar que la ultima release contiene:
- model.pkl
- scaler.pkl
- encoders.pkl

### 1.2 Confirmar workflow de deploy
Archivo esperado:
- .github/workflows/deploy.yml

Funcion esperada:
- Trigger manual via workflow_dispatch.
- Llamada por curl al secreto RENDER_DEPLOY_HOOK.

### 1.3 Confirmar dependencias de serving
Archivo:
- deployment/requirements.txt

Dependencias minimas esperadas:
- fastapi
- uvicorn
- requests
- scikit-learn
- pandas
- numpy
- joblib

## 2) Test local opcional de la API (antes de desplegar)

### 2.1 Instalar dependencias de serving
Comando:
- pip install -r deployment/requirements.txt

### 2.2 Definir repo origen de artefactos (opcional)
Si no defines variable, la app usa un repo por defecto.

Windows PowerShell:
- $env:GITHUB_REPO = "<owner>/<repo>"

Linux o macOS:
- export GITHUB_REPO="<owner>/<repo>"

### 2.3 Levantar API localmente
Comando:
- uvicorn deployment.app.main:app --host 0.0.0.0 --port 8000

### 2.4 Probar health
Comando:
- curl http://localhost:8000/health

Respuesta esperada:
- {"status":"ok"}

### 2.5 Probar prediccion
Comando de ejemplo:
- curl -X POST http://localhost:8000/predict -H "Content-Type: application/json" -d '{"age":39,"workclass":"State-gov","fnlwgt":77516,"education":"Bachelors","education-num":13,"marital-status":"Never-married","occupation":"Adm-clerical","relationship":"Not-in-family","race":"White","sex":"Male","capital-gain":2174,"capital-loss":0,"hours-per-week":40,"native-country":"United-States"}'

Respuesta esperada:
- JSON con campos prediction y duration.

### 2.6 Probar metricas
Comando:
- curl http://localhost:8000/metrics

Respuesta esperada:
- total_predictions <numero>

## 3) Ejecutar deploy en GitHub Actions

### 3.1 Disparar workflow manual
1. Ir a GitHub -> Actions -> Deploy Model.
2. Click en Run workflow.
3. Seleccionar branch main.
4. Ejecutar.

### 3.2 Verificar ejecucion
En logs del job deploy validar:
- Checkout repository completado.
- Paso Deploy to Render completado (curl al hook sin error).

## 4) Validacion post-deploy en entorno remoto

Asumiendo URL del servicio en variable SERVICE_URL.

Windows PowerShell:
- $env:SERVICE_URL = "https://tu-servicio.onrender.com"

Linux o macOS:
- export SERVICE_URL="https://tu-servicio.onrender.com"

### 4.1 Health check
Comando:
- curl $env:SERVICE_URL/health

(En Linux/macOS: curl $SERVICE_URL/health)

### 4.2 Smoke test de prediccion
Comando:
- curl -X POST $env:SERVICE_URL/predict -H "Content-Type: application/json" -d '{"age":39,"workclass":"State-gov","fnlwgt":77516,"education":"Bachelors","education-num":13,"marital-status":"Never-married","occupation":"Adm-clerical","relationship":"Not-in-family","race":"White","sex":"Male","capital-gain":2174,"capital-loss":0,"hours-per-week":40,"native-country":"United-States"}'

(En Linux/macOS usa $SERVICE_URL en lugar de $env:SERVICE_URL)

### 4.3 Verificar metricas
Comando:
- curl $env:SERVICE_URL/metrics

## 5) Orden operativo recomendado
1. Integration (validar PR).
2. Build (entrenar y publicar release).
3. Deploy (desplegar consumo de ultima release).

## 6) Runbook de rollback
Si el deploy falla o degrada comportamiento:
1. Identificar ultima release estable de modelo.
2. Regenerar release estable como latest (o volver a disparar build estable).
3. Ejecutar de nuevo Deploy Model.
4. Repetir pruebas de /health y /predict.

## 7) Checklist de cierre Sprint 3
- Deploy Model ejecutado en success.
- Servicio remoto responde /health correctamente.
- Endpoint /predict responde con prediccion valida.
- Endpoint /metrics refleja trafico de pruebas.
- Documentado procedimiento de rollback.

## Troubleshooting rapido
- Error: Failed to load artifacts en arranque
  - Verificar que existe una latest release con model.pkl, scaler.pkl y encoders.pkl.

- Error 500 en /predict
  - Validar payload: nombres de columnas y valores categoricos deben ser compatibles con encoders.

- Deploy no inicia
  - Verificar secreto RENDER_DEPLOY_HOOK y permisos del repositorio para Actions.

- Health OK pero predict falla
  - Revisar logs de la app para detectar columnas faltantes o tipos de dato invalidos.
