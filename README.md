# [GUIA] Integración básica Amplify SDK

En esta guía vamos a aprender como integrar Amplify SDK a un proyecto en Next JS desde cero.

```
Si utilizan otra tecnología o framework algunas cosas de esta guía pueden no funcionar o requerir adaptación
```

## 1. Creación de cuenta

Para poder utilizar Amplify SDK es requisito indispensable el tener una cuenta dentro de la plataforma ya que vamos a utilizar una serie de claves que se generan dentro de la misma. Las cuentas en la plataforma de Amplify son gratuitas y pueden ser creadas desde https://admin.getamplify.app/registro

## 2. Configuración de cuenta

Luego de crear tu cuenta, es necesario configurarla para acceder a todas las funcionalidades. 
Para hacerlo podes ingresar https://admin.getamplify.app/configuracion en donde vas a encontrar que existen dos tipos de cuentas (tienda única y plataforma) las cuales están pensadas para distintos modelos de negocio

```
IMPORTANTE: Esta elección queda asociada a la cuenta y es permanente.
```

### Tienda única

Elige esta opción si tu negocio se basa en la venta de bienes físicos o digitales. Ejemplo: Venta de cursos o indumentaria.

### Plataforma

Elige esta opción si tu negocio es una plataforma que cobra una comisión por el uso de la misma. Ejemplo: Cafecito, Uber, Rappi.

