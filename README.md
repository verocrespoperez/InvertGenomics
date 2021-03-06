# InvertGenomics

###**Pasos para genotipificar:**

1. Demultiplexing
2. De novo mapping
3. Resumen de resultados con Populations
4. Filtrado de matrices
5. Análisis de las matrices de SNPs con R

## 1. Demultiplexing
Ilumina generó ocho _Pools_ que fueron entregados en carpetas con archivos comprimidos en formato **.fq.gz**. Cada carpeta contenía dos archivos, uno por cada lectura (Read 1 y Read 2). Dentro de cada archivo, de cada pool, las secuencias estaban asociadas a un _Barcode_ específico por individuo*.
> `* el laboratorio envió junto con los resultados un archivo con los barcodes asignados a cada individuo.`  

Los archivos **.rem** (removed) son archivos que no han calificado para el Ilumina. Aunque no son  muy útiles se guardan junto con los demas archivos. 

> NOTA: antes de empezar el _demultiplexing_ la calidad de cada lectura fue revisada con la ayuda del software **FastQC**, el cual descomprime los archivos **.gzip** y analiza algunos parámetros de calidad del secuenciamiento.

La finalidad del _demultiplexing_ es dar una identidad a cada secuencia de cada _Pool_. Es decir, identificar y separar las lecturas de cada individuo mediante la asociación de cada uno con su _Barcode_. En nuestro caso trabajaremos con ambas lecturas en conjunto (lectura doble). Todo este proceso fue realizado con la ayuda del software Stacks, mediante el uso de la función process_radtags. Para esto, preparamos una matriz (.txt) con los códigos de extracción (un codigo por individuo) con sus respectivos barcodes (matriz "barcodes"). Esta matriz se utilizó para el proceso de demultiplexing:

####**Example code for pool 1:**   

Código:  

`process_radtags -P -b ./Barcode_P1_R1.txt -c -q -r  --renz_1 ecoRI --renz_2 mspI -p /Users/Vero/Documents/BaseSpace/JA18493-109360251/ddRadMacros/raw/Pool1_R1R2 --inline_null -o /Users/Vero/Documents/BaseSpace/JA18493-109360251/ddRadMacros/Output_P1_R1R2`

