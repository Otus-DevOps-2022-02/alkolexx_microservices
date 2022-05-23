FROM ubuntu:16.04

ENV APP_HOME /app

RUN apt-get update \
    && apt-get install -y ruby-full ruby-dev build-essential \
    && gem install bundler --no-ri --no-rdoc \
    && mkdir $APP_HOME \
    && rm -rf /var/lib/apt/*

WORKDIR $APP_HOME
ADD . $APP_HOME
RUN bundle install

ENV POST_SERVICE_HOST post
ENV POST_SERVICE_PORT 5000
ENV COMMENT_SERVICE_HOST comment
ENV COMMENT_SERVICE_PORT 9292

CMD ["puma"]