![image](https://github.com/Fundit-Finance/sdk-self-with-next/assets/4969737/ae429f82-6e14-44d3-a365-a495878a862b)

```
El código de esta guía utiliza como tipo de cuenta "Tienda única"
```

## 3. Generando nuestras API Keys

Para poder acceder a información sensible de tu cuentas vas a necesitar autenticarte. Amplify utiliza un sistema de API Keys el cual se tienen que enviar en cada request desde un entorno seguro (servidor).

Podes generar tus API Keys en https://admin.getamplify.app/api-keys.

```
IMPORTANTE: Las API Keys generadas solo son validas en el ambiente en donde fueron creadas por lo que vas a tener que generar una para prod y otra para dev.
```

Podes cambiar de ambiente utilizando el switch que esta al lado de tu nombre en la barra de navegación superior

![image](https://github.com/Fundit-Finance/sdk-self-with-next/assets/4969737/404fc829-8175-4dab-9b84-c873a7f09eac)

Una vez que tengas tu API Key guardala en un lugar seguro. La misma solo se muestra al momento de la creación y no se puede recuperar (el equipo de Amplify tampoco puede hacerlo). Por lo que si pierdes tu API Key vas a tener que crear una nueva.

## 4. Creando nuestro proyecto

```
Versiones utilizadas en esta guía
Next: 14.1.4
Amplify: 0.0.66
```

En esta ocasión vamos a estar usando Next para la creación de nuestro proyecto pero Amplify es compatible con cualquier stack tecnológico mientras tengamos un frontend para mostrar el SDK y un backend para enviar los requests de forma segura.
A. Clona el siguiente repositorio:  https://github.com/Fundit-Finance/sdk-self-with-next
B. Crea en la raíz del proyecto un archivo que se llame .env.local copiando el contenido de .env.example
C. Reemplaza las variables API_KEY y CLIENT_ID por las que generaste en el paso 3 de esta guía
D. Instala las dependencias del proyecto ejecutando npm install
E. Inicia el servidor con el comando npm run dev
F. Ingresa a http://localhost:3000 y si todo esta configurado correctamente deberías ver el SDK funcionando

![image](https://github.com/Fundit-Finance/sdk-self-with-next/assets/4969737/e5368d79-4ce8-4b97-8c86-c3f82ab7dcf3)

## 5. Entendiendo el flujo de la integración

Ahora que ya tenemos nuestra integracion funcional, vamos a entender un poco mejor como funciona el repositorio que clonamos y como podemos personalizarlo para nuestras necesidades. Arranquemos con el frontend.
Para que Amplify se renderice correctamente tenemos que pasarles algunos parametros:

### createPaymentIntentUrl<string>

URL a la cual el frontend le va a hacer la request HTTP de forma automatica al renderizarse para crear la [Intencion de pago](https://amplify-docs.gitbook.io/amplify/amplify-documentation/api/intencion-de-pago). Esta URL tiene que ser una API que admita el método POST. En el ejemplo es `/api/amplify/intent`

### createPaymentIntentData

Si bien esta propiedad no es requerida para que Amplify funcione, en la gran mayoría de los casos vamos a querer guardar información en nuestro backend o enviar metadata a Amplify para poder relacionar pagos con compradores. Esta propiedad es un objeto el cual se va a inyectar automáticamente en el body en el post que se hace a la URL que definimos en `createPaymentIntentUrl<string>`

### environment<string>
Admite los valores `test` y `production`. Los cuales sirven para identificar que tipo de API Key estamos utilizando. Esto va a determinar si a la hora de renderizar el SDK mostramos las redes  y los tokens de prueba o los productivos.

### theme<string>
Amplify soporta light y dark mode por lo que con la propiedad `theme` podemos elegir que tema queremos para nuestra integración

### language<string>
Admite los valores `EN` y `ES`. Sirve para cambiar el idioma de todos los textos internos del SDK

Con estos parámetros ya vas a poder manejar todo el flujo básico de Amplify pero si queres personalizarlo un poco mas, podes encontrar mas información en las secciones [Propiedades](https://amplify-docs.gitbook.io/amplify/amplify-documentation/sdk/propiedades) y [Personalizacion](https://amplify-docs.gitbook.io/amplify/amplify-documentation/sdk/personalizacion)

Para la parte del backend el archivo que hay que analizar es `pages/api/amplify/intent.js`. En donde te vas a encontrar con lo siguiente:

```javascript
import axios from "axios";

export default async function handler(req, res) {
  if (req.method == "POST") {
    try {
      const { API_KEY, CLIENT_ID, BASE_URL } = process.env;

      if (!API_KEY || !CLIENT_ID || !BASE_URL) {
        throw new Error(
          "API_KEY, CLIENT_ID or BASE_URL missing in env.local file"
        );
      }

      const newPaymentIntent = await axios.post(
        `${BASE_URL}/api/payment_intent`,
        {
          apiKey: process.env.API_KEY,
          clientId: process.env.CLIENT_ID,
        }
      );
      return res.status(200).json(newPaymentIntent.data);
    } catch (err) {
      console.log(err);
    }
  }
  return res.status(200).json({ success: "This is not relevant for our code" });
}
```

A. Validamos que el request sea tipo POST
B. Leemos las variables API_KEY, CLIENT_ID y BASE_URL de nuestro archivo de configuración y en caso de que no exista alguna de las claves lanzamos un error
C. Hacemos un HTTP Request a `/api/payment_intent` enviando nuestra API KEY y CLIENT ID. Este endpoint también admite cierta información adicional que podemos enviarle para personalizar nuestra experiencia, podes ver que campos se pueden mandar en 
D. ¡IMPORTANTE! Retornamos a nuestro frontend la información que obtenemos como respuesta del HTTP Request que hicimos en el punto C

## 6. Probando nuestra integracion

Ahora que ya tenemos nuestra integración funcional y que entendemos como es el flujo de la misma podemos hacer un pago de prueba para ver que todo este impactando correctamente.

```
No tenes tokens de prueba? Aprende como conseguirlos de forma gratuita en https://amplify-docs.gitbook.io/amplify/amplify-documentation/obteniendo-tokens-de-prueba
```

Si el pago se ejecuto de forma correcta, en https://admin.getamplify.app/ deberías ver la información del pago

![image](https://github.com/Fundit-Finance/sdk-self-with-next/assets/4969737/e960c2e2-88ed-4a58-9bd2-ede7e7efefd4)
