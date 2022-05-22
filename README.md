# ** HW-13 - DOCKER-3 **

- Научился запускать связанные контейнеры
- Возникли проблемы с созданием контейнеров, исправилось следующим образом:
1. /comment/Dockerfile
```
RUN apt-get update -qq && \
    apt-get install -y build-essential && \
    rm -rf /var/lib/apt/* && \
    mkdir $APP_HOME
```
2. /post-py/Dockerfile
```
RUN apk --no-cache --update add build-base && \
    pip install --upgrade pip && \
    pip install -r /app/requirements.txt && \
    apk del build-base
```
3. /ui/Dockerfile
```
RUN apt-get update \
    && apt-get install -y ruby-full ruby-dev build-essential \
    && gem install bundler --no-ri --no-rdoc \
    && mkdir $APP_HOME \
    && rm -rf /var/lib/apt/*
```
