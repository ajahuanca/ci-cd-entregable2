# Práctica 2 – Implementación de Pipeline CI/CD con Enfoque DevSecOps

## 1. Introducción

El presente proyecto tiene como objetivo diseñar, implementar y validar un **pipeline CI/CD con enfoque DevSecOps**, partiendo de un repositorio base funcional que integra frontend y backend desarrollados en **JavaScript (Node.js)**, contenedores Docker y orquestación mediante **Docker Compose**.

El enfoque DevSecOps aplicado busca integrar **calidad, seguridad y automatización** desde las primeras etapas del ciclo de desarrollo, asegurando que el código **solo avance** cuando cumple criterios funcionales y de seguridad claramente definidos.

---

## 2. Análisis del repositorio y verificación de funcionalidad previa

Antes de integrar el pipeline CI/CD, se realizó un análisis técnico del repositorio base con el objetivo de comprender su arquitectura y verificar su correcto funcionamiento inicial.

### 2.1 Análisis estructural

El repositorio presenta las siguientes características:

- Separación clara entre **frontend** y **backend**.
- Proyecto desarrollado íntegramente en **JavaScript / Node.js**.
- Uso de **Dockerfiles** para la construcción de imágenes.
- Archivo **docker-compose.yml** en la raíz del proyecto backend para levantar el stack completo.
- Endpoint protegido `/api/auth/login`.
- API Gateway expuesto en el **puerto 3000**.

Asimismo, se verificó la existencia de:
- `package.json` y `package-lock.json` (clave para instalaciones reproducibles).
- Scripts de linting y testing definidos en el proyecto.

---

### 2.2 Verificación funcional previa al pipeline

Antes de aplicar controles automáticos, se validó que el sistema funcionara correctamente en su estado base.

#### Levantamiento del stack

```bash
docker compose up --build
```
Se comprobó que:

* Los contenedores levantan correctamente.
* El API Gateway responde en http://localhost:3000.

**Verificación de control de acceso**
```
curl http://localhost:3000/api/auth/login
```

### 2.3 Conclusión del análisis previo

El repositorio se considera funcional cuando:

* El stack levanta sin errores críticos.
* Los servicios se comunican correctamente.
* No existen exposiciones directas de endpoints protegidos.

Este paso es fundamental, ya que en DevSecOps no se puede automatizar ni asegurar un sistema que no funcione correctamente en su estado inicial.

## 3. Diseño del pipeline CI/CD
### 3.1 Ubicación del pipeline

El pipeline CI/CD fue implementado como workflow de GitHub Actions en:
```
.github/workflows/devsecops.yml
```
Esto permite:

* Ejecución automática en push y pull request.
* Versionamiento del pipeline junto al código.
* Trazabilidad completa de ejecuciones.

### 3.2 Diseño global del flujo

El pipeline sigue la secuencia lógica recomendada por DevSecOps:
```
Code → Quality → Test → Security → Build → Container Security → Run / Verify
```

Cada etapa actúa como un gate automático, impidiendo que el código avance si no cumple los criterios establecidos.

### 3.3 Etapas implementadas
| Etapa del pipeline              | Herramienta                  | ¿Por qué se usa?                                                    | Objetivo en el proyecto                                            |
| ------------------------------- | ---------------------------- | ------------------------------------------------------------------- | ------------------------------------------------------------------ |
| Instalación reproducible        | npm ci                       | Garantiza instalaciones determinísticas usando `package-lock.json`. | Evitar inconsistencias entre entornos y asegurar reproducibilidad. |
| Análisis de calidad de código   | ESLint                       | Detecta errores comunes y malas prácticas en JavaScript.            | Mantener código limpio, consistente y mantenible.                  |
| Testing automático              | Jest (librería del proyecto) | Verifica automáticamente el correcto funcionamiento del sistema.    | Detectar regresiones antes de avanzar a build y seguridad.         |
| Seguridad del código (SAST)     | Semgrep                      | Analiza el código fuente en busca de patrones inseguros.            | Detectar vulnerabilidades antes de ejecutar o desplegar.           |
| Seguridad de dependencias (SCA) | npm audit                    | Identifica CVEs en dependencias externas.                           | Proteger la cadena de suministro del software.                     |
| Build de contenedores           | Docker                       | Empaqueta la aplicación en artefactos portables.                    | Garantizar consistencia entre entornos.                            |
| Versionado de artefactos        | SHA / run-number / semver    | Permite identificar cada imagen generada.                           | Asegurar trazabilidad y rollback.                                  |
| Seguridad de contenedores       | Trivy                        | Escanea imágenes Docker en busca de CVEs.                           | Evitar ejecutar imágenes vulnerables.                              |
| Run / Deploy simulado           | Docker Compose               | Levanta el stack completo del sistema.                              | Validar integración real entre servicios.                          |
| Smoke Test / DAST mínimo        | curl                         | Verifica comportamiento en ejecución real.                          | Confirmar control de acceso y disponibilidad.                      |

## 4. Justificación técnica de decisiones (enfoque DevSecOps)

Se adoptó un enfoque DevSecOps integrando seguridad y calidad desde etapas tempranas (shift-left), convirtiendo el pipeline en un sistema de control automático y no solo informativo.

Las herramientas fueron seleccionadas por su:

* Adecuación al stack JavaScript / Node.js.
* Capacidad de automatización.
* Impacto directo en la mitigación de riesgos reales.

**Justificación por herramienta**

| Herramienta         | Fase DevSecOps          | Riesgo mitigado            | ¿Por qué es necesaria aunque el sistema funcione?   |
| ------------------- | ----------------------- | -------------------------- | --------------------------------------------------- |
| npm ci              | Code / CI               | Builds inconsistentes      | Elimina el “funciona en mi máquina”.                |
| ESLint              | Code / CI               | Errores y malas prácticas  | Código funcional puede ser inseguro o inmantenible. |
| Tests automáticos   | Test                    | Regresiones funcionales    | Protege el sistema ante cambios futuros.            |
| Semgrep             | Security (Shift-left)   | Vulnerabilidades en código | Muchas brechas no rompen funcionalidad.             |
| npm audit           | Security (Supply Chain) | CVEs en dependencias       | Ataques modernos provienen de librerías externas.   |
| Docker              | Build / Release         | Diferencias entre entornos | Asegura consistencia de ejecución.                  |
| Versionado          | Release                 | Falta de trazabilidad      | Permite auditoría y rollback.                       |
| Trivy               | Security (Release)      | CVEs en imágenes Docker    | La imagen puede ser vulnerable aunque funcione.     |
| Docker Compose      | Deploy / Operate        | Errores de integración     | Valida el sistema completo en ejecución.            |
| Smoke / DAST mínimo | Operate                 | Exposición de endpoints    | Verifica seguridad en runtime.                      |

## 5. Ejecución del pipeline

### 5.1 Ejecución automática

El pipeline se ejecuta automáticamente en:

```
push a la rama main
pull request
```

Esto garantiza que ningún cambio llegue a producción sin pasar por todos los controles.

### 5.2 Verificación de etapas (gates)

Se comprobó que:

* Si ESLint falla, el pipeline se detiene.
* Si los tests fallan, no se construyen contenedores.
* Si npm audit detecta HIGH o CRITICAL, el pipeline falla.
* Si Trivy detecta vulnerabilidades HIGH o CRITICAL, la imagen no se considera válida.
* Si el smoke test falla, el job final se detiene.

Esto demuestra que el pipeline actúa como un sistema automático de control.