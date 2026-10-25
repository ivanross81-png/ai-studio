# Evaluación de LTX Desktop

Repositorio: https://github.com/Lightricks/LTX-Desktop/
Documentación: https://github.com/Lightricks/LTX-Desktop/blob/main/docs/gen-space-and-video-editor.md

## Hipótesis
Usarlo como referencia para editor, timeline y exportación FFmpeg; no asumir integración directa en web.

## Auditar
- Licencia de código y avisos de dependencias.
- Modelo de datos de proyecto, secuencias, pistas, clips y metadatos.
- Acoplamiento UI con Electron/IPC, filesystem y procesos locales.
- Backend FastAPI local y exportación FFmpeg.
- Funciones disponibles por API frente a funciones solo locales.
- Licencias separadas de pesos, modelos, outputs y API.

## Decisión
Reutilizar solo componentes desacoplados, mantenibles y con licencia compatible. Si timeline depende de IPC o rutas locales, implementar compositor web propio y usar FFmpeg en un worker de servidor.

## Checklist
- [ ] Fijar commit/tag auditado.
- [ ] Inventariar carpetas, dependencias y licencias.
- [ ] Diagramar UI -> API -> filesystem/modelos.
- [ ] Probar exportación en entorno aislado.
- [ ] Documentar modelo de timeline.
- [ ] Prototipar editor web.
- [ ] Registrar conclusión en ADR.

No inferir permiso comercial por la licencia de la aplicación: código, pesos y servicios pueden tener términos diferentes. Obtener revisión legal antes del lanzamiento.