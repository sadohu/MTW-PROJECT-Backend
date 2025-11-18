# Documentación del Proyecto MTW-PROJECT-Backend

Bienvenido a la documentación completa del sistema MTW-PROJECT-Backend. Esta carpeta contiene análisis detallados, arquitectura técnica y recomendaciones para el proyecto.

## Contenido de la Documentación

### 📋 [ANALISIS_SISTEMA.md](./ANALISIS_SISTEMA.md)
**Análisis completo del dominio del negocio y funcionalidades**

Contenido:
- Descripción del propósito del sistema
- Modelo de datos completo con todas las entidades
- Lógica de negocio y flujos principales
- Arquitectura del sistema en capas
- Características técnicas
- Observaciones y áreas de mejora
- Conclusiones sobre lo que se quería plantear

**Ideal para:**
- Entender qué hace el sistema
- Conocer el modelo de negocio
- Comprender las reglas de negocio
- Identificar áreas de mejora

---

### 🏗️ [ARQUITECTURA_TECNICA.md](./ARQUITECTURA_TECNICA.md)
**Análisis técnico profundo de la arquitectura y tecnologías**

Contenido:
- Stack tecnológico detallado
- Estructura de paquetes y organización del código
- Flujos de datos y diagramas de secuencia
- Patrones de diseño implementados
- Configuración de seguridad (y problemas identificados)
- Configuración de JPA/Hibernate
- Análisis de rendimiento
- Integración con frontend
- Documentación con Swagger

**Ideal para:**
- Desarrolladores nuevos en el proyecto
- Entender decisiones técnicas
- Optimización de rendimiento
- Refactorización de código

---

### 🚀 [RECOMENDACIONES_Y_ROADMAP.md](./RECOMENDACIONES_Y_ROADMAP.md)
**Plan de acción para mejoras y evolución del sistema**

Contenido:
- **Mejoras Críticas Prioritarias:**
  - Seguridad de autenticación (JWT, BCrypt)
  - Protección de credenciales
  - Manejo de excepciones
  - Corrección de tipos de datos
  - Validaciones completas

- **Mejoras de Mediano Plazo:**
  - Implementación de DTOs
  - Suite de testing completa
  - Auditoría de cambios
  - Optimización de consultas

- **Optimizaciones de Largo Plazo:**
  - Caché con Redis
  - Paginación
  - Sistema de notificaciones
  - Dashboard y analytics

- **Roadmap de Implementación** (6+ sprints)
- **Checklist de Seguridad**
- **Mejores Prácticas de Desarrollo**
- **Recursos y Referencias**

**Ideal para:**
- Planificación de sprints
- Priorización de tareas
- Implementación de mejoras
- Seguimiento de progreso

---

## Resumen Ejecutivo

### ¿Qué es MTW-PROJECT-Backend?

El sistema MTW-PROJECT-Backend es una **plataforma integral de gestión de servicios de transporte corporativo** diseñada para el mercado peruano. Permite a empresas gestionar todo el ciclo de vida de reservas de movilidad, desde la solicitud hasta la facturación.

### Funcionalidades Principales

1. **Gestión de Reservas (Bookings)**
   - Creación y seguimiento de reservas de transporte
   - Estados: En Reserva → Pendiente → En Proceso → Finalizado
   - Asignación de conductores
   - Gestión de pagos (conductor y cliente)

2. **Gestión de Actores**
   - Empresas clientes
   - Pasajeros
   - Conductores y sus vehículos

3. **Facturación y Contabilidad**
   - Generación de facturas con IGV (Perú)
   - Registro contable
   - Múltiples monedas

4. **Reportes**
   - Exportación a Excel (Apache POI)
   - Reportes de empresas (actualmente)

5. **Autenticación**
   - Sistema de login básico
   - ⚠️ Requiere mejoras de seguridad urgentes

### Tecnologías Clave

- **Backend:** Spring Boot 3.1.6, Java 17
- **Base de Datos:** MySQL
- **ORM:** JPA/Hibernate
- **Documentación API:** Swagger UI
- **Frontend:** Angular (puerto 4200)

### Estado Actual del Proyecto

✅ **Fortalezas:**
- Arquitectura clara en capas
- Modelo de dominio bien definido
- Gestión de estados de reservas implementada
- Integración con Swagger

