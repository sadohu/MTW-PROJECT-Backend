# Análisis del Sistema MTW-PROJECT-Backend

## Información General

**Nombre del Proyecto:** MTW-PROJECT-Backend
**Tecnologías:** Spring Boot 3.1.6, Java 17, MySQL, JPA/Hibernate
**Base de Datos:** mtw_project (MySQL)
**Puerto Frontend:** 4200 (Angular)
**Documentación API:** Swagger UI habilitada en `/swagger-ui.html`

---

## 1. Descripción del Dominio del Negocio

### Propósito del Sistema
El sistema **MTW-PROJECT-Backend** es una **plataforma de gestión de servicios de transporte corporativo** que facilita la administración integral de reservas de movilidad para empresas. El sistema maneja todo el flujo desde la solicitud de transporte hasta la facturación y contabilidad.

### Características Principales
1. **Gestión de Reservas (Bookings):** Control completo del ciclo de vida de reservas de transporte
2. **Gestión de Conductores:** Administración de conductores y sus vehículos
3. **Gestión de Pasajeros:** Registro y seguimiento de pasajeros
4. **Gestión de Empresas:** Manejo de empresas clientes que solicitan servicios
5. **Facturación:** Generación y gestión de facturas
6. **Contabilidad:** Registro contable de las operaciones
7. **Reportes:** Generación de reportes en formato Excel
8. **Autenticación:** Sistema de login para usuarios del sistema

---

## 2. Modelo de Datos

### 2.1 Entidades Principales

#### **Booking (Reserva)**
Entidad central del sistema que representa una reserva de transporte.

**Atributos:**
- `idBooking` (Long): ID único de la reserva
- `date` (Date): Fecha de la reserva
- `time` (Date): Hora de la reserva
- `company` (Company): Empresa que solicita el servicio
- `applicant` (String): Nombre del solicitante
- `area` (Area): Área organizacional del solicitante
- `passenger` (Passenger): Pasajero que viajará
- `pickUp` (String): Dirección de recojo
- `ubigeoPickUp` (Ubigeo): Ubicación geográfica del recojo
- `destination` (String): Dirección de destino
- `ubigeoDestination` (Ubigeo): Ubicación geográfica del destino
- `notes` (String): Notas adicionales
- `currency` (Currency): Moneda del precio
- `price` (String): Precio del servicio
- `driver` (Driver): Conductor asignado
- `status` (String): Estado de la reserva
- `driverPaymentStatus` (Boolean): Estado de pago al conductor
- `clientPaymentStatus` (Boolean): Estado de pago del cliente
- `bill` (Bill): Factura asociada

**Estados de una Reserva:**
1. **"En Reserva"**: Estado inicial cuando no hay conductor asignado
2. **"Pendiente"**: Cuando se asigna un conductor
3. **"En Proceso"**: Cuando el servicio está en ejecución
4. **"Finalizado"**: Cuando el servicio ha sido completado

#### **Driver (Conductor)**
Representa a los conductores que prestan el servicio.

**Atributos:**
- `idDriver` (Long): ID único
- `names` (String): Nombres
- `lastNames` (String): Apellidos
- `idNumber` (String): Número de identificación
- `phone` (String): Teléfono
- `status` (String): Estado del conductor
- **Datos del Vehículo:**
  - `model` (String): Modelo del vehículo
  - `brand` (String): Marca del vehículo
  - `carPlate` (String): Placa del vehículo
  - `year` (String): Año del vehículo
  - `color` (String): Color del vehículo

#### **Company (Empresa)**
Empresas clientes que solicitan servicios de transporte.

**Atributos:**
- `idCompany` (Long): ID único
- `businessName` (String): Razón social (obligatorio)
- `idNumber` (String): RUC (11 dígitos, obligatorio)
- `address` (String): Dirección (obligatorio)
- `tradeName` (String): Nombre comercial
- `phone` (String): Teléfono con validación de formato

#### **Passenger (Pasajero)**
Personas que utilizan el servicio de transporte.

**Atributos:**
- `idPassenger` (Long): ID único
- `names` (String): Nombres
- `lastNames` (String): Apellidos
- `idUbigeo` (Long): Ubicación geográfica
- `status` (String): Estado
- `adress` (String): Dirección [nota: hay un typo en el código]
- `phone` (String): Teléfono

