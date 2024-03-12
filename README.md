# build Opticks
We want to build opticks with exactly the same tools that we use to build artg4tk later on. 
```bash
cat > test.sh <<EOF
export WORK_DIR=\${PWD}
source /cvmfs/larsoft.opensciencegrid.org/products/setup
setup larsoft v09_74_00 -qe20:prof
unsetup geant4
setup geant4 v4_11_1_p01ba -q e20:prof
setup ninja v1_11_0
setup cmake v3_27_4
git clone https://github.com/simoncblyth/opticks.git
cd opticks
git tag
git checkout v0.2.7
mkdir -p \${WORK_DIR}/local/opticks/externals/
cd \${WORK_DIR}/local/opticks/externals/
ln -s \${OptiX_INSTALL_DIR} OptiX
cd \${WORK_DIR}

EOF
```
create a setup script for Opticks


```bash   
cat > setup_opticks.sh << EOF
#------------------------------------------------------------------------------------
# to optionally enable logging in relevant classes uncomment the following lines
#export SEvt=INFO
#export G4CXOpticks=INFO
#export SEventConfig=INFO
#export CSGOptiX=INFO
#export SSim__stree_level=2
#------------------------------------------------------------------------------------
export WORK_DIR=\${PWD}
source /cvmfs/larsoft.opensciencegrid.org/products/setup
setup larsoft v09_74_00 -qe20:prof
unsetup geant4
setup geant4 v4_11_1_p01ba -q e20:prof
setup ninja v1_11_0
setup cmake v3_27_4
export OPTICKS_HOME=\${PWD}/opticks 
opticks-(){ . \${OPTICKS_HOME}/opticks.bash && opticks-env \$* ; }
#op(){ op.sh \$* ; }
#o(){ cd \$(opticks-home) ; hg st ; }
_path_prepend() {
if [ -n "\$2" ]; then
case ":\$(eval "echo \\$\$1"):" in
*":\$2:"*) :;;
*) eval "export \$1=\$2\${\$1:+\":\$\$1\"}" ;;
esac
else
case ":\$PATH:" in
*":\$1:"*) :;;
*) export PATH="\$1\${PATH:+":\$PATH"}" ;;
esac
fi
}
_path_append() {
if [ -n "\$2" ]; then
case ":\$(eval "echo \\$\$1"):" in
*":\$2:"*) :;;
*) eval "export $1=\${\$1:+\"\$\$1:\"}$2" ;;
esac
else
case ":\$PATH:" in
*":\$1:"*) :;;
*) export PATH="\${PATH:+"\$PATH:"}\$1" ;;
esac
fi
}
export CUDA_INSTALL_DIR=/opt/nvidia/hpc_sdk/Linux_x86_64/23.3/cuda/11.8
#export PATH=\${CUDA_INSTALL_DIR}/bin:\$PATH
_path_prepend "\${CUDA_INSTALL_DIR}/bin"
_path_prepend "."


# make sure you set OptiX_INSTALL_DIR,OPTICKS_COMPUTE_CAPABILITY,CUDA_INSTALL_DIR correctly for your system 
export OptiX_INSTALL_DIR=/home/wenzel/NVIDIA-OptiX-SDK-7.5.0-linux64-x86_64
export OPTICKS_COMPUTE_CAPABILITY=75
export CUDA_SAMPLES=/data/software/cuda-samples
export G4INSTALL=/cvmfs/larsoft.opensciencegrid.org/products/geant4/v4_11_1_p01ba/Linux64bit+3.10-2.17-e20-prof
export LOCAL_BASE=\${WORK_DIR}/local
export CMAKE_PREFIX_PATH=/cvmfs/larsoft.opensciencegrid.org/products/boost/v1_80_0/Linux64bit+3.10-2.17-e20-prof/:\${G4INSTALL}:\${LOCAL_BASE}/opticks/externals:\${OptiX_INSTALL_DIR}:\${WORK_DIR}/opticks/cmake/Modules/:\${WORK_DIR}/local/opticks:\${WORK_DIR}/local/opticks:\${WORK_DIR}/local/opticks/externals/
#:\${CLHEP_BASE_DIR}
export PYTHONPATH=\${WORK_DIR}
export OPTICKS_HOME=\${WORK_DIR}/opticks
export OPTICKS_PREFIX=\${WORK_DIR}/local/opticks                            
export OPTICKS_INSTALL_PREFIX=\$LOCAL_BASE/opticks
export OPTICKS_OPTIX_PREFIX=/home/wenzel/NVIDIA-OptiX-SDK-7.5.0-linux64-x86_64
export OPTICKS_CUDA_PREFIX=\${CUDA_INSTALL_DIR}
export OPTICKS_EMBEDDED_COMMANDLINE_EXTRA="--rngmax 100 --rtx 1 --skipaheadstep 10000"
export OPTICKS_MAX_PHOTON=10000000

_path_prepend "\${LOCAL_BASE}/bin"
# make sure to add the compiler options
new=" -fPIC" 
case ":\${CXXFLAGS:=\$new}:" in
*:"\$new":*)  ;;
*) CXXFLAGS="$CXXFLAGS:\$new"  ;;
esac
new=" -fPIC" 
case ":\${CFLAGS:=\$new}:" in
*:"\$new":*)  ;;
*) CFLAGS="\$CFLAGS:\$new"  ;;
esac
# speed up the make process
new=" -j\$((\$(grep -c ^processor /proc/cpuinfo) - 1))"
case ":\${MAKEFLAGS:=\$new}:" in
*:"\$new":*)  ;;
*) MAKEFLAGS="\$MAKEFLAGS:\$new"  ;;
esac
# deal with the LD_LIBRARYPATH
new=\${OptiX_INSTALL_DIR}/lib64/
case ":\${LD_LIBRARY_PATH:=\$new}:" in
*:"\$new":*)  ;;
*) LD_LIBRARY_PATH="\$new:\$LD_LIBRARY_PATH"  ;;
esac
new=\${OPTICKS_HOME}/externals/lib
case ":\${LD_LIBRARY_PATH:=\$new}:" in
*:"\$new":*)  ;;
*) LD_LIBRARY_PATH="\$new:$LD_LIBRARY_PATH"  ;;
esac
new=\${CUDA_INSTALL_DIR}/lib64/
case ":\${LD_LIBRARY_PATH:=\$new}:" in
*:"\$new":*)  ;;
*) LD_LIBRARY_PATH="\$new:\$LD_LIBRARY_PATH"  ;;
esac
new=\${LOCAL_BASE}/opticks/lib/
case ":\${LD_LIBRARY_PATH:=\$new}:" in
*:"\$new":*)  ;;
*) LD_LIBRARY_PATH="\$new:\$LD_LIBRARY_PATH"  ;;
esac
new="."
case ":\${LD_LIBRARY_PATH:=\$new}:" in
*:"\$new":*)  ;;
*) LD_LIBRARY_PATH="\$new:\$LD_LIBRARY_PATH"  ;;
esac
opticks-
new=\${CUDA_INSTALL_DIR}/bin
case ":\${PATH:=\$new}:" in
*:"\$new":*)  ;;
*) PATH="\$new:\$PATH"  ;;
esac
new=\${OPTICKS_HOME}/bin/
case ":\${PATH:=\$new}:" in
*:"\$new":*)  ;;
*) PATH="\$new:\$PATH"  ;;
esac
new=\${OPTICKS_HOME}/ana/
case ":\${PATH:=\$new}:" in
*:"\$new":*)  ;;
*) PATH="\$new:\$PATH"  ;;
esac
new=\${LOCAL_BASE}/opticks/lib/
case ":\${PATH:=\$new}:" in
*:"\$new":*)  ;;
*) PATH="\$new:\$PATH"  ;;
esac
new=\${CUDA_SAMPLES}/bin/x86_64/linux/release/
case ":\${PATH:=\$new}:" in
*:"\$new":*)  ;;
*) PATH="\$new:\$PATH"  ;;
esac
oinfo-(){
echo 'LD_LIBRARY_PATH:';
echo '================';
echo  \${LD_LIBRARY_PATH}| tr : \\\\n;
echo;
echo 'PATH:';
echo '=====';
echo  \${PATH}| tr : \\\\n;
echo;
echo 'CMAKE_PREFIX_PATH:';
echo '==================';
echo  \${CMAKE_PREFIX_PATH}| tr : \\\\n;
}
dinfo-(){    
nvidia-smi;
\$CUDA_SAMPLES/bin/x86_64/linux/release/deviceQuery
}
export OPTICKS_KEY=CaTS.X4PhysicalVolume.World_PV.6a511c07e6d72b5e4d71b74bd548e8fd

EOF
```

 Now setup the environment and build the Opticks externals and Opticks itself.  

