# Use Ubuntu 16.04 as the base image
FROM ubuntu:16.04

# Set environment variable to avoid interactive prompts during installation
ENV DEBIAN_FRONTEND=noninteractive

# Update package manager and install dependencies
RUN apt-get update && \
    apt-get install -y \
    wget \
    curl \
    build-essential \
    zlib1g-dev \
    libncurses5-dev \
    libgdbm-dev \
    libnss3-dev \
    libssl-dev \
    libreadline-dev \
    libffi-dev \
    libbz2-dev \
    openjdk-8-jdk \
    scala \
    git

# Set JAVA_HOME environment variable
ENV JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
ENV PATH=$JAVA_HOME/bin:$PATH

# Download and install Scala
RUN wget https://downloads.lightbend.com/scala/2.11.12/scala-2.11.12.deb && \
    dpkg -i scala-2.11.12.deb && \
    rm scala-2.11.12.deb

# Download and compile Python 3.4.10
RUN cd /usr/src && \
    wget https://www.python.org/ftp/python/3.4.10/Python-3.4.10.tgz && \
    tar xzf Python-3.4.10.tgz && \
    cd Python-3.4.10 && \
    ./configure --enable-optimizations && \
    make altinstall && \
    ln -s /usr/local/bin/python3.4 /usr/bin/python3.4 && \
    cd .. && \
    rm -rf Python-3.4.10 Python-3.4.10.tgz

# Download and install Apache Spark 2.4.4
RUN mkdir -p /spark-2.4.4 && \
    wget https://archive.apache.org/dist/spark/spark-2.4.4/spark-2.4.4-bin-hadoop2.7.tgz && \
    tar xzf spark-2.4.4-bin-hadoop2.7.tgz -C /spark-2.4.4 --strip-components=1 && \
    rm spark-2.4.4-bin-hadoop2.7.tgz


# Create /spark-2.4.4/reproduction_files directory and add files
RUN mkdir -p /spark-2.4.4/reproduction_files && \
    echo "id,name,age\n1,John,25\n2,Alice,30\n3,Bob,22" > /spark-2.4.4/reproduction_files/test_data.csv && \
    echo "import time\nimport os\n\nfrom pyspark.sql import SparkSession\nfrom pyspark.sql import functions as F\nfrom pyspark.sql import types as T\n\ntarget_dir = './reproduction_files/'\n\nspark = SparkSession.builder.appName('DataframeCount').getOrCreate()\n\nwhile True:\n    for f in os.listdir(target_dir):\n        df = spark.read.load(target_dir + f, format='csv')\n        print('Number of records: {0}'.format(df.count()))\n    time.sleep(15)" > /spark-2.4.4/reproduction_files/count_file.py

RUN echo "./bin/spark-submit --master local[*] --deploy-mode client --executor-memory 4g --executor-cores 3 ./reproduction_files/count_file.py" > /spark-2.4.4/temp.sh && \
    chmod +x /spark-2.4.4/temp.sh

# Set environment variables for Spark and PySpark
ENV SPARK_HOME=/spark-2.4.4
ENV PATH=$SPARK_HOME/bin:$PATH
ENV PYSPARK_PYTHON=python3.4
ENV PYSPARK_DRIVER_PYTHON=python3.4

# Set working directory
WORKDIR /spark-2.4.4

# Expose Spark Web UI port
EXPOSE 4040

# Set default command when container starts
CMD ["bash"]

