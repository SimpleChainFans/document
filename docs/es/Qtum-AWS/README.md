# Simplechain en AWS ec2

Simplechain ha estado disponible en el Mercado de AWS desde principios de julio de 2018, este tutorial le mostrará cómo comenzar con Simplechain en AWS.

Ejecutar Simplechain en AWS es increíblemente fácil, podemos implementar una nueva instancia de EC2 directamente desde el mercado. Comencemos por ir al panel de control de EC2 y hacer clic en **"Iniciar instancia"**

![awsmarketplace](https://docs.Simplechain.site/en/Simplechain-AWS/aws.jpg)

Al hacer clic en "Iniciar instancia", irá a la siguiente pantalla, donde podrá elegir una imagen de máquina de Amazon (AMI).

En la parte superior, tenemos un cuadro de búsqueda, escribamos **"Simplechain"** y presione enter.

![awsmarketplace](https://docs.Simplechain.site/en/Simplechain-AWS/aws2.jpg)

Esto muestra el 1er Simplechain AMI, ahora todo lo que tenemos que hacer es hacer clic en **"Select"**

![awsmarketplace](https://docs.Simplechain.site/en/Simplechain-AWS/aws3.jpg)

Aquí nos encontramos con una lista de tipos de instancia, cada instancia tiene diferentes especificaciones y costos, que varían de una región a otra. Para obtener una lista completa de especificaciones + costos, consulte la documentación de AWS sobre este tema.  <https://aws.amazon.com/ec2/pricing/on-demand/>

![awsmarketplace](https://docs.Simplechain.site/en/Simplechain-AWS/aws4.jpg)

A continuación, revisaremos los detalles de nuestra instancia (tamaño del disco, configuración de seguridad, etc.)

![awsmarketplace](https://docs.Simplechain.site/en/Simplechain-AWS/aws5.jpg)

Aquí se nos solicita crear un par de claves para acceder a esta instancia, es un archivo de clave privada que se usa para acceder a través de ssh, esto es más seguro, pero debe tener MUCHO cuidado con este archivo, cualquier persona con este archivo tendrá acceso completo a tu servidor !.

![awsmarketplace](https://docs.Simplechain.site/en/Simplechain-AWS/aws6.jpg)

Tenga en cuenta que el nombre del par de claves es "**Simplechainaws**", esto generará un archivo llamado**"Simplechainaws.pem.txt"**, necesitamos cambiarle el nombre a **Simplechainaws.pem** y cambia los permisos a 0400 escribiendo `mv Simplechainaws.pem.txt Simplechainaws.pem && chmod 400 Simplechainaws.pem`.

![awsmarketplace](https://docs.Simplechain.site/en/Simplechain-AWS/aws7.jpg)

ok, ahora nuestra instancia ha sido lanzada, toma unos segundos para arrancar. Una vez hecho esto, podemos acceder a través de **ssh** con la clave .pem que creamos / descargamos durante la configuración.

![awsmarketplace](https://docs.Simplechain.site/en/Simplechain-AWS/aws8.jpg)

Para acceder a nuestra instancia, escribamos ssh -i Simplechainaws.pem ubuntu @**yourinstanceipaddress**

Escriba "sí" para agregar la huella digital de la clave ECDSA de la instancia ec2 a su archivo conocido_hosts.

Al iniciar sesión, verá la siguiente pantalla, escriba ls para ver los archivos / carpetas que ya están configurados para usted.

![awsmarketplace](https://docs.Simplechain.site/en/Simplechain-AWS/term1.jpg)

`qmix` es la carpeta de instalación y lanzamiento del IDE web de Qmix, es un gran IDE web de código abierto para Simplechain.

[https://qmix.blockchainspaceman.com](https://qmix.blockchainspaceman.com/) es la interfaz oficial del desarrollador Qmix

Puedes encontrar el código fuente aquí <https://github.com/spacemanholdings/qmix>

## Accediendo a Qmix incorporado

Qmix se inicia al iniciarse en la AMI Simplechain AMI. Para usarlo, solo apunta tu navegador hacia [http://yourec2ipaddress:5555](http://yourec2ipaddress:5555/)

![awsmarketplace](https://docs.Simplechain.site/en/Simplechain-AWS/qmix.jpg)

![awsmarketplace](https://docs.Simplechain.site/en/Simplechain-AWS/term2.jpg)

También hay una página de manual disponible, simplemente escriba man README.man y obtendrá una documentación básica sobre cómo usar esta AMI.

Se recomienda realizar una actualización completa del sistema antes de usar Simplechain, esto actualizará todas las aplicaciones y bibliotecas a su última versión, incluyendo parches de seguridad importantes y correcciones de errores, esto incluye Simplechain, que en el momento de la escritura tiene la versión 0.16.1 disponible en los repositorios .

![awsmarketplace](https://docs.Simplechain.site/en/Simplechain-AWS/term5.jpg)

Para actualizar todo, solo necesitamos escribir: `sudo apt update && sudo apt upgrade`

Esto sincronizará cada repositorio e instalará las actualizaciones disponibles.

![awsmarketplace](https://docs.Simplechain.site/en/Simplechain-AWS/term4.jpg)

Una vez que terminemos de actualizar, podemos iniciar el daemon Simplechain escribiendo Simplechaind -daemon

Simplechain-cli -getinfo muestra qué versión estamos ejecutando y cuál es el estado de nuestra sincronización de blockchain.

![awsmarketplace](https://docs.Simplechain.site/en/Simplechain-AWS/term3.jpg)

## Consejos de seguridad:

Las instancias de AWS ec2 son muy seguras por defecto, sin embargo, siempre hay ajustes adicionales que podemos hacer para estar más seguros.

1. Cambie el nombre de usuario predeterminado: de forma predeterminada, las instancias de AWS tienen: "ubuntu" como nombre de usuario y no una contraseña. Se recomienda cambiar esto y crear su propio usuario y contraseña. Una vez que haya creado el nuevo usuario, elimine el usuario predeterminado.
2. Solo abra los puertos que realmente necesita para permitir el acceso. Por defecto, esta instancia de EC2 permitirá el acceso desde 22, 3888 y 5555.
3. Añade tu propia clave ssh al archivo `~./.ssh/authorized_keys` Asegúrese de seguir los procedimientos seguros para acceder a estos casos.