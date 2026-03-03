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
---

### 2. **Unity Catalog - Catalog y Schema** (REQUERIDO)

Según la política del workspace, debes usar el catálogo **`users`** con tu schema personal:

**Configuración recomendada:**
```python
catalog = "users"                    # ✅ Usa el catálogo que tengas acesso y permiso
schema = "tu_nombre"                 # ✅ El schema 
model_name = "stateful_lakebase_agent"  # Nombre de tu modelo
```

**Permisos necesarios:**
* `USE CATALOG` en `users`
* `USE SCHEMA` en tu schema personal
* `CREATE MODEL` en tu schema

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
**¡Listo para ejecutar!** 🚀
