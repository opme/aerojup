# install xfoil  (taken from https://github.com/thomaseizinger/docker-xfoil/)
FROM intel/oneapi-hpckit:latest as xfoil
RUN apt-get update
RUN apt-get install -y wget
RUN apt-get install -y git
RUN apt-get install -y libx11-dev
RUN wget https://web.mit.edu/drela/Public/web/xfoil/xfoil6.99.tgz -O /tmp/xfoil6.99.tgz
RUN tar -xzvf /tmp/xfoil6.99.tgz -C /
WORKDIR /Xfoil
COPY xfoil.patch .
RUN git apply xfoil.patch
WORKDIR /Xfoil/orrs/bin
RUN make osgen
RUN make osmap.o
WORKDIR /Xfoil/orrs
RUN bin/osgen osmaps_ns.lst
WORKDIR /Xfoil/plotlib
RUN make libPlt_gSP.a
WORKDIR /Xfoil/bin
RUN make xfoil
RUN make pplot
RUN make pxplot
WORKDIR /Xfoil

# install avl
FROM intel/oneapi-hpckit:latest as avl
RUN apt-get update
RUN apt-get install -y wget
RUN apt-get install -y git
RUN apt-get install -y libx11-dev
RUN wget https://web.mit.edu/drela/Public/web/avl/avl3.36.tgz -O /tmp/avl3.36.tgz
RUN tar -xzvf /tmp/avl3.36.tgz -C /
WORKDIR /Avl
RUN cp eispack/Makefile.ifort eispack/Makefile
RUN cp bin/Makefile.ifort bin/Makefile
WORKDIR /Avl/plotlib
RUN make ifort
WORKDIR /Avl/eispack
RUN make
WORKDIR /Avl/bin
RUN make

FROM continuumio/anaconda3 as openvsp
RUN apt-get update
ENV TZ=Europe/Minsk
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN apt-get install -y wget cmake libxml2-dev g++ libjpeg-dev libglm-dev libeigen3-dev libcminpack-dev libglew-dev swig
WORKDIR /
RUN wget -O - https://github.com/OpenVSP/OpenVSP/tarball/OpenVSP_3.24.0 | tar xz
RUN mkdir -p /OpenVSP-OpenVSP-9b64795/SuperProject/build
RUN mkdir -p /usr/local/openvsp
WORKDIR /OpenVSP-OpenVSP-9b64795/SuperProject/build
RUN cmake .. \
        -DCMAKE_INSTALL_PREFIX=/usr/local/openvsp \
        -DVSP_NO_GRAPHICS=true \
        -DCMAKE_BUILD_TYPE=Release \
        -DCMAKE_PREFIX_PATH='/usr' \
        -DVSP_USE_SYSTEM_CPPTEST=false \
        -DVSP_USE_SYSTEM_LIBXML2=true \
        -DVSP_USE_SYSTEM_EIGEN=true \
        -DVSP_USE_SYSTEM_CODEELI=false \
        -DVSP_USE_SYSTEM_FLTK=false \
        -DVSP_USE_SYSTEM_GLM=true \
        -DVSP_USE_SYSTEM_GLEW=true \
        -DVSP_USE_SYSTEM_CMINPACK=true \
        -DPYTHON_EXECUTABLE='/opt/conda/bin/python' \
        -DPYTHON_LIBRARY='/opt/conda/lib' \
        -DPYTHON_INCLUDE_DIR='/opt/conda/include/python3.8' \
        -DPYTHON_INCLUDE_PATH='/opt/conda/include'
RUN make

#
# base image
#
FROM continuumio/anaconda3
EXPOSE 8888

RUN apt-get update
RUN apt-get install -y unzip libcminpack-dev

# copy xfoil binaries
COPY --from=xfoil /usr/local/bin/xfoil /usr/bin
COPY --from=xfoil /usr/local/bin/pplot /usr/bin
COPY --from=xfoil /usr/local/bin/pxplot /usr/bin
ENV LD_LIBRARY_PATH="/opt/conda/lib:${LD_LIBRARY_PATH}"

# install openvsp
RUN mkdir -p /usr/local/openvsp /opt/conda/lib/python3.8/site-packages/openvsp
COPY --from=openvsp /OpenVSP-OpenVSP-9b64795/SuperProject/build/OpenVSP-prefix/src/OpenVSP-build/OpenVSP-3.24.0-Linux.zip /usr/local/openvsp
WORKDIR /usr/local/openvsp
RUN unzip OpenVSP-3.24.0-Linux.zip
WORKDIR /usr/local/openvsp/OpenVSP-3.24.0-Linux/python/utilities
RUN pip install .
WORKDIR /usr/local/openvsp/OpenVSP-3.24.0-Linux/python/degen_geom
RUN pip install .
WORKDIR /usr/local/openvsp/OpenVSP-3.24.0-Linux/python/openvsp
RUN pip install .
WORKDIR /usr/local/openvsp/OpenVSP-3.24.0-Linux/python/AvlPy
RUN pip install .

# install AeroSandbox. ADRpy aerospace libraries
RUN pip install AeroSandbox ADRpy pythreejs numpy-stl

# copy ipython notebook
COPY aerodesign.ipynb /
COPY sample /sample

# add command
WORKDIR /
CMD ["jupyter", "lab", "--ip='0.0.0.0'", "--port=8888", "--no-browser", "--allow-root"]