#### **Bill (Factura)**
Documentos de facturación del sistema.

**Atributos:**
- `idBill` (Long): ID único
- `series` (String): Serie de la factura
- `number` (String): Número de factura
- `date` (String): Fecha de emisión
- `subTotal` (String): Subtotal
- `igv` (String): IGV (Impuesto General a las Ventas - Perú)
- `total` (String): Total
- `idCurrency` (Long): Moneda
- `status` (String): Estado ("Activo" / "Inactivo")

#### **Accounting (Contabilidad)**
Registros contables de las operaciones.

**Atributos:**
- `idAccounting` (Long): ID único
- `idBooking` (Long): ID de reserva relacionada
- `date` (String): Fecha
- `time` (String): Hora
- `status` (String): Estado
- `notes` (String): Notas

#### **Area**
Áreas organizacionales de las empresas.

**Atributos:**
- `idArea` (Long): ID único
- `name` (String): Nombre (único)

#### **Ubigeo**
Ubicaciones geográficas (sistema de codificación territorial de Perú).

**Atributos:**
- `idUbigeo` (Long): Código único
- Representa departamentos, provincias y distritos del Perú

#### **Currency (Moneda)**
Tipos de moneda para las transacciones.

**Atributos:**
- `idCurrency` (Long): ID único
- Información de la moneda

#### **Users (Usuarios)**
Usuarios del sistema.

**Atributos:**
- `idUser` (Long): ID único
- `username` (String): Nombre de usuario
- `password` (String): Contraseña
- `status` (String): Estado
- `idUserType` (Long): Tipo de usuario

---

## 3. Lógica de Negocio

### 3.1 Flujo de Reservas (Booking)

#### Creación de Reserva
1. Se crea una nueva reserva con la información del solicitante, pasajero, origen y destino
2. **Si se asigna un conductor inmediatamente:** Estado = "Pendiente"
3. **Si NO se asigna conductor:** Estado = "En Reserva"

#### Asignación de Conductor
- Cuando se asigna un conductor a una reserva en estado "En Reserva", el estado cambia a "Pendiente"

#### Ejecución del Servicio
- El conductor puede cambiar el estado a "En Proceso" cuando comienza el servicio
- Al finalizar, el estado cambia a "Finalizado"

#### Gestión de Pagos
El sistema maneja dos estados de pago independientes:
- **`driverPaymentStatus`**: Indica si se ha pagado al conductor
- **`clientPaymentStatus`**: Indica si el cliente ha pagado

### 3.2 Flujo de Facturación

1. Las facturas se crean con estado "Activo"
2. Se pueden asociar múltiples reservas a una factura
3. Las facturas incluyen:
   - Subtotal
   - IGV (19% generalmente en Perú)
   - Total
4. El sistema implementa **eliminación lógica**: las facturas se marcan como "Inactivo" en lugar de eliminarse

### 3.3 Sistema de Búsqueda y Filtros

El sistema permite filtrar reservas por múltiples parámetros:
- ID de reserva
- Nombre del solicitante
- ID de empresa
- ID de pasajero
- ID de conductor

**Implementación:** Consulta JPQL dinámica que aplica filtros solo si los valores no son nulos o vacíos.

### 3.4 Generación de Reportes

El sistema utiliza **Apache POI** para generar reportes en formato Excel (.xlsx):
- Actualmente implementado para exportar lista de empresas
- El formato incluye:
  - Encabezados formateados
  - Ajuste automático de columnas
  - Datos tabulados

---

## 4. Arquitectura del Sistema

### 4.1 Patrón de Capas

El sistema sigue una **arquitectura en capas (Layered Architecture)**:

```
┌─────────────────────────────────────┐
│         Controllers (REST)          │  ← Capa de Presentación
├─────────────────────────────────────┤
│            Services                 │  ← Capa de Lógica de Negocio
├─────────────────────────────────────┤
│          Repositories               │  ← Capa de Acceso a Datos
├─────────────────────────────────────┤
│         Entities (JPA)              │  ← Capa de Modelo
├─────────────────────────────────────┤
│        MySQL Database               │  ← Capa de Persistencia
└─────────────────────────────────────┘
```

