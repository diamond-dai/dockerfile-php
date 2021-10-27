FROM golang AS builder
# alpine使うと動かなくなる、soとか関係ないはずじゃなかったっけ
# bash: /usr/local/bin/mhsendmail: No such file or directory
RUN go install github.com/mailhog/mhsendmail@latest

FROM wordpress:php7.4-apache

  # install mhsendmail
COPY --from=builder /go/bin/mhsendmail /usr/local/bin/
RUN chmod +x /usr/local/bin/mhsendmail \
  && echo 'sendmail_path = "/usr/local/bin/mhsendmail --smtp-addr=mailhog:1025"' > /usr/local/etc/php/conf.d/sendmail.ini

RUN apt update && apt install -y --no-install-recommends \
  # install mysql-client
  default-mysql-client \

  # install for pyenv
  build-essential \
  libffi-dev \
  libssl-dev \
  zlib1g-dev \
  libbz2-dev \
  libreadline-dev \
  libsqlite3-dev \

  # install util
  git \
  sudo \
  less \
  vim \
  git \
  zip \
  unzip \
  rsync \

  # cleanup
  && rm -rf /var/lib/apt/lists/* \
  && apt clean

ENV HOME /root
ENV PYENV_ROOT $HOME/.pyenv
ENV PATH $PYENV_ROOT/bin:$PATH

# Download pyenv
RUN git clone https://github.com/pyenv/pyenv.git ~/.pyenv \
  && echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc \
  && echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc \
  && echo 'eval "$(pyenv init --path)"' >> ~/.bashrc \
  && . ~/.bashrc \
  && PYTHON_VER=$(pyenv install --list | grep -v - | grep -v b | grep -v a | grep -v rc | tail -1) \
  && pyenv install $PYTHON_VER \
  && pyenv global $PYTHON_VER

# enable modules
RUN a2enmod rewrite \
  &&  a2enmod include \
  &&  a2enmod headers

# Countermeasures against events that cache does not disappear
RUN  sh -c "echo 'EnableSendfile off' >> /etc/apache2/apache2.conf" \

  # install wp-cli
  && curl -O https://raw.githubusercontent.com/wp-cli/builds/gh-pages/phar/wp-cli.phar \
  && chmod +x wp-cli.phar \
  && mv wp-cli.phar /usr/local/bin/wp \
  && mkdir -p /var/www/.wp-cli/cache \
  && chown -R www-data /var/www/.wp-cli \
  && mkdir /var/www/.composer/ \
  && chown -R www-data /var/www/.composer \

  # install composer
  && curl -sS https://getcomposer.org/installer | php \
  && mv composer.phar /usr/local/bin/composer

# keep env
RUN sed -i -e 's/^Defaults.*env_reset/Defaults env_keep \+= \"PATH\"/g' /etc/sudoers

# add php conf
RUN  echo '\
  log_errors = On\n\
  error_log = /proc/self/fd/2\n\
  error_reporting = E_ALL\n\
  ' >> /usr/local/etc/php/conf.d/debug.ini
