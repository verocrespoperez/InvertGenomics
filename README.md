# InvertGenomics

## Notas _de novo_ sequencing macroinvertebrates

###Primero: hay que crear un archivo con los codigos de los individuos y los sitios a los que pertenecen.

Para hacer esto:  

1. Hacer una lista de todos los archivos de la carpeta donde estan las muestras. Para esto:

	ls > newfilename.txt  
	**¡Importante 1!** antes se debe cambiar el directorio (con cd) para estar en la carpeta donde están los archivos (samples).  
	**¡Importante 2!** borrar los archivos innecesarios como los rem (removidos durante el demultiplexing). También, si solo se va a analizar los R1, dejar solo los archivos de read 1. 
	  
2. Editar el archivo en TextWrangler para borrar partes innecesarias:   

	En el nombre: **001And_G14V.1.fq.gz** no es necesario todo lo que va después del punto. Además, falta otra columna con el nombre (o código de los sitios). Para modificar todo esto:
 
[Codigo grep](https://github.com/verocrespoperez/InvertGenomics/blob/master/Fotos/GrepTW.jpg)

De esta forma, se genera un archivo de texto con el formato correcto:

[PopMap file](https://github.com/verocrespoperez/InvertGenomics/blob/master/Fotos/ScreenShot_PopMap.jpg)

NOTA: ¡¡¡Cuidar que los mismos sitios tengan el mismo nombre!!!
> Algunos bichos dicen "Purg" y otros "Pur"
> Se puede corregir esto con un grep:
> ... insert image

###Segundo: denovo sequencing

Se usa el programa **`denovo_map.pl`**

Código: `denovo_map.pl -M 3 -n 2 -o ./DeNovo_And_R1/ --popmap ./PopMap_And_R1.txt --samples ./Andesiops_R1`
> -o: output directory  
> --popmap: archivo popmap  
> --samples: directorio donde están las muestras  
> -M y -n son parámetros de STACKS 