# cyclopy
Un blog mais aussi et surtout un memo sur le langage Python mais aussi l'IT de façon plus générale.


## Running pelican on localhost

```sh
pelican --listen
```

## Update content

```sh
git add .
git commit -m "Your message"
git push origin content
```

## Publish new version of the blog

```sh
make html
ghp-import output
git push origin gh-pages:master
```