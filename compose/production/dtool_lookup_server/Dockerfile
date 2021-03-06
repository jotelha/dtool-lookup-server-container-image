FROM python:3.8-slim

ENV DTOOL_LOOKUP_SERVER_VERSION=0.16.0
ENV DTOOL_LOOKUP_SERVER_DEPENDENCY_GRAPH_PLUGIN_VERSION=0.1.4
ENV DTOOL_LOOKUP_SERVER_DIRECT_MONGO_PLUGIN_VERSION=0.1.3

ENV PYTHONUNBUFFERED 1

RUN apt-get --yes update && \
    apt-get --no-install-recommends --yes install gcc libc6-dev libpq-dev supervisor cron && \
    pip install gunicorn psycopg2 && \
    pip install importlib_metadata  && \
    pip install dtool-lookup-server==$DTOOL_LOOKUP_SERVER_VERSION \
                dtool-lookup-server-dependency-graph-plugin==$DTOOL_LOOKUP_SERVER_DEPENDENCY_GRAPH_PLUGIN_VERSION \
                dtool-lookup-server-direct-mongo-plugin==$DTOOL_LOOKUP_SERVER_DIRECT_MONGO_PLUGIN_VERSION \
                dtool-smb && \
    apt-get remove --purge --autoremove -y gcc && \
    rm -rf /var/lib/apt/lists/*
# remove importlib_metadata when https://github.com/IMTEK-Simulation/dtool-lookup-server-direct-mongo-plugin/issues/1 resolved

#
# Prepare application code
#
RUN mkdir /app
RUN mkdir -p /root/.config/dtool
COPY dtool/dtool.json /root/.config/dtool

COPY compose/production/dtool_lookup_server/wsgi.py /app
COPY compose/production/dtool_lookup_server/start /start
COPY compose/production/dtool_lookup_server/init_db /init_db
COPY compose/production/dtool_lookup_server/register_users_and_base_uris /register_users_and_base_uris
COPY compose/production/dtool_lookup_server/refresh_index /refresh_index
COPY compose/production/dtool_lookup_server/refresh_index /etc/cron.daily/refresh_index
COPY compose/production/dtool_lookup_server/supervisord.conf /etc/supervisor/conf.d/supervisord.conf
# cron job needs "sourcible" env vars definition within container. However, the docker-style
# file_env does not treat quotes the same way as the shell. Therefore, we copy the env_file
# into the container to merely mark all environment variables to be defined via the container
# compose mechanism. Their actual values at image build time won't be taken into account.
# Instead, actual values are read at container launch (see start file):
COPY envs/production/dtool_lookup_server /refresh_index.envs.reference
RUN chmod 0744 /etc/cron.daily/refresh_index
RUN chmod +x /init_db
RUN chmod +x /register_users_and_base_uris
RUN chmod +x /refresh_index
RUN sed -i 's/\r//' /start
RUN chmod +x /start

WORKDIR /app
