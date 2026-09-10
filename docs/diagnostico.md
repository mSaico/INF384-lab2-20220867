Los 4 defectos encontrados estan en el archivo pipeline.yml y son: 
1. Instalación mutable de dependencias: Se instalan las dependencias usando requirements.txt, el cual resuelve versiones sobre la marcha, en lugar de usar requirements.lock
- Línea: pip install -r requirements.txt de ambos jobs
- Consecuencia: El pipeline podría instalar una versión distinta a la probada localmente
2. Ausencia de caché de dependencias: Los entornos virtuales se reconstruyen desde cero descargando todo por internet en cada ejecución, sin guardar los paquetes en caché.
- Línea: uses: actions/setup-python@v5
- Consecuencia: Se genera una relantización innecesaria, consumiendo más tiempo y recursos de los necesarios.
3. Falso positivo de calidad: El escaneo de SonarCloud es asíncrono y no detiene el pipeline si el código incumple las políticas establecidas por el quality gate.
- Linea: uses: SonarSource/sonarqube-scan-action@v8
- Consecuencia: Código con baja cobertura o vulnerabilidades se marca como exitoso y avanza por el pipeline.
4. Publicación incondicional y sin trazabilidad: El artefacto se publica en paralelo sin esperar los resultados de las pruebas, y se empaqueta con un nombre genérico en lugar de reflejar la versión.
- Línea: En la definición del job publicar: y en el paso de publicacion del paquete.
-Consecuencia: Se pueden publicar artefactos rotos al no depender de la validación, y es imposible identificar qué versión de código contiene un artefacto descargado.

Defecto que explica la duración: El defecto que explica la duración es el defecto 2 de la Ausencia de caché. En la línea base se registró una duración de aproximadamente 60 segundos y esto se debe a que descargar 
  e instalar pip y todas las dependencias desde cero abarca más de la mitad de ese tiempo.

El vínculo con su caso: El defecto 3 ataca directamente nuestro caso 03. Al no detener el código defectuoso de forma automatizada,los errores avanzan hacia las etapas manuales de revisión y pruebas. 
  Esto impacta el rendimiento acumulado del flujo, obligando a las tareas a retroceder constantemente y generando la enorme brecha de tiempo de retrabajo (desperdicio) que detectamos en nuestro 
  Value Stream Map entre el escenario esperado y el real.

La métrica DORA: Las dos métricas alcanzables sin despliegue son: Lead Time para Cambios y Tasa de fallos en cambios. Elegí Lead Time para Cambios ya que al habilitar el caché y configurar correctamente los pull requests, 
  se acelera la retroalimentación que recibe el desarrollador, disminuyendo el tiempo desde el primer commit hasta la integración limpia.

El proxy: El número concreto que se va a medir es la duración total de ejecución del pipeline en GitHub Actions (medido en segundos).
  
