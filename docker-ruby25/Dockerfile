FROM ruby:2.5

RUN apt-get -y update

# Create an user with the same uid/gid as the user running docker-compose in development to avid permissions conflicts
ARG uid=1000
ARG gid=1000
RUN addgroup --gid $gid ruby25
RUN useradd -m --uid $uid -g ruby25 ruby25

WORKDIR /app
USER ruby25
