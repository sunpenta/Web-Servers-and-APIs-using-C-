FROM  gcc:12.3

RUN apt-get -qq update
RUN apt-get -qq upgrade
RUN apt-get -qq install cmake

RUN apt-get install -y libboost-all-dev --no-install-recommends
RUN apt-get -qq install build-essential libtcmalloc-minimal4 && \ 
    ln -s /usr/lib/libtcmalloc_minimal.so.4 /usr/lib/libtcmalloc_minimal.so