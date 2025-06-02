<!-- <p align="center">
<img src="/src/frontend/static/icons/Hipster_HeroLogoMaroon.svg" width="300" alt="Online Boutique" />
</p> -->

**¿Qué es Online Boutique?** Online Boutique es una aplicación de demostración de microservicios diseñada con un enfoque cloud-first, desarrollada por Google.

Se trata de una tienda e-commerce donde los usuarios pueden navegar productos, agregarlos al carrito y realizar compras. Esta aplicación sirve como ejemplo completo para demostrar cómo modernizar aplicaciones empresariales utilizando tecnologías y servicios de Google Cloud.
Este repositorio está basado en la plantilla oficial de Google Cloud Platform (GCP), y fue adaptado en el contexto de la actividad integradora académica. Nuestro objetivo es cumplir con los requerimientos del proyecto.

## Arquitectura

**Online Boutique** está compuesta por 11 microservicios escritos en distintos lenguajes de programación, que se comunican entre sí mediante gRPC.

[![Architecture of
microservices](/docs/img/architecture-diagram.png)](/docs/img/architecture-diagram.png)

Find **Protocol Buffers Descriptions** at the [`./protos` directory](/protos).

| Servicio                                              | Lenguaje      | Descripcion                                                                                                                       |
| ---------------------------------------------------- | ------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| [frontend](/src/frontend)                           | Go            | Expone un servidor HTTP que muestra el sitio web. No requiere registro/inicio de sesión y genera IDs de sesión automáticamente. |
| [cartservice](/src/cartservice)                     | C#            | Guarda los productos del carrito de compras del usuario en Redis y los recupera.                                                           |
| [productcatalogservice](/src/productcatalogservice) | Go            | Proporciona una lista de productos desde un archivo JSON. Permite buscar productos y obtener su detalle.                        |
| [currencyservice](/src/currencyservice)             | Node.js       | Convierte montos de dinero entre distintas monedas. Usa valores reales del Banco Central Europeo. Es el servicio con más QPS. |
| [paymentservice](/src/paymentservice)               | Node.js       | Simula el cobro con tarjeta de crédito y devuelve un ID de transacción.                                     |
| [shippingservice](/src/shippingservice)             | Go            | Calcula estimaciones de costos de envío según el carrito y simula el envío a una dirección.                                 |
| [emailservice](/src/emailservice)                   | Python        | Envía por correo electrónico una confirmación del pedido al usuario (simulado).                                                                                   |
| [checkoutservice](/src/checkoutservice)             | Go            | Recupera el carrito del usuario, arma la orden y coordina el pago, el envío y el correo de confirmación.                            |
| [recommendationservice](/src/recommendationservice) | Python        | Recomienda otros productos en base a los que hay en el carrito.                                                                      |
| [adservice](/src/adservice)                         | Java          | Proporciona anuncios de texto según palabras clave del contexto.                                                                                   |
| [loadgenerator](/src/loadgenerator)                 | Genera carga automáticamente enviando peticiones que simulan flujos reales de usuarios en la tienda.                                              |

## Screenshots

| Home Page                                                                                                         | Checkout Screen                                                                                                    |
| ----------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------ |
| [![Screenshot of store homepage](/docs/img/online-boutique-frontend-1.png)](/docs/img/online-boutique-frontend-1.png) | [![Screenshot of checkout screen](/docs/img/online-boutique-frontend-2.png)](/docs/img/online-boutique-frontend-2.png) |

## Guía rápida (en GKE)

1. Asegurate de tener los siguientes requisitos:
   - [Un proyecto en Google Cloud](https://cloud.google.com/resource-manager/docs/creating-managing-projects#creating_a_project).
   - Un entorno de consola con `gcloud`, `git`, and `kubectl` instalados.

2. Cloná la última versión del proyecto adaptado.

   ```sh
   git clone https://github.com/notsssofi/online-boutique-integrador.git
   cd online-boutique-integrador
   ```

3. Configurá el proyecto de Google Cloud y la región. Asegurate de habilitar la API de Google Kubernetes Engine.

   ```sh
   export PROJECT_ID=<PROJECT_ID>
   export REGION=us-central1
   gcloud services enable container.googleapis.com \
     --project=${PROJECT_ID}
   ```

   Reemplazá `<PROJECT_ID>` con el ID real de tu proyecto en Google Cloud.

4. Creá el clúster de GKE y obtené las credenciales para conectarte desde tu entorno local.

   ```sh
   gcloud container clusters create-auto online-boutique \
     --project=${PROJECT_ID} --region=${REGION}
   ```

   Este paso puede tardar unos minutos.

5. Desplegá Online Boutique dentro del clúster en el entorno de producción (prod).

   ```sh
   kubectl create ns prod
   kubectl apply -f namespaces/prod/kubernetes-manifests.yaml -n prod
   ```

6. Esperá a que los pods estén en estado Running.

   ```sh
   kubectl get pods -n prod
   ```

   Deberías ver una salida como la siguiente después de unos minutos:

   ```
   NAME                                     READY   STATUS    RESTARTS   AGE
   adservice-76bdd69666-ckc5j               1/1     Running   0          2m58s
   cartservice-66d497c6b7-dp5jr             1/1     Running   0          2m59s
   checkoutservice-666c784bd6-4jd22         1/1     Running   0          3m1s
   currencyservice-5d5d496984-4jmd7         1/1     Running   0          2m59s
   emailservice-667457d9d6-75jcq            1/1     Running   0          3m2s
   frontend-6b8d69b9fb-wjqdg                1/1     Running   0          3m1s
   loadgenerator-665b5cd444-gwqdq           1/1     Running   0          3m
   paymentservice-68596d6dd6-bf6bv          1/1     Running   0          3m
   productcatalogservice-557d474574-888kr   1/1     Running   0          3m
   recommendationservice-69c56b74d4-7z8r5   1/1     Running   0          3m1s
   redis-cart-5f59546cdd-5jnqf              1/1     Running   0          2m58s
   shippingservice-6ccc89f8fd-v686r         1/1     Running   0          2m58s
   ```

7. Accedé al frontend web desde un navegador utilizando la IP externa del servicio.

   ```sh
   kubectl get service frontend-external -n prod | awk '{print $4}'
   ```

   Abrí `http://EXTERNAL_IP` en tu navegador para ver la tienda funcionando.

8. ¡Listo! Ya tenés Online Boutique desplegado y funcionando en un entorno real en Google Cloud Kubernetes Engine.

9. Si ya no lo necesitás, podés eliminar el clúster:

   ```sh
   gcloud container clusters delete online-boutique \
     --project=${PROJECT_ID} --region=${REGION}
   ```

   Este proceso también puede tardar algunos minutos.
