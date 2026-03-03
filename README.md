%md
# 📋 README - Prerequisitos para Ejecutar este Notebook

## 🎯 Resumen

Este notebook despliega un agente con estado (stateful agent) que usa:
* **LangGraph** para orquestación
* **Lakebase** para persistencia de memoria conversacional
* **Unity Catalog** para registro de modelos
* **Model Serving** para deployment en producción

---

## ✅ Prerequisitos Requeridos

### 1. **Lakebase Instance** (REQUERIDO)

Necesitas crear una instancia de Lakebase para almacenar la memoria conversacional:

**Pasos para crear:**
1. Abre **Lakebase App** desde el apps switcher (☰)
2. Click en **"New Project"** y selecciona **"Autoscaling"**
3. Configura:
   * **Project name**: `dv-agents-memory` (o tu nombre preferido)
   * **PostgreSQL version**: 17
   * **Compute settings**:
     * Min CU: 0.5 (para scale-to-zero y ahorro de costos)
     * Max CU: 1.0 (suficiente para demos)
     * Suspend timeout: 300s (5 minutos)

**⚠️ IMPORTANTE**: Actualiza el nombre de la instancia en Cell 3 (línea 42):
```python
LAKEBASE_INSTANCE_NAME = "TU_NOMBRE_DE_INSTANCIA"  # Cambiar esto
```

**Estimación de Costos**: ~$0.10-0.50/día con scale-to-zero habilitado

---

### 2. **Unity Catalog - Catalog y Schema** (REQUERIDO)

Según la política del workspace, debes usar el catálogo **`users`** con tu schema personal:

**Configuración recomendada:**
```python
catalog = "users"                    # ✅ Usa el catálogo users
schema = "tu_nombre"                 # ✅ Tu schema personal (ejemplo: daniel_vargas)
model_name = "stateful_lakebase_agent"  # Nombre de tu modelo
```

**Permisos necesarios:**
* `USE CATALOG` en `users`
* `USE SCHEMA` en tu schema personal
* `CREATE MODEL` en tu schema

**Para configurar** (actualiza Cell 11):
1. Reemplaza los espacios en blanco con tus valores
2. Asegúrate de que tu schema existe:
   ```sql
   CREATE SCHEMA IF NOT EXISTS users.tu_nombre;
   ```

**✅ Verificación:**
```python
from databricks.sdk import WorkspaceClient
w = WorkspaceClient()

# Verifica acceso al catálogo
try:
    cat = w.catalogs.get("users")
    print(f"✅ Acceso a catálogo: {cat.name}")
except:
    print("❌ Sin acceso al catálogo users")
```

---

### 3. **LLM Endpoint** (REQUERIDO)

El notebook usa **`databricks-claude-3-7-sonnet`**

**Verifica disponibilidad:**
```python
from databricks.sdk import WorkspaceClient
w = WorkspaceClient()

try:
    endpoint = w.serving_endpoints.get("databricks-claude-3-7-sonnet")
    print(f"✅ Endpoint disponible: {endpoint.state.ready}")
except:
    print("❌ Endpoint no encontrado - actualiza LLM_ENDPOINT_NAME en Cell 3")
```

**Alternativas si no está disponible:**
* `databricks-meta-llama-3-3-70b-instruct`
* `databricks-dbrx-instruct`

Actualiza `LLM_ENDPOINT_NAME` en Cell 3 (línea 38) si usas otro endpoint.

---

## 🔧 Pasos de Configuración

### **Paso 1: Crear Lakebase Instance**
```
Lakebase App → New Project → Autoscaling
Name: dv-agents-memory (o tu nombre)
PG Version: 17
```

### **Paso 2: Actualizar Cell 3 - agent.py**
Cambia la línea 42:
```python
LAKEBASE_INSTANCE_NAME = "tu-instancia-lakebase"  # ← ACTUALIZAR AQUÍ
```

### **Paso 3: Actualizar Cell 11 - UC Registration**
Reemplaza los espacios vacíos:
```python
catalog = "users"                           # Catálogo users
schema = "tu_nombre_apellido"              # Tu schema personal
model_name = "stateful_lakebase_agent"     # Nombre del modelo
```

### **Paso 4: Ejecutar el Notebook**
1. **Cell 2**: Instala dependencias (~2-3 minutos)
2. **Cell 3**: Crea agent.py con Lakebase config
3. **Cells 4-6**: Prueba el agente localmente
4. **Cell 7**: Restart Python (limpia conexiones)
5. **Cell 9**: Log a MLflow
6. **Cell 12**: Registra en Unity Catalog
7. **Cell 14**: Deploy a Model Serving

---

## 📊 Políticas del Workspace (Field Eng)

**Retención y Tags:**
* ✅ Tu schema personal en `users.*` NO necesita tags RemoveAfter
* ⚠️ Model Serving endpoints sin tag RemoveAfter se eliminan después de 14 días
* ⚠️ Lakebase instance debe tener justificación o se elimina en 30 días

**Para mantener recursos más tiempo:**
1. Agrega tag `RemoveAfter` con fecha YYYY-MM-DD
2. O solicita excepción en [FEINFRA Jira](https://go/freshservice)

---

## 🚨 Troubleshooting

### Error: "Lakebase instance not found"
**Causa**: Nombre de instancia incorrecto en Cell 3
**Solución**: 
1. Lista tus instancias: Lakebase App → Projects
2. Actualiza `LAKEBASE_INSTANCE_NAME` en Cell 3

### Error: "Schema not found"
**Causa**: Schema no existe en Unity Catalog
**Solución**:
```sql
CREATE SCHEMA IF NOT EXISTS users.tu_nombre;
```

### Error: "Permission denied on catalog"
**Causa**: Sin permisos en catálogo users
**Solución**: Pide permisos a workspace admin o usa otro workspace

### Error: "LLM endpoint not found"
**Causa**: Endpoint no disponible en el workspace
**Solución**: Lista endpoints disponibles y actualiza Cell 3:
```python
from databricks.sdk import WorkspaceClient
w = WorkspaceClient()
for ep in w.serving_endpoints.list():
    if 'claude' in ep.name or 'llama' in ep.name:
        print(ep.name)
```

---

## 💡 Mejores Prácticas

1. **Scale-to-zero**: Configura tu Lakebase instance para scale-to-zero (min_cu=0.5) para ahorrar costos
2. **Nombres descriptivos**: Usa nombres claros para tu modelo y endpoint
3. **Testing local primero**: Ejecuta Cells 4-6 antes de deployar
4. **Limpieza**: Detén Model Serving endpoints cuando no los uses
5. **Git backup**: Guarda este notebook en un repo Git

---

## 📚 Referencias

* [Lakebase Documentation](https://docs.databricks.com/en/oltp/projects/)
* [Unity Catalog Models](https://docs.databricks.com/en/mlflow/models-in-uc.html)
* [Databricks Agents](https://docs.databricks.com/en/generative-ai/agent-framework/)
* [Workspace Policy](go/fe/workspaces)

---

## ✅ Checklist Pre-Ejecución

- [ ] Lakebase instance creada y nombre actualizado en Cell 3
- [ ] Valores de catalog/schema/model_name actualizados en Cell 11
- [ ] LLM endpoint verificado y disponible
- [ ] Permisos de Unity Catalog confirmados
- [ ] Compute serverless disponible (default en este workspace)

**¡Listo para ejecutar!** 🚀
