FROM docker.elastic.co/elasticsearch/elasticsearch:8.19.0

# Install required Liferay plugins for Elasticsearch 8.19.0
RUN /usr/share/elasticsearch/bin/elasticsearch-plugin install --batch analysis-icu &&  \
    /usr/share/elasticsearch/bin/elasticsearch-plugin install --batch analysis-kuromoji && \
    /usr/share/elasticsearch/bin/elasticsearch-plugin install --batch analysis-smartcn && \
    /usr/share/elasticsearch/bin/elasticsearch-plugin install --batch analysis-stempel
