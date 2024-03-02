# Algorithmic Tales Website

To build the website use 

```
hugo 
```

to serve locally and see posts with drafts use 

```
hugo serve -D
```

## Deployment

rsync the `/public` files with 

```
rsync -avz --delete public/ <user>@<server>:/var/www/algorithmictales.com
```