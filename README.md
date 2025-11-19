🧠 MLOps Tutorial – Entrenamiento, Registro y Gestión de Modelos con MLflow + GitHub Actions + Azure Blob

Este proyecto implementa un flujo completo de MLOps, desde el entrenamiento del modelo hasta su registro automático en MLflow, utilizando:

GitHub Actions para CI/CD

Azure Blob Storage como almacenamiento de artefactos

MLflow Tracking Server como registro de experimentos y modelos

Python + scikit-learn para el modelo

pytest para tests automatizados

El objetivo es entrenar un modelo con el dataset Adult Income, registrarlo automáticamente en MLflow y crear nuevas versiones cada vez que se hace push a main.

📌 Arquitectura del Proyecto
├── src/
│   ├── main.py               # Entrena el modelo y lo guarda
│   ├── data_loader.py        # Carga los datos UCI Adult
│   ├── model.py              # Pipeline y funciones de entrenamiento
│   ├── evaluate.py           # Evaluación del modelo
│
├── models/
│   └── model.pkl             # Modelo entrenado (generado automáticamente)
│
├── scripts/
│   └── register_model.py     # Registra el modelo en MLflow
│
├── model_tests/
│   └── test_model.py         # Tests automatizados
│
├── run_id.txt                # Guarda el RUN_ID del último entrenamiento
├── requirements.txt
└── .github/workflows/build.yml  # Pipeline CI/CD

🚀 Flujo completo del pipeline
1️⃣ GitHub Actions se activa al hacer push a main

El workflow:

Instala dependencias

Descarga el dataset Adult

Ejecuta src/main.py

Guarda el run_id.txt

Ejecuta tests

Registra el modelo en MLflow con scripts/register_model.py

2️⃣ Entrenamiento – src/main.py

Conecta al MLflow Tracking Server usando:

mlflow.set_tracking_uri(os.getenv("MLFLOW_URL"))


Inicia el experimento definido en:

EXPERIMENT_NAME: adult-income-jordi-g


Entrena un pipeline scikit-learn

Evalúa el modelo

Guarda:

artefactos → Azure Blob (vía MLflow)

run_id → run_id.txt

3️⃣ Registro automático del modelo – scripts/register_model.py

Toma el RUN_ID del entrenamiento y ejecuta:

mlflow.register_model(
    model_uri=f"runs:/{run_id}/models/model.pkl",
    name=model_name
)


✔️ Si el modelo no existe → lo crea
✔️ Si ya existe → crea una nueva versión

📦 Cómo funcionan los artefactos

MLflow guarda los artefactos en Azure Blob Storage, usando la connection string:

AZURE_STORAGE_CONNECTION_STRING


Eso permite almacenar:

model.pkl

métricas

parámetros

gráficos

🧨 Errores que tuvimos y cómo los solucionamos
❌ Error 1 – MLflow no encontraba el tracking server (404 HTML error)

Log:

API request ... failed with error code 404


Causa:

En el código usabas MLFLOW_URI

En GitHub Actions solo existía MLFLOW_URL

Resultado:

set_tracking_uri() recibía None

Python hablaba con un servidor inexistente → HTML 404

✔️ Solución:

Unificar todo a MLFLOW_URL:

mlflow.set_tracking_uri(os.getenv("MLFLOW_URL"))


Y en GitHub Actions:

MLFLOW_URL: ${{ vars.MLFLOW_URL }}

❌ Error 2 – Run not found (Run 'XXXX' not found)

Log:

Run '87694ee4...' not found


Causa:

El entrenamiento sí usaba el MLflow remoto

Pero register_model.py estaba usando otro backend (local ./mlruns)

MLflow buscaba el run localmente en vez de en el server remoto

✔️ Solución:

Hacer que ambos scripts usen la misma URI:

mlflow.set_tracking_uri(os.getenv("MLFLOW_URL"))

❌ Error 3 – Azure Blob fallaba (ValueError: Connection string missing required details)

Log:

ValueError: Connection string missing required connection details.
KeyError: 'ACCOUNTNAME'


Causa:

En GitHub Actions, el secret AZURE_STORAGE_CONNECTION_STRING estaba mal.

Era una URL o valor incompleto.

Azure Blob requiere este formato exacto:

DefaultEndpointsProtocol=https;
AccountName=XXXX;
AccountKey=YYYY;
EndpointSuffix=core.windows.net

✔️ Solución:

Copiar la connection string completa desde:

Azure Portal → Storage Account → Access Keys → Connection string

Ponerla en el secret:

AZURE_STORAGE_CONNECTION_STRING

🎯 Resultado Final: Pipeline funcional

Una vez corregidos los tres errores:

✔️ Entrena correctamente
✔️ Guarda artefactos en Azure Blob
✔️ Registra modelos en MLflow
✔️ Crea nuevas versiones automáticamente
✔️ Tests se ejecutan sin fallos

🏁 Comandos útiles
Ejecutar entrenamiento local:
python src/main.py

Registrar el modelo manualmente:
python scripts/register_model.py
