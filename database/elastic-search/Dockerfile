# Sử dụng base image của Elasticsearch
FROM docker.elastic.co/elasticsearch/elasticsearch:8.10.2 as builder

USER root
ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update -y && apt-get install -y software-properties-common build-essential
RUN gcc --version
RUN apt-get update -y && apt-get install -y make cmake pkg-config wget git

ENV JAVA_HOME=/usr/share/elasticsearch/jdk
ENV PATH=$JAVA_HOME/bin:$PATH

# Build coccoc-tokenizer
RUN echo "Build coccoc-tokenizer..."
WORKDIR /tmp
RUN git clone https://github.com/duydo/coccoc-tokenizer.git
RUN mkdir /tmp/coccoc-tokenizer/build
WORKDIR /tmp/coccoc-tokenizer/build
RUN cmake -DBUILD_JAVA=1 ..
RUN make install

# Build analysis-vietnamese
RUN echo "analysis-vietnamese..."
WORKDIR /tmp
RUN wget https://dlcdn.apache.org/maven/maven-3/3.8.8/binaries/apache-maven-3.8.8-bin.tar.gz \
    && tar xvf apache-maven-3.8.8-bin.tar.gz
ENV MVN_HOME=/tmp/apache-maven-3.8.8
ENV PATH=$MVN_HOME/bin:$PATH

COPY ./elasticsearch-analysis-vietnamese /tmp/elasticsearch-analysis-vietnamese
WORKDIR /tmp/elasticsearch-analysis-vietnamese
RUN mvn verify clean --fail-never
RUN mvn --batch-mode -Dmaven.test.skip -e package


FROM docker.elastic.co/elasticsearch/elasticsearch:8.10.2

ARG COCCOC_INSTALL_PATH=/usr/local
ARG COCCOC_DICT_PATH=$COCCOC_INSTALL_PATH/share/tokenizer/dicts

COPY --from=builder $COCCOC_INSTALL_PATH/lib/libcoccoc_tokenizer_jni.so /usr/lib
COPY --from=builder $COCCOC_DICT_PATH $COCCOC_DICT_PATH
COPY --from=builder /tmp/elasticsearch-analysis-vietnamese/target/releases/elasticsearch-analysis-vietnamese-8.10.2.zip /
RUN echo "Y" | elasticsearch-plugin install file:///elasticsearch-analysis-vietnamese-8.10.2.zip
Run elasticsearch-plugin install analysis-icu

# Expose cổng mặc định của Elasticsearch
EXPOSE 9200 9300

# Chạy Elasticsearch khi container được khởi chạy
CMD ["elasticsearch"]



# # Sử dụng base image của Elasticsearch
# FROM docker.elastic.co/elasticsearch/elasticsearch:8.10.2

# Run elasticsearch-plugin install analysis-icu

# # Expose cổng mặc định của Elasticsearch
# EXPOSE 9200 9300

# # Chạy Elasticsearch khi container được khởi chạy
# CMD ["elasticsearch"]