>**-P:** Demultiplexing the two runs (R1 and R2). También sirve con **--paired**  
**-b**: Barcode file. Archivos de texto que asocian a cada barcode con cada individuo.  
**-_r_ -_c_ -_q_**: defaults. Ver [aquí](http://catchenlab.life.illinois.edu/stacks/comp/process_radtags.php) para más información.    
**-p**: Raw files. Ubicación (carpeta) donde están las lecturas crudas para ser procesadas. Si se usa -P se procesan ambas lecturas juntas.  
**-o**: Output directory. Ubicación donde se depositará el archivo generado al final del proceso (output).  
**--inline_null**: barcode is inline with sequence, occurs only on single-end read (default).     
**--renz_1**: Enzima de restricción 1 en formato reconocible para Stacks. En nuestro caso **ecoR1**.  
**--renz_2**: Enzima de restricción 2 en formato reconocible para Stacks. En nuestro caso **mspI**.

Durante este proceso es importante revisar muy bien la sintaxis, pues el mínimo error caligráfico evitará que se realice el proceso. Para corregir errores usamos el software bbEdit (o TextWrangler), que muestra caracteres escondidos.

Click [here](http://catchenlab.life.illinois.edu/stacks/comp/process_radtags.php) for additional process_radtags options

####**Results - demultiplexing  with process\_radtags**  
**Pool 1:**  
27132412 total sequences  
   79572 barcode not found drops (0.3%)  
   33396 low quality read drops (0.1%)  
  190903 RAD cutsite not found drops (0.7%)  
26828541 retained reads (98.9%)

**Pool 2:**  (Redro)  
29801616 total sequences  
   95236 barcode not found drops (??)  
   36857 low quality read drops (??)  
  216842 RAD cutsite not found drops (??)  
29452681 retained reads (??)

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

**Pool 5:**  (Redro)  
29332378 total sequences  
   103512 barcode not found drops (??)  
   34778 low quality read drops (??)  
  177205 RAD cutsite not found drops (??)  
29016883 retained reads (??) 

**Pool 6:**  
24683186 total sequences  
   87448 barcode not found drops (0.4%)  
   28906 low quality read drops (0.1%)  
  144436 RAD cutsite not found drops (0.6%)  
24422396 retained reads (98.9%)  

**Pool 7:**  (Redro)  
29365106 total sequences  
   111598 barcode not found drops (??)  
   36271 low quality read drops (??)  
  169717 RAD cutsite not found drops (??)  
29047520 retained reads (??) 

**Pool 8:**  (Redro)  
24115006 total sequences  
   91558 barcode not found drops (??)  
   29237 low quality read drops (??)  
  167554 RAD cutsite not found drops (??)  
23826657 retained reads (??) 

**IMPORTANT NOTES:**  
If error: "(filenames can consist of letters, numbers, '.', '-' and '_')", there are probably errors in the barcode file (i.e. extra spaces or unwanted simbols).  
> * Solution: open the barcodes file with TextWrangler, click on View -> Text Display -> Show invisibles
>   * This shows spaces, line breaks, etc.  
>   * Remove excess spaces or unwanted symbols 


## 2. _de novo_ mapping  
Esta serie de pasos sirve para filtrar los datos con stacks y generar al final la matriz de SNPs de acuerdo a los parámetros -m, -M y -N (ver abajo).

###Primer paso: ordenar los archivos
Depués del **demulpitplexing**, y antes del _de novo_ mapping, se agrupa el contenido de los _Pools_ en una carpeta por cada taxón.

Para crear directorios se usa la función **mkdir**, pero antes se debe cambiar el directorio al lugar donde queremos crear la nueva carpeta.

Código:

`cd /Users/Vero/Documents/BaseSpace/JA18493-109360251/ddRadMacros/TaxaDemultOutputs`

`mkdir Test`

Para mover archivos de una carpeta a otra se usa la función _mv_ (move).

###Segundo paso: crear un archivo con los codigos de los individuos y los sitios a los que pertenecen (Pop map file).

Para hacer esto:  

1. Hacer una lista de todos los archivos de la carpeta donde estan las muestras y crear un archivo de texto con esa lista. Para esto:

	ls > newfilename.txt  
	**¡Importante 1!** antes se debe cambiar el directorio (con cd) para estar en la carpeta donde están los archivos (samples).  
	**¡Importante 2!** borrar los archivos innecesarios como los rem (removidos durante el demultiplexing). También, en el caso de que se vayan a analizar solo los R1, dejar solo los archivos de read 1 en la carpeta. 
	  
2. Editar el archivo de texto en TextWrangler para borrar partes innecesarias:   

	Ejemplo:   
	En el nombre: **001And_G14V.1.fq.gz** no es necesario todo lo que va después del primer punto. Además, falta otra columna con el nombre (o código) de los sitios. Para modificar todo esto:
 
![Codigo grep](https://github.com/verocrespoperez/InvertGenomics/blob/master/Fotos/GrepTW.jpg)

De esta forma, se genera un archivo de texto con el formato correcto:

![PopMap file](https://github.com/verocrespoperez/InvertGenomics/blob/master/Fotos/ScreenShot_PopMap.jpg)

NOTA: ¡¡¡Cuidar que los mismos sitios tengan el mismo nombre!!!
> Algunos bichos dicen "Purg" y otros "Pur"
> Se puede corregir esto con un grep:
> ... insert image

NOTA2: solo debe haber una fila por individuo, incluso si se van a analizar las dos repeticiones (R1 y R2)

###Tercer paso: denovo sequencing

Para empezar a filtrar los datos se usa el programa **`denovo_map.pl`**

Código:   

`denovo_map.pl -M 3 -n 2 -o ./DeNovo_And_R1/ --popmap ./PopMap_And_R1.txt --samples ./Andesiops_R1`
> **-o**: output directory (hay que crearlos, eg. con `mkdir`)  
> **--popmap**: archivo popmap  
> **--samples**: directorio donde están las muestras (demultiplexed files)  
> **-M** y **-n** son parámetros de STACKS<br><br/>
**Parámetros de stacks** <br><br/> 
_**[ustacks]**: within individuals_   
> **-m**: _Minimum stack depth / minimum depth of coverage_ (default = 3). Numero de _reads_ idénticos para ser considerados un _stack_.   
```Entonces, si m = 3, solo grupos formados por 3 o más secuencias idénticas se considerarán como stacks.```   
> **-M**: _Distance allowed between stacks_ (default = 2). Comparación entre los stacks creados con el parámetro **-m** para agrupar alelos de un mismo locus. **-M** representa el número de nucleótidos que pueden ser diferentes entre dos _stacks_ para unirlos.  Las diferencias en nucleótidos podrían deberse a polimorfismos entre alelos o a errores de secuenciación.  
```Entonces, si M = 2, se permiten máximo dos diferencias entre stacks para agruparlos.```   
> **-N**: Permite reincorporar _secondary reads_ no incluidos durante el primer paso, para tratar de tener el mayor _coverage_ posible.<br><br/>
_**[cstacks]**: among individuals_  
> **-n**: _Distance allowed between catalog loci_ (default = 1). Un catálogo contiene todos los loci y alelos de la población. El valor de **-n** representa el número de diferencias que se permiten entre loci de de varios individuos pra formar un _stack_. Paris et al. [(2017)](https://github.com/verocrespoperez/InvertGenomics/blob/master/Bibliografia/Paris-etal-2017.pdf) recomiendan que `-n`=`-M`, `-n`=`-M`+1 o `-n`=`-M`-1.    

Para más información sobre los parámetros de STACKS ver el siguiente [tutorial](http://catchenlab.life.illinois.edu/stacks/param_tut.php) y revisar el artículo de Paris et al. [(2017)](https://github.com/verocrespoperez/InvertGenomics/blob/master/Bibliografia/Paris-etal-2017.pdf).

#### Determinación de los parámetros (_-m_, _-M_ y _-n_) óptimos
Para determinar los parámetros óptimos elegimos combinaciones lógicas basadas en el artículo de Paris et al. [(2017)](https://github.com/verocrespoperez/InvertGenomics/blob/master/Bibliografia/Paris-etal-2017.pdf).   
 
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

El programa populations de Stacks calcula las estadisticas poblacionales (eg. Fst, pi, etc.). Primero, vamos a mirar los datos de forma muy general buscando loci que estén en el 80% de los individuos (r = 0.8) y asumiendo que todos los individuos pertenecen a la misma población.  

Código:

`populations -P ./DeNovo_And_R1R2_T8 --popmap ./PopMap_And_sin_rem_2.txt -O ./DeNovo_And_R1R2_T8/Populations_And_T8 -p 1 -r 0.8 --write_random_snp --vcf`

Donde:
> **-P**: ruta hacia el directorio de los archivos de Stacks (los generados por el _denovo_ map).    
> **--popmap**: archivo popmap en el que todos los individuos pertentecen a la misma población (i.e. mismo código de sitio para todos).  
> **-O**: Output directory  
> **-r**: _min-samples-per-pop_. Porcentaje mínimo de individuos en los que está un locus en una población para procesar ese locus.  
Entonces, si r = 0.8, entonces solo se procesarán los locus que estén presentes en, por lo menos, 80% de los individuos de la población.    
> **-p**: _min-populations_. Número mínimo de poblaciones en los que debe estar presente un locus para procesar ese locus.  
Entonces, si p = 1, un locus debe estar por lo menos en 1 población para procesar ese locus. 

<br><br/>
###Obtuvimos los siguientes resultados para loci kept y variant sites remained:  

## _ANDESIOPS_
![ResultadosTot](https://github.com/verocrespoperez/InvertGenomics/blob/master/Fotos/ResultsDNovoAnd_Total.png)
Resultados totales obtenidos del denovo_map para las distintas combinaciones de parámetros.

![ResultadosPops](https://github.com/verocrespoperez/InvertGenomics/blob/master/Fotos/ResultsDNovoAnd_Pops.png)
Resultados obtenidos después de correr populations para las distintas combinaciones de parámetros.

**Andesiops:** nos vamos a quedar con dos combinaciones de parámetros:  
1: **m5 M6 n7** (3849 Loci kept y 3813 Variant sites remained) (Repetición **T8**)  
2: **m4 M6 n7** (4408 Loci kept y 4367 Variant sites remained) (Repetición **T7**)  

El problema con los m=4 es que hay algunos individuos que tienen en promedio bajo coverage (< 6)  
```
NOTA: nano .bash_profile sirve para editar el archivo .bash_profile que es un archivo del sistema operativo...
```

## 4. Filtrado de matrices

Luego de decidir la mejor combinación de parámetros, se deben filtar los datos para llegar a la matriz final de SNPs. Para esto hay que:

- Exportar la matriz de SNPs en formato .vcf utilizando populations con parámetros más restrictivos (p = 1 y r = 0.1) y con las poblaciones reales a las que pertenece cada individuo (i.e. el archivo _popmap_ original): 

	Código:  
	
	`populations -P ./DeNovo_And_R1R2_T8 --popmap ./PopMap_And_R1.txt -O ./DeNovo_And_R1R2_T8/Populations_And_T8 -p 1 -r 0.1 --write_random_snp --vcf`

- Luego hay que instalar el programa **vcftools** que sirve para transformar y filtrar los datos. Para esto:
	1. Bajar el programa del internet [aquí.](https://sourceforge.net/projects/vcftools/files/vcftools_0.1.13.tar.gz/download)
	2. Descomprimirlo.  
	
		Código:  
`tar -xvf vcftools_0.1.13.tar.gz`  
		
			NOTA: antes de esto es importante estar en el directorio donde está la carpeta comprimida de vcftools.
		
	3. Luego se va al directorio nuevo de **vcftools** (descomprimido) y se siguen los siguientes pasos para compilar el programa.  
	
		Código:  
	`./configure`solo si no hay el archivo "makefile".  
	`make`  
	`make install`
	
- Transformar el archivo .vcf:  
Para esto primero hay que ir a la carpeta donde está el archivo .vcf creado con populations.   

	Código:  
		
	`cd /Users/Vero/Documents/BaseSpace/JA18493-109360251/ddRadMacros/TaxaDemultOutputs/DeNovo_And_R1R2_T8/Populations_And_T8`

	A continuacióm, se puede ya transformar el archivo .vcf con el siguiente código:

	`vcftools --vcf ./populations.snps.vcf --plink --out And_T8`  
		
		NOTA: Esto produce tres archivos nuevos: .log, .ped y .map.
		
En este paso tenemos:
**66587** loci para Andesiops T8 
**69674** loci para Andesiops T7

### Filtrado de matrices con plink

Plink ayuda a remover SNPs raros (que estan en pocos individuos), individuos con pocos datos y alelos raros. Para estos analisis primero hay que:  

1. Mover el ejecutable de plink al directorio donde están los archivos que se van a filtrar (.ped y .map).
2. Luego se filtran los datos en una serie de pasos:

Codigo:
`./plink --file And_T8 --geno 0.5 --recode --out And_T8_a --noweb`
`--geno`: filtra los loci que están presentes en por lo menos el tanto % de los individuos.

`./plink --file And_T8_a --mind 0.5 --recode --out And_T8_b --noweb`
`--mind`: filtra individuos con muy pocos datos

`./plink --file And_T8_b --maf 0.01 --recode --out And_T8_c --noweb`
`--maf`: (_minor allele frequency_) remueve alelos raros que aparecen en solo el tanto por ciento de los individuos. 

####Resultados para _Andesiops_ T8

 **Con geno = 0.5, mind = 0.5 y maf = 0.01**  

> Resultados **a**:   
> * After frequency and genotyping pruning, there are **9954** SNPs

> Resultados **b**:  
> * 16 of 82 individuals removed for low genotyping ( MIND > 0.5 )
		
		66 individuos restrantes

> Resultados **c**:
> * After frequency and genotyping pruning, there are **7170** SNPs  

Nota: **a** = output de primer filtro (geno), **b** = output de segundo filtro (mind) y **c** = output del tercer filtro (maf)


**Con geno = 0.4 (SNPs que estan en, por lo menos, el 60% de los individuos), mind = 0.5 y maf = 0.01**
> Resultados **a**:   
> * After frequency and genotyping pruning, there are **7217** SNPs

> Resultados **b**:  
> * 15 of 82 individuals removed for low genotyping ( MIND > 0.5 )
		
		67 individuos restrantes
		
> Resultados **c**:	
> * After frequency and genotyping pruning, there are **4721** SNPs

**Con geno = 0.3 (SNPs que estan en, por lo menos, el 70% de los individuos), mind = 0.5 y maf = 0.01**

> Resultados a:  
> * After frequency and genotyping pruning, there are **5752** SNPs

> Resultados b  
> * 10 of 82 individuals removed for low genotyping ( MIND > 0.5 )
		
		72 individuos restrantes
		
> Resultados c	
> * After frequency and genotyping pruning, there are **4615** SNPs

####Nos vamos a quedar con esta combinacion de parametros para _Andesiops_ T8 (geno = 0.3, mind = 0.5 y maf = 0.01) que nos deja con 72 individuos y 4615 SNPs. 

______________________________________
####Resultados para _Andesiops_ T7

 **Con geno = 0.5, mind = 0.5 y maf = 0.01**  

> Resultados **a**:  
> * After frequency and genotyping pruning, there are 10784 SNPs

> Resultados **b**:  
> * 14 of 82 individuals removed for low genotyping ( MIND > 0.5 )

		68 individuos restrantes

> Resultados **c**:
> * After frequency and genotyping pruning, there are 7829 SNPs

**Con geno = 0.4 (SNPs que estan en, por lo menos, el 60% de los individuos), mind = 0.5 y maf = 0.01**
> Resultados **a**:   
> * After frequency and genotyping pruning, there are 7655 SNPs

> Resultados **b**:   
> * 13 of 82 individuals removed for low genotyping ( MIND > 0.5 )

		69 individuos restrantes

> Resultados **c**:	
> * After frequency	and genotyping pruning, there are 5015 SNPs  	

**Con geno = 0.3 (SNPs que estan en, por lo menos, el 70% de los individuos), mind = 0.5 y maf = 0.01**

> Resultados **a**:  
> * After frequency and genotyping pruning, there are 6244 SNPs

> Resultados **b**:   
> * 9 of 82 individuals removed for low genotyping ( MIND > 0.5 )

		73 individuos restrantes

> Resultados **c**:	
> * After frequency and genotyping pruning, there are 5010 SNPs

####Nos vamos a quedar con esta combinacion de parametros para _Andesiops_ T7 (geno = 0.3, mind = 0.5 y maf = 0.01) que nos deja con 73 individuos y 5010 SNPs.

###Linkage
		
Calcula el nivel de linkage entre SNPs. Para esto se usa el siguiente codigo:

`./plink --file And_T8_c --r2 --out And_T8_Link`  
	
El input file es el archivo final de lo anterior (aplicados los tres filtros seleccionados). Con esto se obtienen varios archivos (.ld, .log .nosex). El archivo .ld inidca el nivel de linkage entre pares de SNPs y nos sirve para luego excluir SNPs con alto linkage. Para esto: 

- Abrimos el archivo .ld en TextWrangler y lo transformamos a .txt para abrirlo con Excel, donde ordenamos las filas de acuerdo al r2 que representa la proporcion de linkage.   

		Nota 1: En el archivo .ld los nombres de los SNPs estan dados como: 4432:3. El primer número representa el número de locus (Stacks asigna cualquier numero a los locus) y el segundo la posicion del SNP en ese locus.    

		Nota 2: Si excel no lee bien los nombres de las celdas (¡¡¡en nuestro caso les dio formato de fecha!!!) se puede cambiar con TextWrangler los ":" por "-".

- Elegimos un umbral de linkage y excluimos todos los SNPs que tengan un linkage mayor a este umbral. En el caso de _Andesiops_ vamos a excluir SNPs con linkage mayor a 0.6 (con > 0.5 perdemos demasiados SNPs). 

- En la columna SNP_A del excel, elegimos todos los SNPs con r2 mayor o igual a nuestro umbral. Copiamos esas celdas, pegamos en otra hoja de Excel y removemos los duplicados (boton "Remove Duplicates" de la pestaña "Data").

- Copiamos y pegamos esas celdas en un archivo de TextWrangler (cambiamos "-" por ":" de ser necesario) y lo grabamos como .txt con un nombre como "Blacklist_AndT8.txt". Este archivo se va a usar para excluir los SNPs con alto linkage en lo siguiente:

`./plink --file And_T8_c --exclude BlackList_AndT8.txt --recode --out And_T8_d` 

#### Esto nos deja con:
**2633** SNPs y **72** individuos para _Andesiops_ T8  

**2892** SNPs y **73** individuos para _Andesiops_ T7


###Heterocigocidad

Para estimar el nivel de heterocigocidad, que nos da una idea del nivel de endogamia en las poblaciones, usamos el siguiente código:

`./plink --noweb --file And_T8_d --het --out And_T8_Het`

El _input file_ es el output del paso anterior, osea ya filtrado el linkage, en este caso `And_T8_d`.

## 5. Análisis de las matrices de SNPs con R
Los archivos .ped y .map que se obtienen de lo anterior se deben convertir a archivos de structure (.str). Esto se hace con el programa PGDSpider...  
Por alguna razón, el archivo .str obtenido asigna una población diferente a cada individuo, en lugar de asignarles las poblaciones del archivo popmap. El archivo se ve así:  

![ArchivoStr_SinModf](https://github.com/verocrespoperez/InvertGenomics/blob/master/Fotos/Screen_Shot_Str1.png)

Para solucionar esto se puede modificar el archivo con grep en TextWrangler de la siguiente manera:

![GrepTW2](https://github.com/verocrespoperez/InvertGenomics/blob/master/Fotos/Screen_Shot_Grep2_Str.png)

El archivo modificado se debería ver así:  

![ArchivoStr_Modf](https://github.com/verocrespoperez/InvertGenomics/blob/master/Fotos/Screen_Shot_Str2.png)




