FROM elasticsearch:7.17.9
RUN echo "installing analysis-icu, analysis-kuromoji, analyisis-smartcn, analysis-stempel..."
RUN /usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-icu &&  \
    /usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-kuromoji && \
    /usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-smartcn && \
    /usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-stempel