```bash
source setup_opticks.sh
opticks-externals-install >& install_ext.log &
tail -f install_ext.log
cd ${WORK_DIR}
opticks-full  >& install_full.log &
tail -f install_full.log
```
```bash
git clone https://github.com/hanswenzel/artg4tkhybrid.git
cp artg4tkhybrid/cmake/OpticksOptionsAsExternal.cmake opticks/cmake/Modules/
```

# build CaTS

CaTSi s a stand alone Geant4 application. Building and running CaTS is a good test that the environment as well as NVIDIA drivers are setup correctly.

```bash 
git clone https://github.com/hanswenzel/CaTS.git
cd CaTS/
source (path to opticks WORK_DIR)/setup_opticks.sh 
cd ../
mkdir CaTS-build
cd CaTS-build

cmake -GNinja -DCMAKE_BUILD_TYPE=Release \
-DWITH_G4CXOPTICKS=ON \
-DCMAKE_PREFIX_PATH="${LOCAL_BASE}/opticks/externals;${LOCAL_BASE}/opticks" \
-DOPTICKS_PREFIX=${LOCAL_BASE}/opticks \
-DCMAKE_MODULE_PATH=${OPTICKS_HOME}/cmake/Modules \
-DCMAKE_INSTALL_PREFIX=../CaTS-install \
../CaTS

ninja install
cd ../CaTS-install/bin
export LD_LIBRARY_PATH=.:$LD_LIBRARY_PATH
./CaTS -g simpleLArTPC.gdml  -m time.mac
```

