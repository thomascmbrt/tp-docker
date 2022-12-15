# Docker et Docker Compose TP2

1) done

2) the `npm install` allow to install dependencies packages and the option `--production` allow to install only production dependencies because sometime we need some dev dependencies like linter or other for the comfort and security development. The goodly practice of docker is to be as much as possible light, so we don't need to install dev dependencies on a production environment.

3) The command to build our image : 
```bash 
docker build -t 'ma_super_app' .
```