### 4.2 Componentes

#### Controllers (Controladores REST)
- `BookingController`: Gestión de reservas (CRUD completo + endpoints especializados)
- `DriverController`: Gestión de conductores
- `PassengerController`: Gestión de pasajeros
- `CompanyController`: Gestión de empresas
- `BillController`: Gestión de facturas
- `AccountingController`: Gestión contable
- `AreaController`: Gestión de áreas
- `CurrencyController`: Gestión de monedas
- `UbigeoController`: Gestión de ubicaciones
- `AuthController`: Autenticación
- `UsersController`: Gestión de usuarios
- `ReportController`: Generación de reportes

**Características Comunes:**
- Prefijo `@CrossOrigin(origins = "http://localhost:4200")` para CORS
- Respuestas estandarizadas con estructura:
  ```json
  {
    "status": "200",
    "message": "Mensaje descriptivo",
    "data": {...}
  }
  ```

#### Services (Servicios)
Implementan la lógica de negocio:
- Interfaces de servicio (ej: `BookingService`)
- Implementaciones (ej: `BookingServiceImpl`)
- **Uso de constantes** para estados y valores fijos
- **Transacciones** donde corresponde (`@Transactional`)

#### Repositories (Repositorios)
Heredan de `CrudRepository<T, ID>`:
- Métodos CRUD estándar
- **Consultas personalizadas** con `@Query` (JPQL)
- Métodos de búsqueda derivados (ej: `findByCompany`)

---

## 5. Características Técnicas

### 5.1 Tecnologías y Dependencias

**Core:**
- Spring Boot 3.1.6
- Java 17
- Spring Data JPA
- Spring Web
- Spring Validation

**Base de Datos:**
- MySQL con conector `mysql-connector-j`
- Hibernate como ORM
- Dialecto: `MySQLDialect`
- Estrategia de nombres: `PhysicalNamingStrategyStandardImpl` (mantiene nombres originales)

**Utilidades:**
- **Lombok**: Reduce boilerplate con `@Getter`, `@Setter`, `@NoArgsConstructor`, `@AllArgsConstructor`
- **Apache POI**: Generación de archivos Excel
- **Spring Boot DevTools**: Recarga en caliente durante desarrollo

**Documentación:**
- **SpringDoc OpenAPI**: Swagger UI integrado
- Accesible en `/swagger-ui.html`

### 5.2 Configuración de JPA

```properties
spring.jpa.show-sql=true
logging.level.org.hibernate.SQL=debug
```

- SQL visible en logs para debugging
- **DDL auto deshabilitado** (`spring.jpa.hibernate.ddl-auto` comentado) → La BD se gestiona manualmente

### 5.3 Manejo de Fechas

- Formato de fecha: `yyyy-MM-dd`
- Formato de hora: `HH:mm`
- Timezone: `America/Lima` (Perú)
- Uso de `@Temporal`, `@DateTimeFormat`, y `@JsonFormat`

### 5.4 Relaciones JPA

**Estrategia de Carga:**
- Uso extensivo de `LAZY` fetching
- `@JsonIgnoreProperties` para evitar serialización de proxies de Hibernate

**Tipos de Relaciones:**
- `@ManyToOne`: Booking → Company, Booking → Driver, etc.
- `@OneToOne`: Booking → Passenger, Booking → Area

---

## 6. Observaciones y Áreas de Mejora

### 6.1 Seguridad
- **Contraseñas en texto plano**: No hay encriptación de contraseñas
- **Sin JWT o Spring Security**: El sistema de autenticación es básico
- **Credenciales en properties**: Usuario y contraseña de BD en archivo de configuración

**Recomendación:** Implementar Spring Security con BCrypt y JWT

### 6.2 Validación
- Validaciones limitadas (solo algunas en `Company`)
- Falta validación de reglas de negocio complejas

**Recomendación:** Implementar validaciones con Bean Validation (`@Valid`, `@NotNull`, etc.)

### 6.3 Manejo de Errores
- No hay manejo centralizado de excepciones
- Los controladores manejan excepciones genéricas

**Recomendación:** Implementar `@ControllerAdvice` para manejo global

