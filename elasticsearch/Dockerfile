FROM elasticsearch:2.3.3

WORKDIR /usr/share/elasticsearch

RUN gosu elasticsearch bin/plugin install analysis-icu \
	&& bin/plugin install https://github.com/elastic/elasticsearch-migration/releases/download/v2.0-alpha2/elasticsearch-migration-2.0-alpha2.zip \
	&& gosu elasticsearch bin/plugin install license \
	&& gosu elasticsearch bin/plugin install marvel-agent

COPY elasticsearch.yml config
