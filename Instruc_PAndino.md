# #ddRAD LimnoPUCE

## ##Demultiplexing

Iluimina reconoce el primer y el sitio de corte (restriction site). Si no reconoce al primer, se borra la lectura para mejorar la calidad de los datos.

_Phred_..(no alcance a anotar bien..)

Ilumina generó ocho pools que fueron entregados en archivos comprimidos formato **.fq.gz** , cada uno de estos archivos (pools) contaba de dos lecturas (R1 y R2). Dentro de cada lectura las secuencias obtenidas estaban asociadas al adaptador específico de cada taxón (individuo). Mientras que las secuencias de cada individuo se encuentra en cada pool. Los archivos **.rem** (removed) son archivos que no han calificado para el Ilumina. Aunque no son  muy útiles son guardados junto con los demas archivos archivos. N

La calidad de cada lectura fue revisada con la ayuda del software FastQ, el cual descomprime los archivos **.gzip** y analiza algunos parámetros de calidad del secuenciamiento... en construcción..!

La finalidad del demultiplexing es la de entregar una identidad a cada secuencia de cada pool. Es decir, identificar las lecturas de cada individuo mediante la asociación de los adaptadores de cada pool con la información codificada de cada muestra secuenciada (códigos durante la extracción de ADN). En nuestro caso, trabajaremos solamente con la primera lectura (lectura simple) y también con ambas en conjunto (lectura doble). Todo este proceso fue realizado con la ayuda del software Stacks, mediante el uso de la función process_radtag. Para lo cual preparamos una matriz con los códigos de extracción con sus respectivos barcodes (matriz "barcodes"). Esta matriz en formato de texto separado por comas fue utilizado junto con la lectura 1 del Pool1.

Usando (Mac Vero y Pool2):

**macs-MacBook-Pro-2: MingaGenomica macuser$ process_radtags -p ./raw/ -o ./process_Pool2_Read1/ -b ./barcodes_Pool2_Read1/Pool2_Read1.txt -r -c -q --inline_null --renz_1 ecoRI --renz_2 mspI**

Donde:

-_p_ Es la ubicación (carpeta) donde están las lecturas crudas para ser "procesadas". Si se usa -P se procesan ambas lecturas juntas.

-_o_ Es la ubicación donde se depositará el archivo generado al final del proceso (output).

-_b_ Archivos que serán asociados en el proceso. Es decir las lecturas de Ilumina (barcodes_Pool2_Read1) y el archivo de texto con la matriz "barcodes".

-_r_ -_c_ -_q_ Son defaults.

_--inline_null--_ Es recomendado cuando se trabaja con dos enzimas de restricción. (Aunque parece que también sale con --inline_index-- cf.).

_--Renz_1_ Enzima de restricción 1 en formato reconosible para Stacks process_radtags. En nuestro caso ecoR1.

_--Renz_2_ Enzima de restricción 2 en formato reconosible para Stacks process_radtags. En nuestro caso mspI.

Durante este proceso es importante revisar muy bien la sintaxis, pues el mínimo error caligráfico evitará que se realice el proceso. Para lo cual usamos el software bbEdit. Que no muestra caracteres escondidos.

Una vez reemplazados los barcodes de los archivos .gzip por los códigos de cada muestra, se crean carpetas para cada genero donde se depositan los respectivos archivos .gzip.

Comando _mv_ nos permite generar una copia de los archivos de nombre distinto.

_Pipe_ (linea vertical) nos permite continuar la sintyaxs de un inpùt como el out put de la suguienmte función. 

El símbolo > nos permite generar un archivo nuevo  a partir del output  de una funcion.



## ##Denovo mapping

Depues del **demulpitplexing >** se agrupa el contenido de los pools en una carpeta por cada taxa, antes de comezar el denovo mapping.

Durante este proceso debemos manejar los nombres de los archivos de cacad carpeta de forma agrupada para editar el texto de los nombres mediante la función _mv_ (move).

**denovo_map.pl**

Mediante el uso de  las funciones:

-_ustack_ (Pipe del denovo map)

-_cstacks_ (Pipe del ustacks)

-_gstacks_ (Pipe del ustacks)

Luego de estos pasos podemos pasar a **Populations**

Los elementos requeridos para la función denovo_map.pl son:

1- Linea de comando:

**denovo_map.pl -M 3 -n 2 -o ./Atop_Test_R1 / --popmap ./popmapfile.txt --samples ./carpeta**

2- Pop map file: Lista de los nombres de los archivos dentro de la carpeta por cada taxa, en formato .txt. Generamos el archivo output mediante: 

**ls > nombre.txt (nombre del bicho) -- samples ./samples** 

En  nuestro caso debemos generar una carpeta con solo el read 1 y otra con ambos reads (1y2 usando la función_--paired_).  Para doble lectura se debe tener en cuenta que en el archivo **pop map**, deben estar las lecturas sin el número de lectura al que corresponde, ni el punto entre el número y el nombre. Es decir, tendremos cada individuo repetido dos veces en el **pop map**, a su vez Stacks asume que los archivos impares son los reads 1 y pares los reads 2.  

El archivo .txt generado (**nombre.txt**) se abre y modifica en Excel, añadiendo una columna nueva con donde ubicamos la localidad de cada muestra (**sin encabezados**).

Luego se reemplazan los terminales (.fq.gz) de cada nombre de archivo mediante la función _buscar_reemplazar_ del Excel.

Para que el Stacks no identifique el punto terminal del formato

e.g. archivo > **001And_G14V.1.fq.gz**

se puede marcar cada sección del nombre de archivo mediante un _Grep_ usando la siguiente expresión regular en TextWrangler:

**IMG01**



## ##Populations