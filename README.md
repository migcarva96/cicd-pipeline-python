# cicd-pipeline-python
Qué ventajas le proporciona a un proyecto el uso de un pipeline de CI? Menciona al menos tres ventajas específicas y explica por qué son importantes

1. Detección temprana de errores
   Cada cambio dispara compilaciones y pruebas automáticas, por lo que los fallos aparecen justo después del commit, no semanas después.
2. Mejor colaboración y visibilidad
   Toda la validación pasa por el mismo pipeline, lo que da un punto único de verdad sobre el estado del proyecto
3. Menor error humano
   Con despliegues rutinarios y repetibles, baja el riesgo de caídas en producción y es más sencillo hacer rollback rápido si algo falla,          reduciendo el tiempo medio de recuperación.

¿Cuál es la diferencia principal entre una prueba unitaria y una prueba de aceptación? Da un ejemplo de algo que probarías con una prueba unitaria y algo que probarías con una prueba de aceptación (en el contexto de cualquier aplicación que conozcas, descríbela primero).

Prueba unitaria
Prueba una unidad mínima de código (función, método, clase) de forma aislada, verificando su lógica con entradas y salidas controladas
La “unitaria” prueba solo una función, aislada, sin red, sin base de datos ni otros servicios.

Supóngamos que se tiene una función interna (Uber):

calcularTarifaEstimadayTiempo(distanciaKm, duracionMin, factorDemanda, tarifaBase)

Lo esperado para esta función en una prueba unitaria:

Scenario: Cálculo correcto de tarifa estimada y tiempo
  Given una distancia de 10 km
  And una duración estimada de 20 minutos
  And un factor de demanda de 1.5
  And una tarifa base de 5000
  When calculo la tarifa estimada y el tiempo
  Then la tarifa estimada debe ser la esperada según la fórmula de negocio
  And el tiempo estimado debe ser 20 minutos

Prueba de aceptación.

Valida el sistema completo desde la perspectiva del usuario o negocio, comprobando que cumple los criterios de aceptación de una historia o requisito

Feature: Solicitud de viaje en la app tipo Uber
  Como usuario pasajero
  Quiero solicitar un viaje desde un origen a un destino
  Para que un conductor me recoja y me lleve a mi destino

  Background:
    Given que tengo una cuenta activa
    And tengo un método de pago válido registrado
    And estoy en una zona con conductores disponibles

  Scenario: Solicitar un viaje con conductores disponibles
    Given que abro la app de viajes
    And ingreso mi ubicación actual como origen
    And ingreso una dirección válida como destino
    When confirmo la solicitud de viaje
    Then debo ver una tarifa estimada para ese trayecto
    And debo ver un tiempo estimado de llegada del conductor
    And debo ver un conductor asignado con su nombre, placa y calificación
    And el estado de mi viaje debe ser "Conductor en camino"

  Scenario: Solicitar un viaje sin conductores disponibles
    Given que abro la app de viajes
    And ingreso mi ubicación actual como origen
    And ingreso una dirección válida como destino
    And no hay conductores disponibles cerca de mi ubicación
    When intento confirmar la solicitud de viaje
    Then debo ver un mensaje indicando que no hay autos disponibles
    And no se debe crear ningún viaje en el sistema


Describe brevemente qué hace cada uno de los steps principales de tu workflow de GitHub Actions (desde el checkout hasta el push de Docker). Explica el propósito de cada uno (qué hace y para qué se hace).

 name: Set up QEMU
        if: github.event_name == 'push' && github.ref == 'refs/heads/main'
        uses: docker/setup-qemu-action@v3

Este step prepara QEMU en el runner, pero solo cuando el workflow se ha disparado por un push directo a main, para que no se ejecute innecesariamente en otras ramas o eventos.



name: Set up Docker Buildx
        if: github.event_name == 'push' && github.ref == 'refs/heads/main'
        uses: docker/setup-buildx-action@v3


Este paso prepara Docker Buildx en el runner para que los pasos siguientes del workflow puedan construir imágenes Docker usando BuildKit (incluyendo multi‑arquitectura, cache, etc.), pero solo cuando el workflow se ejecuta por un push a la rama main



name: Login to Docker Hub
        if: github.event_name == 'push' && github.ref == 'refs/heads/main'
        uses: docker/login-action@v3
        with:
          username: ${{ vars.DOCKERHUB_USERNAME }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}

Este paso inicia sesión en Docker Hub dentro del runner de GitHub Actions usando el usuario y token configurados en variables/secretos, pero solo cuando el workflow se ejecuta por un push a la rama main


name: Build and push Docker image
        if: github.event_name == 'push' && github.ref == 'refs/heads/main'
        uses: docker/build-push-action@v5
        with:
          context: .
          file: ./Dockerfile
          push: true
          tags: |
            ${{ vars.DOCKERHUB_USERNAME }}/${{ github.event.repository.name }}:${{ github.sha }}
            ${{ vars.DOCKERHUB_USERNAME }}/${{ github.event.repository.name }}:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max

Este paso construye la imagen Docker usando el Dockerfile del repositorio y lo push a Docker Hub (con tag por sha y tag latest), utilizando caché de GitHub Actions, pero solo cuando el workflow se ejecuta por un push a la rama main.

¿Qué problemas o dificultades encontraste al implementar este taller? ¿Cómo los solucionaste? (Si no encontraste ningún problema, describe algo nuevo que hayas aprendido).

Se comprueba que cuando se tiene el CI esta automatizado se detectan errores rápido y se reduce el riesgo y el costo de integrar cambios, haciendo el desarrollo más rápido y confiable



¿Qué ventajas ofrece empaquetar la aplicación en una imagen Docker al final del pipeline en lugar de simplemente validar el código?

Empaquetar en Docker al final del pipeline asegura un artefacto listo para desplegar, portable y reproducible, en vez de quedarte solo con la garantía de que “el código compila y pasa pruebas