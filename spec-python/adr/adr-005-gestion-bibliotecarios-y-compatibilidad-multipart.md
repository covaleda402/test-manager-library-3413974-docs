# ADR-005 · Gestión de bibliotecarios y compatibilidad multipart

- **Estado:** aceptada
- **Contexto:**
  1. El spec original contemplaba únicamente `POST /users` (alta de bibliotecarios). Durante la operación de la biblioteca, el administrador necesita listar los bibliotecarios existentes y poder darlos de baja de forma segura.
  2. Al eliminar un bibliotecario, la tabla `library.loan` tiene una llave foránea restrictiva `loaned_by_user_id REFERENCES library.user(id) ON DELETE RESTRICT` (FK-5).
  3. Los navegadores y clientes HTTP al subir binarios mediante `multipart/form-data` con métodos `PUT` experimentan incompatibilidades en servidores PHP/FastCGI donde el stream no rellena nativamente `$_FILES`.
- **Decisión:**
  1. Habilitar `GET /users` (listar) y `DELETE /users/{id}` (dar de baja).
  2. En `DELETE /users/{id}`, validar que el usuario no sea administrador (403 `FORBIDDEN_ROLE`). Verificar en la capa de aplicación si el bibliotecario posee préstamos históricos registrados; si tiene préstamos, rechazar la eliminación con 409 `FORBIDDEN_OPERATION` informando el motivo para salvaguardar la integridad referencial y trazabilidad histórica.
  3. Para la subida de portadas, aceptar en la ruta tanto `PUT` como `POST` con emulación de método (`_method: 'PUT'` y cabecera `X-HTTP-Method-Override: 'PUT'`), garantizando compatibilidad total entre clientes y el backend.
- **Alternativas descartadas:**
  - Borrado en cascada (`ON DELETE CASCADE`) en préstamos: descartado porque destruiría el historial de auditoría de la biblioteca (Art. VIII y Constitución).
  - Dejar a los administradores a ciegas sin ver ni gestionar los bibliotecarios creados.
- **Consecuencias:**
  - La interfaz de usuario cuenta con un panel completo de administración de bibliotecarios con confirmación visual.
  - La subida de imágenes de portadas funciona de manera transparente y consistente en cualquier entorno.
- **Cómo se verifica:**
  - `GET /users` lista los bibliotecarios.
  - `DELETE /users/{id}` elimina usuarios sin préstamos y rechaza con 409 a usuarios con préstamos registrados.
  - Subir una imagen de portada funciona con éxito devolviendo `200 OK` con su respectiva `cover_key`.
