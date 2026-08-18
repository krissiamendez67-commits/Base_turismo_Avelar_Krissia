# Accommodations Tourism – Base de Datos y Consultas SQL

Script de restauración y consultas SQL para la base de datos **`accommodations_tourism`**, un sistema de gestión de alojamientos turísticos (hoteles, hostales, apartamentos, villas, cabañas, resorts, etc.) que administra propietarios, ubicaciones, habitaciones, reservas, huéspedes, pagos y reseñas.

## Origen del script

Extraído de un dump binario de PostgreSQL (formato *custom*, v1.16, generado con PostgreSQL 18.3) y compatible con **PostgreSQL 14+**.

## Requisitos previos

- PostgreSQL 14 o superior
- Cliente `psql`

## Cómo restaurar la base de datos

```bash
# 1. Crear la base de datos
psql -U postgres -c "CREATE DATABASE accommodations_tourism
  WITH TEMPLATE=template0 ENCODING='UTF8'
  LOCALE_PROVIDER=libc LOCALE='en_US.UTF-8';"

# 2. Restaurar el esquema y los datos
psql -U postgres -d accommodations_tourism -f accommodation_database_restore.sql
```

## Estructura del script

El archivo `.sql` está organizado en las siguientes secciones:

1. **Schema** – esquema `public`
2. **Funciones** – `set_updated_at()`, trigger genérico para actualizar la columna `updated_at`
3. **Secuencias** – una secuencia `*_id_seq` por cada tabla con clave primaria autoincremental
4. **Tablas** – definición de las 12 tablas del modelo (ver abajo)
5. **Defaults de secuencia** – asignación de `nextval()` a cada columna `id`
6. **Secuencias OWNED BY** – vínculo de cada secuencia con su columna
7. **Datos** – sentencias `INSERT` con datos de ejemplo/carga inicial
8. **Restricciones (Foreign Keys)** – relaciones entre tablas
9. **Consultas de práctica** – 20 consultas SQL (INSERT, SELECT, UPDATE, DELETE, JOIN, agregaciones, subconsultas)

## Modelo de datos

| Tabla | Descripción |
|---|---|
| `owners` | Propietarios de los alojamientos |
| `locations` | Ubicaciones geográficas (país, ciudad, coordenadas) |
| `accommodation_types` | Tipos de alojamiento (Hotel, Hostal, Apartamento, Casa, Villa, Cabaña, Resort, Guesthouse) |
| `accommodations` | Alojamientos publicados (precio, capacidad, habitaciones, baños, horarios de check-in/out) |
| `amenities` | Catálogo de comodidades (WiFi, Piscina, Parqueo, A/C, Cocina, etc.) |
| `accommodation_amenities` | Tabla intermedia N:M entre alojamientos y comodidades |
| `rooms` | Habitaciones individuales dentro de un alojamiento |
| `guests` | Huéspedes registrados |
| `booking_statuses` | Estados de reserva (Pending, Confirmed, CheckedIn, CheckedOut, Cancelled, NoShow) |
| `bookings` | Reservas (fechas, huéspedes, montos, estado) — incluye columna generada `total_nights` |
| `booking_guests` | Huéspedes adicionales asociados a una reserva |
| `payments` | Pagos asociados a cada reserva |
| `reviews` | Reseñas de los alojamientos (calificación 1–5, título, texto) |
| `staff_users` | Usuarios/personal administrativo del sistema |

### Relaciones principales (Foreign Keys)

- `accommodations.owner_id` → `owners.owner_id`
- `accommodations.accommodation_type_id` → `accommodation_types.accommodation_type_id`
- `accommodations.location_id` → `locations.location_id`
- `rooms.accommodation_id` → `accommodations.accommodation_id`
- `bookings.guest_id` → `guests.guest_id`
- `bookings.accommodation_id` → `accommodations.accommodation_id`
- `bookings.room_id` → `rooms.room_id`
- `bookings.booking_status_id` → `booking_statuses.booking_status_id`
- `booking_guests.booking_id` → `bookings.booking_id`
- `payments.booking_id` → `bookings.booking_id`
- `reviews.booking_id` / `reviews.guest_id` / `reviews.accommodation_id` → `bookings` / `guests` / `accommodations`
- `accommodation_amenities` (tabla N:M) → `accommodations` y `amenities`

## Consultas incluidas

El script incluye 20 consultas de práctica agrupadas por tipo de operación:

| # | Tipo | Descripción |
|---|---|---|
| 1 | `INSERT` | Insertar propietario |
| 2 | `INSERT` | Insertar alojamiento |
| 3 | `INSERT` | Insertar huésped de reserva |
| 4 | `INSERT` | Insertar reserva (pago) |
| 5 | `SELECT` | Alojamientos activos |
| 6 | `SELECT` | Huéspedes filtrados por país |
| 7 | `SELECT` | Reservas filtradas por rango de fechas |
| 8 | `UPDATE` | Actualizar precio de un alojamiento |
| 9 | `UPDATE` | Actualizar estado de una reserva |
| 10 | `DELETE` | Eliminar una reseña |
| 11 | `JOIN` | Reservas + huésped |
| 12 | `JOIN` | Detalle completo de alojamiento + tipo |
| 13 | `JOIN` | Pagos + reserva |
| 14 | `LEFT JOIN` | Alojamientos sin reseñas |
| 15 | `LEFT JOIN` | Alojamientos sin reservas |
| 16 | `AGG` | Total de ingresos (`SUM`) |
| 17 | `AGG` | Promedio de calificación (`AVG`) |
| 18 | `AGG` | Top 5 alojamientos con más reservas |
| 19 | `HAVING` | Alojamientos con más de 3 reservas |
| 20 | Subconsulta | Alojamiento con el precio máximo |

> ⚠️ **Nota:** algunas de estas consultas de práctica contienen inconsistencias de datos de ejemplo (p. ej. `currency_code` con el valor `'USB'` en lugar de un código de moneda válido de 3 letras, o `is_active` con el valor `'F'`), por lo que conviene revisarlas antes de ejecutarlas en un entorno real.

## Autor

Consulta preparada por **Avelar, Krissia**.