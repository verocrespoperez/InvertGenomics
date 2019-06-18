#Git Hub  

Para empezar a modificar un archivo de Marcdown hay que ir primero al directorio donde esta la carpeta ligada al git. En nuestro caso la carpeta se llama **InvertGenomics**, entonces el código sería:

> `cd /Users/Vero/Documents/BaseSpace/JA18493-109360251/ddRadMacros/InvertGenomics`

##**Notas importantes:**

Después de hacer cambios dentro de la carpeta de git (sean creación de archivos o modificaciones al interior de los archivos) se deben seguir los siguientes pasos: ("status", "add", "commit" y "push").

Ejemplos:
> git status  

Indica si han habido cambios en la carpeta.

> git add README.md  

Agrega los cambios (en este cambio "README.md" es el nombre de un archivo en la carpeta).

> git commit -m 'aqui explicar el cambio que se hizo'  

Confirma los cambios

> git push origin master 

Sube los cambios al sitio web de git.
<br/><br/>
**Para borrar archivos del git:**  

> git rm example.md
> git commit -m 'file example.md removed'

This erases the file from git AND from the filesystem.

> git rm --cached example.md
> git commit -m 'file example.md removed'

This erases the file from git BUT NOT from the filesystem.

THEN remember tu **push** the changes to the remote repo!!!
> git push origin master
<br/><br/>

__Para subir imágenes a GitHub__

>git add "archivo"  
>git commit -m 'mensaje'  
>git push origin master

Esto hace que el archivo se suba al git. Se debe copiar el URL de la ubicación de la imagen en git y esta direccion copiar en el codigo de git (eg. en el README.md). Ejemplo:

`![Codigo grep](https://github.com/verocrespoperez/InvertGenomics/blob/master/Fotos/GrepTW.jpg)`


