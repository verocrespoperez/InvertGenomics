# InvertGenomics

###**Pasos para genotipificar:**

1. Demultiplexing
2. De novo mapping
3. Resumen de resultados con Populations

## 1. Demultiplexing
Iluimina reconoce el _Primer_ y el sitio de corte (restriction site). Si no reconoce al primer, se borra la lectura para mejorar la calidad de los datos.

Ilumina generó ocho _Pools_ que fueron entregados en carpetas con archivos comprimidos en formato **.fq.gz**. Cada carpeta contenía dos archivos, uno por cada lectura (Read 1 y Read 2, R1 y R2). Dentro de cada archivo, de cada pool, las secuencias estaban asociadas a un _Barcode_ específico por individuo*.
> `* el laboratorio envió junto con los resultados un archivo con los barcodes asignados a cada individuo.`  

Los archivos **.rem** (removed) son archivos que no han calificado para el Ilumina. Aunque no son  muy útiles se guardan junto con los demas archivos archivos. 

> NOTA: antes de empezar el demultiplexing la calidad de cada lectura fue revisada con la ayuda del software **FastQC**, el cual descomprime los archivos **.gzip** y analiza algunos parámetros de calidad del secuenciamiento.

La finalidad del demultiplexing es la dar una identidad a cada secuencia de cada _Pool_. Es decir, identificar y separar las lecturas de cada individuo mediante la asociación de los _Barcodes_ a cada indiviudo. En nuestro caso, trabajaremos solamente con la primera lectura (lectura simple) y también con ambas en conjunto (lectura doble). Todo este proceso fue realizado con la ayuda del software Stacks, mediante el uso de la función process_radtags. Para esto, preparamos una matriz (.txt) con los códigos de extracción (un codigo por individuo) con sus respectivos barcodes (matriz "barcodes"). Esta matriz se utilizó para el proceso de demultiplexing:

Usando (Mac Vero y Pool2):

    macs-MacBook-Pro-2: MingaGenomica macuser$ process_radtags -p ./raw/ -o ./process_Pool2_Read1/ -b 
    ./barcodes_Pool2_Read1/Pool2_Read1.txt -r -c -q --inline_null --renz_1 ecoRI --renz_2 mspI    
  
**Donde:**

-_p_ Es la ubicación (carpeta) donde están las lecturas crudas para ser "procesadas". Si se usa -P se procesan ambas lecturas juntas.

-_o_ Es la ubicación donde se depositará el archivo generado al final del proceso (output).

-_b_ Archivos de texto que asocian a cada barcode con cada individuo.

-_r_ -_c_ -_q_ Son defaults.

_--inline_null--_ Es recomendado cuando se trabaja con dos enzimas de restricción. (Aunque parece que también sale con --inline_index-- cf.).

_--Renz_1_ Enzima de restricción 1 en formato reconosible para Stacks process_radtags. En nuestro caso ecoR1.

_--Renz_2_ Enzima de restricción 2 en formato reconosible para Stacks process_radtags. En nuestro caso mspI.

Durante este proceso es importante revisar muy bien la sintaxis, pues el mínimo error caligráfico evitará que se realice el proceso. Para lo cual usamos el software bbEdit. Que no muestra caracteres escondidos.

Una vez reemplazados los barcodes de los archivos .gzip por los códigos de cada muestra, se crean carpetas para cada genero donde se depositan los respectivos archivos .gzip.

Comando _mv_ nos permite generar una copia de los archivos de nombre distinto.

_Pipe_ (linea vertical) nos permite continuar la sintyaxs de un inpùt como el out put de la suguienmte función. 

El símbolo > nos permite generar un archivo nuevo  a partir del output  de una funcion.

####**Example code for pool 1:** 

>process\_radtags -P -b ./Barcode\_P1\_R1.txt -c -q -r --inline\_index --renz\_1 ecoRI --renz\_2 mspI -p ./raw/Pool1\_R1R2 -o ./Output\_P1\_R1R2\_B --inline\_null

>-P: Demultiplexing the two runs (R1 and R2)  
-b: Barcode file  
-p: Raw files  
-o: Output directory