# build artg4tk vs Opticks  

 ```bash      
 export WORK_DIR=${PWD}
 #
 # Build artg4tk and envs
 #
 rm -rf artg4tktest
 source /cvmfs/larsoft.opensciencegrid.org/products/setup
 setup larsoft v09_74_00 -qe20:prof
 unsetup geant4
 setup geant4 v4_11_1_p01ba -q e20:prof
 setup ninja v1_11_0
 setup cmake v3_27_4
 export MRB_PROJECT=artg4tk
 setup mrb 
 setup  cetmodules v3_24_01
 # do this once
 mkdir artg4tktest
 cd artg4tktest
 mrb newDev  -v v12_00 -q e20:prof
 source ${WORK_DIR}/artg4tktest/localProducts_artg4tk_v12_00_e20_prof/setup
 cd $MRB_SOURCE
 mrb g artg4tk
 mrbsetenv
 cd $MRB_BUILDDIR
 export OptiX_INSTALL_DIR=/home/wenzel/NVIDIA-OptiX-SDK-7.5.0-linux64-x86_64
 export LOCAL_BASE=${WORK_DIR}/local
 export OPTICKS_HOME=${WORK_DIR}/opticks
 export OPTICKS_OPTIX_PREFIX=${OptiX_INSTALL_DIR}
 export OPTICKS_PREFIX=${LOCAL_BASE}/opticks
 export CUDA_INSTALL_DIR=/opt/nvidia/hpc_sdk/Linux_x86_64/23.3/cuda/11.8
 ls 
 #
 # no need
 #
            -DCMAKE_MODULE_PATH=${OPTICKS_HOME}/cmake/Modules

 mrb b -j 10
 cd ${WORK_DIR}
 #
 # build with opticks
 #
 export LOCAL_BASE=${TOP_DIR}/local
 export OPTICKS_HOME=${TOP_DIR}/opticks
 export OptiX_INSTALL_DIR=/home/wenzel/NVIDIA-OptiX-SDK-7.5.0-linux64-x86_64
 export OPTICKS_COMPUTE_CAPABILITY=75
 export CUDA_INSTALL_DIR=/opt/nvidia/hpc_sdk/Linux_x86_64/23.3/cuda/11.8
 export PATH=${CUDA_INSTALL_DIR}/bin:$PATH
 export CUDA_SAMPLES=/data/software/cuda-samples
 export CMAKE_PREFIX_PATH="/cvmfs/larsoft.opensciencegrid.org/products/boost/v1_80_0/Linux64bit+3.10-2.17-e20-prof:${G4INSTALL}:${LOCAL_BASE}/opticks/externals:${OptiX_INSTALL_DIR}:${WORK_DIR}/opticks/cmake/Modules:${WORK_DIR}/local/opticks:${WORK_DIR}/local/opticks:${CLHEP_BASE_DIR}"

export WORK_DIR=${PWD}
export OptiX_INSTALL_DIR=/home/wenzel/NVIDIA-OptiX-SDK-7.5.0-linux64-x86_64
export LOCAL_BASE=${WORK_DIR}/local
export OPTICKS_HOME=${WORK_DIR}/opticks
export OPTICKS_OPTIX_PREFIX=${OptiX_INSTALL_DIR}
export OPTICKS_PREFIX=${LOCAL_BASE}/opticks
export CUDA_INSTALL_DIR=/opt/nvidia/hpc_sdk/Linux_x86_64/23.3/cuda/11.8

 mrb b -j 10 -DWITH_G4CXOPTICKS=ON  \
 -DCMAKE_PREFIX_PATH="${LOCAL_BASE}/opticks/externals;${LOCAL_BASE}/opticks;${OptiX_INSTALL_DIR}" \
 -DOPTICKS_PREFIX=${LOCAL_BASE}/opticks \
 -DCMAKE_MODULE_PATH=${OPTICKS_HOME}/cmake/Modules
```
