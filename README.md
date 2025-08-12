# Writeups de Hacking

## Customización

Las principales customizaciones del blog se encuentran en el archivo `_config.yml` donde se puede editar el título, la descripción, Google Analytics y si se desea mostrar un enlace de descarga del sitio.

Para customización de layout en general se puede editar archivo `_layouts/default.html`. 

Para customización de head se puede editar el archivo `_includes/head-custom.html`.

## Estructura del repositorio
```
hacking-writeups/
├── _config.yml              <-- Documentaciones técnicas específicas
├── _layouts/default.html    <-- API REST, script de base de datos y tests
├── images/                  <-- Imágenes utilizadas en los posts
├── blog/                    <-- Posts random de lo que aprendí, temas especificos de hacking, etc
├── writeups/
    ├── tryhackme/           <-- Writeups de TryHackMe
    ├── hackthebox/          <-- Writeups de HackTheBox
├── index.html               <-- Página principal del blog, redirección a los posts
├── README.md                <-- Este archivo con guía general y enlaces útiles
├── .gitignore
```

(En vez de utilizar imagenes capturas de pantalla, puedo utilizar markdown copiar y pegar resultado de terminal).

## Filosofía

Hacer writeups para aprender y registrar lo aprendido, y también podría llegar a ser útil para otros.

### Preview local
1. Clonar repositorio
2. `cd` al directorio del repositorio
3. Ejecutar `bundle install --verbose`
4. Iniciar servidor `jekyll serve`
5. Visitar [`localhost:4000`](http://localhost:4000) 


## Mención

Mención especial a los creadores del tema Hacker de GitHub Pages, que me han servido de inspiración para crear este sitio. El tema original se puede encontrar en [github.com/pages-themes/hacker](https://github.com/pages-themes/hacker).

