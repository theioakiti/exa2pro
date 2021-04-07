Bootstrap: docker
From: ubuntu:18.04

%setup
   # make directory for test MPI program
   mkdir ${SINGULARITY_ROOTFS}/mpitestapp
   cp mpitest.F90 ${SINGULARITY_ROOTFS}/mpitestapp/

%post
	apt-get update \
	&& apt-get install -y wget gnupg \
	&& wget http://developer.download.nvidia.com/compute/cuda/repos/ubuntu1710/x86_64/cuda-repo-ubuntu1710_9.2.148-1_amd64.deb \
	&& dpkg -i cuda-repo-ubuntu1710_9.2.148-1_amd64.deb \
	&& apt-key adv --fetch-keys https://developer.download.nvidia.com/compute/cuda/repos/ubuntu1710/x86_64/7fa2af80.pub \
	&& rm -f cuda-repo-ubuntu1710_9.2.148-1_amd64.deb \
	&& apt-get update \
	&& apt-get install -y \
		autoconf \
		automake \
		bison \
		cuda-command-line-tools-9-2 \
		cuda-compiler-9-2 \
		cuda-libraries-9-2 \
		cuda-libraries-dev-9-2 \
		opencl-headers \
		flex \
		git \
		gcc \
		gcc-6 \
		gfortran \
		gfortran-6 \
		gperf \
		libhwloc-dev \
		libsqlite3-dev \
		libtool \
		libxerces-c-dev \
		make \
		pkg-config \
		python3 \
		python \
		vim \
		libfxt-dev \
		libsimgrid-dev \
		libboost-dev \
	&& rm -Rf /var/lib/apt/lists/* \
	&& mkdir /usr/local/cuda-9.2/lib \
	&& ln -s /usr/local/cuda-9.2/lib64 /usr/local/cuda-9.2/lib/x86_64

	# install dependencies for miniKKR (blas, lapack, papi-tools)
	apt-get update
	apt-get install -y libblas-dev
	apt-get install -y liblapack-dev liblapack-doc
	apt-get install -y papi-tools
	apt-get install -y libpapi-dev

	# install dependencies for co2_capture
	apt-get -y install --no-install-recommends build-essential patch libopenblas-dev 

	# install libtool-bin (needed for starpu gitlab master branch)
	apt-get update
	apt-get dist-upgrade -y
	apt-get update
	apt-get install -y libtool-bin

	# download, build and install CMake
	cd /tmp
        wget https://github.com/Kitware/CMake/releases/download/v3.15.6/cmake-3.15.6.tar.gz -P /tmp
        tar -xzf /tmp/cmake-3.15.6.tar.gz -C /tmp
        cd /tmp/cmake-3.15.6
        ./bootstrap --prefix=/usr/local
        make -j4
        make install
        cd /tmp
	rm ./cmake-3.15.6.tar.gz
	rm -rf ./cmake-3.15.6
	
    # download, build and install Catch2
    git clone https://github.com/catchorg/Catch2.git
    cd ./Catch2
    git checkout fd9f5ac661f87335ecd70d39849c1d3a90f1c64d # v2.13.1
    cmake -Bbuild -H. -DBUILD_TESTING=OFF
    cmake --build build/ --target install
    cd /tmp
    rm -rf ./Catch2

    # Install mpich 3.1.4
    wget -q http://www.mpich.org/static/downloads/3.1.4/mpich-3.1.4.tar.gz
    tar xf mpich-3.1.4.tar.gz
    cd mpich-3.1.4
    ./configure --enable-fast=all,O3 --prefix=/usr --disable-wrapper-rpath
    make
    make install
    ldconfig
    cd /mpitestapp
    mpifort mpitest.F90 -o mpitest
    cd /tmp

	# # download, build and install StarPU
	# wget http://starpu.gforge.inria.fr/files/starpu-1.3.4/starpu-1.3.4.tar.gz -P /tmp
	# tar -xzf /tmp/starpu-1.3.4.tar.gz -C /tmp
	# cd /tmp/starpu-1.3.4
	# ./configure \
	# 		--with-fxt \
	# 		--disable-build-examples \
	# 		--disable-build-tests \
	# 		--disable-build-doc 
	# make -j4
	# make install
	# cd /tmp
	# rm -rf starpu-1.3.4

	# download, build and install StarPU (from exa2pro gitlab repo)
	git clone https://oauth2:3bHxZoCBz8hC8QJKz4u7@gitlab.seis.exa2pro.iti.gr/exa2pro/starpu.git
	cd ./starpu
	git checkout master
	./autogen.sh
	./configure --with-fxt --disable-build-examples --disable-build-tests --disable-build-doc
	make -j4
	make install
	cd /tmp
	rm -rf ./starpu

	# download, build and install SkePU
	git clone --recursive https://oauth2:3bHxZoCBz8hC8QJKz4u7@gitlab.seis.exa2pro.iti.gr/exa2pro/skepu-clang.git
	cd ./skepu-clang
	mkdir build && cd build
 	cmake -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=/opt/skepu-clang ..
 	MAKEFLAGS=-j4 make
 	make install
 	cd /tmp
	rm -rf ./skepu-clang

    # download, build and install XPDL
    export XERCES_HOME=/usr/local
    export XPDL_HOME=/opt/xpdl
    export PATH=/opt/xpdl/bin:/opt/compu/bin:$PATH:/usr/local/cuda-9.2/bin
    git clone https://oauth2:3bHxZoCBz8hC8QJKz4u7@gitlab.seis.exa2pro.iti.gr/exa2pro/xpdl.git
    cp -r xpdl /opt
    cd /opt/xpdl
    cd src && MAKEFLAGS=-j4 make && xpdl_compiler ../example/system.xml
    cd /tmp
    rm -rf xpdl

    # download, build and install ComPU
    export STARPU_HOME=/usr/local/
    export CT_HOME=/opt/compu
    git clone https://oauth2:3bHxZoCBz8hC8QJKz4u7@gitlab.seis.exa2pro.iti.gr/exa2pro/compu.git
    cp -r compu /opt
    cd /opt/compu/src \
		&& MAKEFLAGS=-j4 make \
		&& install -Dm755 compose /opt/compu/bin/compose
	cd /tmp
    rm -rf compu

    # download, build and install skepu-mcxx
    export PATH=/opt/xpdl/bin:/opt/compu/bin:/opt/skepu-mcxx/bin:$PATH:/usr/local/cuda-9.2/bin
    export MERCURIUM=/opt/skepu-mcxx
    git clone https://oauth2:3bHxZoCBz8hC8QJKz4u7@gitlab.seis.exa2pro.iti.gr/exa2pro/skepu-mcxx.git
    cd skepu-mcxx
    autoreconf -fiv
    ./configure --prefix=$MERCURIUM
    make
    make install
    cd /tmp
    rm -rf skepu-mcxx

    # install maven
    apt update
    apt -y install default-jdk
    apt -y install maven

    # download, build and install TDM Tool
    export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
    git clone https://oauth2:3bHxZoCBz8hC8QJKz4u7@gitlab.seis.exa2pro.iti.gr/exa2pro/Exa2Pro.git
    cd Exa2Pro
    mvn clean package
    cd /opt
    mkdir tdm
    cd tdm
    mv /tmp/Exa2Pro/target/TDmanagement-1.0.0-SNAPSHOT-jar-with-dependencies.jar .
    cp -r /tmp/Exa2Pro/sonar-icode-cnes-plugin-1.3.0 .
    cp -r /tmp/Exa2Pro/sonar-scanner-4.2-linux .
    cp -r /tmp/Exa2Pro/td-forecaster .
    cp -r /tmp/Exa2Pro/clustering .
    cd /tmp
    rm -rf Exa2Pro

    # install dependencies for TD Forecaster
    apt install -y python3-pip
    pip3 install numpy==1.18.1
    pip3 install pandas==1.0.3
    pip3 install scikit-learn==0.22.1

    # # download, build and install LQCD
    # #cd /tmp
    # git clone https://oauth2:3bHxZoCBz8hC8QJKz4u7@gitlab.seis.exa2pro.iti.gr/exa2pro/t6.1-lattice-qcd.git
    # cp -r t6.1-lattice-qcd /opt
    # cd /opt/t6.1-lattice-qcd
    # mkdir build && cd build
    # cmake ../src
    # make && make install

%runscript
    /mpitestapp/mpitest

# %test
#     cd /tmp/starpu-1.3.4
#     find . -type f -name Makefile -exec sed -i "s:MPIEXEC_ARGS = :MPIEXEC_ARGS = --allow-run-as-root:" {} \;
#     make check

%environment
	export XERCES_HOME=/usr/local
    export XPDL_HOME=/opt/xpdl
    export PATH=/opt/xpdl/bin:/opt/compu/bin:$PATH:/usr/local/cuda-9.2/bin
    export STARPU_HOME=/usr/local/
    export CT_HOME=/opt/compu
    export PATH=/opt/xpdl/bin:/opt/compu/bin:/opt/skepu-mcxx/bin:$PATH:/usr/local/cuda-9.2/bin
    export MERCURIUM=/opt/skepu-mcxx
    export JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64

%help
	An Ubuntu 18.04 container with full StarPU, SkePU, ComPU and XPDL installations. Package list:
	autoconf (GNU Autoconf) 2.69
	automake (GNU automake) 1.15.1
	bison (GNU Bison) 3.0.4
	cmake version 3.15.6
	Catch2 v2.13.1
	flex 2.6.4
	git version 2.17.1
	GNU gperf 3.1
	GNU patch 2.7.6
	libhwloc-dev
	mpich 3.1.4
	GNU gcc, g++, gfortran 7.5.0
	GNU Make 4.1
	Python 2.7.17, 3.6.9
	libblas-dev 3.7.1-4ubuntu1
	libxerces-c-dev 3.2.0+debian-2
	libhwloc-dev 1.11.9-1
	liblapack-dev 3.7.1-4ubuntu1
	libpapi-dev 5.6.0-1
	libopenblas-dev 0.2.20+ds-4
	StarPU commit 351a134faccbe1ff41cacc258fc4730056d7b713
	SkePU 3 commit ff9c573b27e382ed4521fed3d999c7f6f74aba46
	XPDL commit 4ff00a4870d3ca5b4c85b618e7bffbe28933584a
	ComPU commit 567bc926149d257306bb2cdbd076d7fcc2f56a3f
	Mercurium commit 4e17881b3c2a56109486b7875185272e2b71a77d
	TDM Tool commit 2f7f72d598b0cda012f743ad1cfd88a9e6b9a484
	In order to run a use case application do the following:
	1. Shell into the container (singularity shell exa2pro_mpich.sif)
	2. Navigate to use case application directory
	3. Build and test the app as you would normally do on a system you are SSHed into
	Note: if the use case application directory is located outside your $HOME directory, you should also bind-mount that directory when shelling into the container
	e.g. singularity shell --bind /path/to/application/directory:/path/to/create/application/directory/inside/container exa2pro_mpich.sif
	In order to check if the container runs properly, you can run the following:
	singularity run exa2pro_mpich.sif
