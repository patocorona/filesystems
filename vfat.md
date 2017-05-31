La tabla de ubicación de ficheros (FAT) fue diseñada e implementada en Febrero de 1976 por un muchacho llamado Bill Gates durante una estancia de 5 días en el Hotel Hilton en Albuquerque. Lo desarrolló para una versión de Basic que pudiera almacenar programas y datos en disquetes. El diseño de la FAT fue incorporado por Tim Paterson en una primera versión de un sistema operativo para el chip 8086 de Intel. Gates compró los derechos del sistema, y lo reescribió para crear la primera versión del MS-DOS. Como resultado directo, Bill Gates es hombre más rico de América.
La FAT está estrechamente relacionada con las entradas de directorio del directorio raíz. El directorio raíz es la tercera zona creada al formatear un disco y comienza en el primer sector libre detrás de la segunda copia de la FAT. Un directorio está formado por elementos de 32 bytes llamados entradas de directorio. Hay una entrada de directorio por cada fichero almacenado en el directorio, una entrada por cada subdirectorio y, sólo en el directorio raíz, una entrada por la etiqueta o volumen del disco.

Características de la VFAT
El sistema de ficheros VFAT, a pesar de tener características de implementación distintas, no es un nuevo sistema de ficheros, utiliza la misma estructura de directorio, formato y tipo de partición que la FAT tradicional. El sistema de ficheros VFAT es, simplemente, una forma de almacenar más información en las entradas de directorio de la FAT. La característica más importante de la VFAT es la capacidad de almacenar nombres largos. Como se basa en la FAT tradicional, cada fichero ha de tener un nombre de 8 caracteres más 3 de extensión. Sin embargo, la VFAT añade bloques de directorios adicionales para almacenar un nombre de fichero más largo. Con el sistema VFAT extendido, Microsoft ha insertado entradas de directorio suplementarias, extra, para cualquier tipo de ficheros con nombres "extendidos". (Cualquier nombre que legalmente se adecue al antiguo esquema de codificación 8.3, no tiene entradas extra.). Llamamos a estas entradas extra, slots. Un slot es, a grandes rasgos, una entrada de directorio de formato especial. Cada slot de los que acompañan a una entrada de directorio tradicional, puede almacenar hasta 13 caracteres de los que componen el nombre largo. Imaginémonos los slots como un etiquetado especial para la entrada de directorio a la que corresponden. Microsoft prefiere referirse a la entrada 8.3 de un directorio como su alias, y a las entradas de directorio de un slot extendidas, como al nombre del fichero.

Si el aspecto de los slots parece un poco extraño, es sólo debido a los esfuerzos de Microsoft de mantener la compatibilidad con el software anterior. Los slots se han de disfrazar, encubrir para proteger al software anterior de situaciones de error. Con esta finalidad se ha tomado una serie de medidas:

-	Los tamaños de las entradas de directorio tradicionales y de los slots son iguales.
-	El byte de atributo para la entrada de directorio slot siempre está puesto a 0x0F (15 hex). Esto corresponde a una entrada de directorio antigua con atributos de "oculto", "sistema", "sólo lectura" y "etiqueta de volumen". La mayoría de software antiguo ignorará cualquier entrada de directorio con el bit de "etiqueta de volumen" activado. Las entradas de etiqueta de volumen reales no tienen los otros tres bits activados.
-	El cluster de inicio (unsigned char start[2]) siempre se pone a 0, que es un valor imposible para un fichero de DOS
-	Además, la posición de los bytes de atributo y cluster de inicio dentro de las entradas de directorio tradicionales y de los slots, coinciden.

Por tanto, el MS-DOS reconoce un slot como una entrada de directorio. Cuando intente leerlo, los valores del atributo y cluster de inicio -por lo dicho anteriormente- servirán para informarle de que no es una entrada de directorio, y que debe ignorarla. Como el sistema FAT extendido es compatible con niveles anteriores, el software antiguo es capaz de modificar las entradas de directorio. Se deben tomar medidas para asegurar la validez de los slots. El sistema de ficheros FAT extendido puede verificar que un slot pertenece, de hecho, a una entrada de directorio de 8.3 caracteres, de la forma siguiente:

Posicionado. Los slots de un fichero siempre están inmediatamente después de su entrada de directorio 8.3 correspondiente. Además, cada slot posee un identificador que señala su orden dentro del nombre de fichero extendido. Mostramos un breve ejemplo de una entrada de directorio 8.3 y sus correspondientes slots de nombre largo para el fichero "Mi Fichero largo.Extensión":
    ==> ficheros anteriores...
    ==> slot #2, id = 0x02, caracteres = "rgo.Extensión"
    ==> slot #1, id = 0x01, caracteres = "Mi Fichero la"
    ==> entrada de directorio, nombre = "MIFICHER.EXT">

Se observa que los slots se almacenan del último al primero. Los slots se numeran de 1 hasta N. Al N-ésimo slot se le aplica un OR lógico con 0x40 para indicar que es el último.

Checksum. Cada slot tiene un valor "alias_checksum". El checksum se calcula a partir del nombre 8.3 utilizando el siguiente algoritmo:


    for (sum = i = 0; i < 11; i++) 
    {       
      sum = (((sum & 1)<<7)| ((sum & 0xfe)>>1)) + name[i]       
    };

Si hay slots, en el slot final se almacena un NULL Unicode (0x0000) después del último carácter. Después de esto, todos los caracteres sin utilizar en el slot final son puestos a un valor Unicode igual a 0xFFFF. Finalmente, señalar que el nombre extendido se almacena en Unicode. El Unicode es un esquema de codificación de 16 bits unificado que proporciona soporte a todos los tipos de escritura de la Tierra. Por tanto cada carácter Unicode tiene dos bytes.