>Click [here](http://catchenlab.life.illinois.edu/stacks/comp/process_radtags.php) for additional process_radtags options

####**Results - demultiplexing  with process\_radtags**  
**Pool 1:**  
27132412 total sequences  
   79572 barcode not found drops (0.3%)  
   33396 low quality read drops (0.1%)  
  190903 RAD cutsite not found drops (0.7%)  
26828541 retained reads (98.9%)

**Pool 2:**  

**Pool 3:**  
20637354 total sequences  
   84620 barcode not found drops (0.4%)  
   25263 low quality read drops (0.1%)  
  157634 RAD cutsite not found drops (0.8%)  
20369837 retained reads (98.7%)

**Pool 4:**  
26747308 total sequences  
   90238 barcode not found drops (0.3%)  
   33530 low quality read drops (0.1%)  
  183589 RAD cutsite not found drops (0.7%)  
26439951 retained reads (98.9%)    

**Pool 5:**  

**Pool 6:**  
24683186 total sequences  
   87448 barcode not found drops (0.4%)  
   28906 low quality read drops (0.1%)  
  144436 RAD cutsite not found drops (0.6%)  
24422396 retained reads (98.9%)  

**Pool 7:**  

**Pool 8:**  

**IMPORTANT NOTES:**  
If error: "(filenames can consist of letters, numbers, '.', '-' and '_')", there are probably errors in the barcode file (i.e. extra spaces or unwanted simbols).  
> * Solution: open the barcodes file with TextWrangler, click on View -> Text Display -> Show invisibles
>   * This shows spaces, line breaks, etc.  
>   * Remove excess spaces or unwanted symbols 


## 2. _de novo_ mapping  

###Primer paso: ordenar los archivos
Depues del **demulpitplexing**, y antes del _de novo_ mapping, se agrupa el contenido de los pools en una carpeta por cada taxa.

Para crear directorios se usa la función **mkdir**, pero antes se debe cambiar el directorio al lugar donde queremos crear la nueva carpeta.

Ejemplos:

	cd /Users/Vero/Documents/BaseSpace/JA18493-109360251/ddRadMacros/TaxaDemultOutputs
	
	mkdir Test

Para mover archivos de una carpeta a otra se usa la función _mv_ (move).

###Segundo paso: crear un archivo con los codigos de los individuos y los sitios a los que pertenecen (Pop map file).

Para hacer esto:  

1. Hacer una lista de todos los archivos de la carpeta donde estan las muestras. Para esto:

	ls > newfilename.txt  
	**¡Importante 1!** antes se debe cambiar el directorio (con cd) para estar en la carpeta donde están los archivos (samples).  
	**¡Importante 2!** borrar los archivos innecesarios como los rem (removidos durante el demultiplexing). También, si solo se va a analizar los R1, dejar solo los archivos de read 1. 
	  
2. Editar el archivo en TextWrangler para borrar partes innecesarias:   

	En el nombre: **001And_G14V.1.fq.gz** no es necesario todo lo que va después del punto. Además, falta otra columna con el nombre (o código de los sitios). Para modificar todo esto:
 
![Codigo grep](https://github.com/verocrespoperez/InvertGenomics/blob/master/Fotos/GrepTW.jpg)

De esta forma, se genera un archivo de texto con el formato correcto:

![PopMap file](https://github.com/verocrespoperez/InvertGenomics/blob/master/Fotos/ScreenShot_PopMap.jpg)

NOTA: ¡¡¡Cuidar que los mismos sitios tengan el mismo nombre!!!
> Algunos bichos dicen "Purg" y otros "Pur"
> Se puede corregir esto con un grep:
> ... insert image

###Tercer paso: denovo sequencing

Para empezar a filtrar los datos se usa el programa **`denovo_map.pl`**

Código: `denovo_map.pl -M 3 -n 2 -o ./DeNovo_And_R1/ --popmap ./PopMap_And_R1.txt --samples ./Andesiops_R1`
> -o: output directory (hay que crearlos, eg. con _mkdir_)  
> --popmap: archivo popmap  
> --samples: directorio donde están las muestras (demultiplexed files)  
> -M y -n son parámetros de STACKS 

Donde:
> -_m_: [ustacks]: Minimum stack depth / minimum depth of coverage (default: 3); para generar stacks idénticos.

> -_M_: [ustacks]: Distance allowed between stacks (default: 2); permite comparar entre los stacks creados en el parámetro -m; cuantos lugares polimórficos se permiten dentro de los individuos para formar un stack, define los heterocigotos dentro de los individuos.

> -_N_: [ustacks]: Permite reincorporar secondary reads no incluidos durante el primer paso, para tratar de tener el mayor coverage posible.

> -_n_: [cstacks]: Distance allowed between catalog loci. A catalog contains all the loci and alleles in the population.

Para más información sobre los parámetros de STACKS ver el siguiente [tutorial](http://catchenlab.life.illinois.edu/stacks/param_tut.php) y revisar el artículo de Paris et al. (2017).

#### Determinación de los parámetros (_m_, _M_ y _n_) óptimos
Para determinar los parámetros óptimos elegimos combinaciones lógicas basadas en el artículo de Paris et al. (2017) y corrimos denovo_map para esas combinaciones.   
 
**Para _Andesiops_ probamos:**  
m4M6n4  
m4M6n5  
m4M6n6  
m4M6n7  
m4M7n7  
m4M7n8  
m5M6n5  
m5M6n6  
m5M6n7  
m5M7n7  
m5M7n8  
m6M6n6  
m6M6n7  
m6M7n7  
m6M7n8  

## 3. Resumen de resultados con Populations

El programa populations de stacks calcula las estadisticas poblacionales (eg. Fst, pi, etc.). El código es:

> `populations -P ./DeNovo_And_R1R2_T8 --popmap ./PopMap_And_R1.txt -O ./DeNovo_And_R1R2_T8/Populations_And_T8 -p 0.1 -r 0.1 --write_random_snp --vcf`

Donde:
> -_r_: 
    
> -_p_: 

Los resultados obtenidos 

Obtuvimos los siguientes resultados para loci kept y variant sites remained:  


Andesiops: nos vamos a quedar con dos combinaciones de parámetros:
1: m5 M6 n7 (3849 Loci kept y 3813 Variant sites remained) (Repetición T8)
2: m4 M6 n7 (4408 Loci kept y 4367 Variant sites remained) (Repetición T7)

El problema con los m=4 es que hay algunos individuos que tienen en promedio bajo coverage (< 6)

nano .bash_profile