⚠️ **Áreas Críticas:**
- **Seguridad:** Contraseñas sin encriptar, sin JWT
- **Validación:** Validaciones limitadas
- **Testing:** Sin suite de tests
- **Tipos de Datos:** Uso incorrecto de String para valores monetarios

### Próximos Pasos Recomendados

**Prioridad CRÍTICA (Inmediato):**
1. Implementar Spring Security + JWT
2. Encriptar contraseñas con BCrypt
3. Proteger credenciales de base de datos
4. Implementar manejo centralizado de excepciones

**Prioridad ALTA (1-2 meses):**
1. Corregir tipos de datos monetarios (String → BigDecimal)
2. Implementar validaciones completas
3. Crear suite de tests
4. Implementar DTOs

**Prioridad MEDIA (3-6 meses):**
1. Optimización de consultas
2. Implementación de auditoría
3. Paginación y ordenamiento
4. Sistema de notificaciones

Ver [RECOMENDACIONES_Y_ROADMAP.md](./RECOMENDACIONES_Y_ROADMAP.md) para el plan detallado.

---

## Diagramas

### Diagrama de Flujo de Reservas

```
┌─────────────────┐
│  Nueva Reserva  │
└────────┬────────┘
         │
         ▼
   ¿Tiene conductor?
         │
    ┌────┴────┐
   SÍ        NO
    │          │
    ▼          ▼
"Pendiente"  "En Reserva"
    │          │
    └────┬─────┘
         │
         ▼
   ¿Servicio inicia?
         │
         ▼
   "En Proceso"
         │
         ▼
   ¿Servicio termina?
         │
         ▼
   "Finalizado"
         │
         ▼
    Facturación
```

### Diagrama de Entidades Principales

```
       Company
          │
          │ 1
          │
          │ N
       Booking ─────────> Bill
          │
          ├──> Driver
          ├──> Passenger
          ├──> Area
          ├──> Ubigeo (Pickup)
          ├──> Ubigeo (Destination)
          └──> Currency
```

---

## Preguntas Frecuentes

### ¿Cómo empiezo a trabajar en el proyecto?

1. Lee [ANALISIS_SISTEMA.md](./ANALISIS_SISTEMA.md) para entender el negocio
2. Revisa [ARQUITECTURA_TECNICA.md](./ARQUITECTURA_TECNICA.md) para entender la estructura
3. Consulta [RECOMENDACIONES_Y_ROADMAP.md](./RECOMENDACIONES_Y_ROADMAP.md) para tareas pendientes

### ¿Qué mejoras debo implementar primero?

Sigue el roadmap en [RECOMENDACIONES_Y_ROADMAP.md](./RECOMENDACIONES_Y_ROADMAP.md), comenzando con las mejoras críticas de seguridad.

### ¿Dónde está la documentación de la API?

Una vez que ejecutes el proyecto, accede a:
- Swagger UI: `http://localhost:8080/swagger-ui.html`
- OpenAPI JSON: `http://localhost:8080/v3/api-docs`

### ¿Cómo contribuir?

1. Crea una rama feature: `git checkout -b feature/mi-mejora`
2. Implementa los cambios
3. Escribe tests
4. Actualiza documentación si es necesario
5. Crea un Pull Request

---

## Glosario de Términos

- **Booking (Reserva):** Solicitud de servicio de transporte
- **Ubigeo:** Sistema de codificación geográfica de Perú (Departamento-Provincia-Distrito)
- **IGV:** Impuesto General a las Ventas (Perú)
- **RUC:** Registro Único de Contribuyentes (identificador tributario peruano de 11 dígitos)
- **Area:** Departamento o área organizacional de una empresa

---

## Información de Contacto

Para preguntas sobre esta documentación o el proyecto:
- Revisa los archivos de documentación
- Consulta el código fuente
- Revisa issues en el repositorio

---

## Historial de Versiones

| Versión | Fecha      | Cambios                           |
|---------|------------|-----------------------------------|
| 1.0     | 2025-11-18 | Documentación inicial completa    |

---

## Licencia

[Especificar licencia del proyecto]

---

**Última actualización:** 2025-11-18
**Mantenido por:** Equipo de Desarrollo MTW Project
