# PharmaSoft Frontend

Aplicación Angular servida como SPA por Nginx. El informe de optimización de la sesión 13 se entrega junto al repositorio del backend.

## Despliegue en producción

### Arquitectura

```text
Navegador -- HTTPS :443 --> Nginx :80/:8081 -- /api --> PharmaBackend :8080
                                                        |--> Oracle :1521
                                                        `--> Redis :6379
```

Para desarrollo local, la SPA se publica en `http://localhost:8081`; Nginx reenvía `/api/` al backend en el puerto 8080. Oracle y Redis son servicios internos del backend; la instancia local de Oracle usa el puerto publicado 1522 y Redis el 6379.

### Configuración

| Configuración | Propósito |
| --- | --- |
| `src/environments/environment.ts` | URL del API en desarrollo |
| `src/environments/environment.development.ts` | URL del API para `ng serve` |
| `src/environments/environment.ts` | URL relativa `/api/v1` detrás de Nginx en producción |
| `FRONTEND_ORIGINS` (backend) | Origen permitido por CORS si frontend y API se publican separados |

La URL del API se sustituye durante la compilación Angular. No incrustar tokens ni secretos en los archivos de entorno.

### Construcción y despliegue

1. Instalar dependencias con `pnpm install --frozen-lockfile`.
2. Ejecutar `pnpm exec ng build --configuration production` y revisar los budgets.
3. Publicar el contenido de `dist/pharma-frontend/browser/` en la raíz de Nginx; conservar `deploy/nginx.conf` para los encabezados de caché y el proxy `/api/`.
4. Comprobar `/`, `/login`, navegación directa a una ruta interna y llamadas al API. El `index.html` debe revalidarse; los JS/CSS con hash pueden conservar caché larga e inmutable.
5. Para volver atrás, desplegar el directorio de artefactos de la versión anterior y recargar Nginx. Mantener juntos `index.html` y todos los recursos con hash de esa versión.

### Estado de prácticas

| Práctica | Estado | Acción pendiente |
| --- | --- | --- |
| Rutas lazy y chunks JS/CSS con hash | Cumple | Revisar tamaño en cada build. |
| Reporte del inicio diferido y pantallas de ventas separadas | Cumple | Comparar transferencia real con DevTools en cada cambio de estrategia. |
| `index.html` revalidado y JS/CSS cacheados como inmutables | Cumple | Confirmar también origen memoria/disco/304 en el navegador. |
| Respuestas API sin caché de navegador | Cumple con los encabezados predeterminados de Spring Security | Mantener el proxy sin sobrescribir esos encabezados. |
| Compresión y HTTPS de producción | Parcial | Habilitar TLS y compresión en el proxy que publique el servicio. |

### Recomendaciones priorizadas para tres locales

1. **Prioridad alta:** terminar TLS/HTTPS en el proxy y redirigir HTTP a HTTPS.
2. **Prioridad alta:** versionar y conservar artefactos del frontend para poder volver atrás sin mezclar archivos hash de distintas versiones.
3. **Prioridad media:** servir la misma versión validada del frontend desde los tres locales y monitorear el backend compartido y Redis.