### 6.4 Tipos de Datos
- Uso de `String` para campos numéricos (`price`, `subTotal`, `igv`, `total`)
- Posible pérdida de precisión en cálculos

**Recomendación:** Usar `BigDecimal` para valores monetarios

### 6.5 Nomenclatura
- Typo en `Passenger.adress` (debería ser `address`)
- Inconsistencia en nombres de servicios (`CurrencyServiveImpl` vs `CurrencyServiceImpl`)

### 6.6 Testing
- Solo existe el archivo de test por defecto
- **No hay tests unitarios ni de integración**

**Recomendación:** Implementar suite de tests con JUnit 5 y Mockito

### 6.7 Documentación
- README muy básico
- Falta documentación de API endpoints
- No hay diagramas de arquitectura

---

## 7. Modelo Entidad-Relación Conceptual

```
┌─────────────┐       ┌──────────────┐
│   Company   │──────<│   Booking    │
└─────────────┘       └──────────────┘
                             │
                             ├──────> ┌────────────┐
                             │        │  Passenger │
                             │        └────────────┘
                             │
                             ├──────> ┌────────────┐
                             │        │   Driver   │
                             │        └────────────┘
                             │
                             ├──────> ┌────────────┐
                             │        │    Bill    │
                             │        └────────────┘
                             │
                             ├──────> ┌────────────┐
                             │        │    Area    │
                             │        └────────────┘
                             │
                             └──────> ┌────────────┐
                                      │   Ubigeo   │
                                      └────────────┘
```

---

## 8. API Endpoints Principales

### Booking (Reservas)
- `GET /booking` - Listar todas las reservas
- `GET /booking/{id}` - Obtener reserva por ID
- `POST /booking` - Crear nueva reserva
- `PUT /booking` - Actualizar reserva
- `DELETE /booking/{id}` - Eliminar reserva
- `GET /booking/filterBookingsByParams` - Filtrar reservas
- `PUT /booking/finalize` - Finalizar reserva
- `PUT /booking/inProcess` - Marcar reserva en proceso
- `PUT /booking/updateDriverPaymentStatus` - Actualizar pago conductor
- `PUT /booking/updateClientPaymentStatus` - Actualizar pago cliente

### Auth
- `POST /auth/login` - Autenticación de usuario

### Reports
- Endpoint para generar Excel de empresas

---

## 9. Contexto de Implementación

### Mercado Objetivo
El sistema está orientado al **mercado peruano**, evidenciado por:
- Uso de **Ubigeo** (codificación territorial de Perú)
- Campo **RUC** de 11 dígitos (identificador tributario peruano)
- Cálculo de **IGV** (impuesto peruano)
- Timezone `America/Lima`

### Tipo de Clientes
- **Empresas corporativas** que requieren servicios de movilidad para sus empleados
- **Organizaciones con múltiples áreas** que necesitan transporte frecuente

---

## 10. Conclusiones

### Lo Que Funciona Bien
1. **Arquitectura clara y mantenible** con separación de capas
2. **Modelo de dominio rico** que refleja bien el negocio
3. **Gestión de estados** de reservas bien definida
4. **Integración con Swagger** para documentación automática
5. **Uso de Lombok** para reducir código boilerplate
6. **Sistema de filtros flexible** para consultas

### Áreas Críticas que Requieren Atención
1. **Seguridad**: Implementación urgente de autenticación robusta
2. **Validación de Datos**: Fortalecer validaciones de entrada
3. **Tipos de Datos**: Corregir uso de String para valores numéricos
4. **Testing**: Desarrollar suite de tests
5. **Manejo de Errores**: Centralizar gestión de excepciones

### Propósito Final del Sistema
El sistema MTW-PROJECT-Backend busca ser una **plataforma integral de gestión de movilidad corporativa** que:
- Optimice la asignación de conductores a solicitudes de transporte
- Facilite el seguimiento del ciclo completo de reservas
- Simplifique la facturación y contabilidad de servicios
- Proporcione reportes para análisis de operaciones
- Mejore la experiencia de empresas, pasajeros y conductores

---

**Fecha de Análisis:** 2025-11-18
**Versión del Sistema:** 0.0.1-SNAPSHOT
**Analizado por:** Claude (IA)
