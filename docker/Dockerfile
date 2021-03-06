# aerospace software
# author: Michael Shamberger

# build xfoil
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

# build avl
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

# build openvsp
FROM continuumio/anaconda3 as openvsp
RUN apt-get update
ENV TZ=Europe/Minsk
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone
RUN apt-get install -y wget cmake libxml2-dev g++ libjpeg-dev libglm-dev libeigen3-dev libcminpack-dev libglew-dev swig
WORKDIR /
RUN wget -O - https://github.com/OpenVSP/OpenVSP/tarball/OpenVSP_3.25.0 | tar xz
RUN mkdir -p /OpenVSP-OpenVSP-9b64795/SuperProject/build
RUN mkdir -p /usr/local/openvsp
WORKDIR /OpenVSP-OpenVSP-4498334/SuperProject/build
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


# install calculix
# build spooles
FROM intel/oneapi-hpckit:latest as spooles
RUN apt-get update
RUN apt-get install -y wget git
WORKDIR /tmp
RUN wget http://www.netlib.org/linalg/spooles/spooles.2.2.tgz && \
    mkdir spooles.2.2 && \
    cd spooles.2.2 && \
    tar -xvzf ../spooles.2.2.tgz
COPY calculix/spools.Make.inc /tmp/spooles.2.2/Make.inc
RUN cd spooles.2.2 && \
    make lib
# build arpack
FROM intel/oneapi-hpckit:latest as arpack
RUN apt-get update
RUN apt-get install -y wget git
WORKDIR /tmp
RUN wget https://www.caam.rice.edu/software/ARPACK/SRC/arpack96.tar.gz && \
    wget https://www.caam.rice.edu/software/ARPACK/SRC/patch.tar.gz && \
    tar -xvzf arpack96.tar.gz && \
    tar -xvzf patch.tar.gz
COPY calculix/ARmake.inc /tmp/ARPACK
RUN cd ARPACK && \
    make lib
# build calculix
FROM intel/oneapi-hpckit:latest as calculix
RUN apt-get update
RUN apt-get install -y wget git
WORKDIR /tmp
COPY --from=spooles /tmp/spooles.2.2 /tmp/spooles.2.2
COPY --from=arpack /tmp/ARPACK /tmp/ARPACK
RUN wget http://www.dhondt.de/ccx_2.17.src.tar.bz2 && \
    bunzip2 ccx_2.17.src.tar.bz2 && \
    tar xvf ccx_2.17.src.tar
COPY calculix/Makefile /tmp/CalculiX/ccx_2.17/src
RUN cd CalculiX/ccx_2.17/src && \
    make

#
# base image
#
FROM continuumio/anaconda3
EXPOSE 8888

RUN apt-get update
RUN apt-get install -y unzip wget git vim libcminpack-dev ffmpeg libsm6 libxext6

# copy xfoil binaries
COPY --from=xfoil /usr/local/bin/xfoil /usr/local/bin
COPY --from=xfoil /usr/local/bin/pplot /usr/local/bin
COPY --from=xfoil /usr/local/bin/pxplot /usr/local/bin
ENV LD_LIBRARY_PATH="/opt/conda/lib:${LD_LIBRARY_PATH}"

# copy avl binaries
COPY --from=avl /Avl/bin/avl /usr/local/bin

# install openvsp
RUN mkdir -p /usr/local/openvsp /opt/conda/lib/python3.8/site-packages/openvsp
COPY --from=openvsp /OpenVSP-OpenVSP-4498334/SuperProject/build/OpenVSP-prefix/src/OpenVSP-build/OpenVSP-3.25.0-Linux.zip /usr/local/openvsp
WORKDIR /usr/local/openvsp
RUN unzip OpenVSP-3.25.0-Linux.zip
WORKDIR /usr/local/openvsp/OpenVSP-3.25.0-Linux/python/utilities
RUN pip install .
WORKDIR /usr/local/openvsp/OpenVSP-3.25.0-Linux/python/degen_geom
RUN pip install .
WORKDIR /usr/local/openvsp/OpenVSP-3.25.0-Linux/python/openvsp
RUN pip install .
WORKDIR /usr/local/openvsp/OpenVSP-3.25.0-Linux/python/AvlPy
RUN pip install .

# install calculix and ccx2paraview
COPY --from=calculix /tmp/CalculiX/ccx_2.17/src/ccx_2.17 /usr/local/bin
RUN pip install vtk ccx2paraview

# install AeroSandbox. ADRpy aerospace libraries
COPY requirements.txt /
RUN pip install -r /requirements.txt
RUN pip install ADRpy pythreejs numpy-stl ipygany
RUN pip install git+https://github.com/opme/AeroSandbox.git@4eda7db92ea50ff283c4bfafd4eb1cffc1bd724c

# try out openaerostruct and openmdao
WORKDIR /tmp
RUN git clone https://github.com/mdolab/OpenAeroStruct.git && \
    cd OpenAeroStruct && \
    pip install -e .

# copy ipython notebook
COPY aerodesign.ipynb /
COPY sample /sample

# add command
WORKDIR /
CMD ["jupyter", "lab", "--ip='0.0.0.0'", "--port=8888", "--no-browser", "--allow-root"]
