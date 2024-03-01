# build Opticks
We want to build opticks with exactly the same tools that we want to build artg4tk later on.    

      export TOP_DIR=${PWD}
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
      mkdir -p ${WORK_DIR}/local/opticks/externals/
      cd ${WORK_DIR}/local/opticks/externals/
      ln -s ${OptiX_INSTALL_DIR} OptiX
      cd ${WORK_DIR}
      opticks-externals-install >& install_ext.log &
      tail -f install_ext.log
      cd ${WORK_DIR}
      opticks-full  >& install_full.log &
      tail -f install_full.log

# build CaTS

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
      
# build artg4tk vs Opticks     
      
      export TOP_DIR=${PWD}
      #
      # Build artg4tk and envs
      #
      cd ${TOP_DIR}
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
      source ${TOP_DIR}/artg4tktest/localProducts_artg4tk_v12_00_e20_prof/setup
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
      mrb b -j 10 -DWITH_G4CXOPTICKS=ON  \
            -DCMAKE_PREFIX_PATH="${LOCAL_BASE}/opticks/externals;${LOCAL_BASE}/opticks;${OptiX_INSTALL_DIR}" \
            -DOPTICKS_PREFIX=${LOCAL_BASE}/opticks 
      #
      # no need
      #
            -DCMAKE_MODULE_PATH=${OPTICKS_HOME}/cmake/Modules

      mrb b -j 10
      cd ${TOP_DIR}
      #
      # build with opticks
      #
      export LOCAL_BASE=/data/syjun/g4gpu/src/local
      export OPTICKS_HOME=/data/syjun/g4gpu/src/opticks
      export OptiX_INSTALL_DIR=/home/wenzel/NVIDIA-OptiX-SDK-7.5.0-linux64-x86_64
      export OPTICKS_COMPUTE_CAPABILITY=86
      export CUDA_INSTALL_DIR=/opt/nvidia/hpc_sdk/Linux_x86_64/23.3/cuda/11.8
      export PATH=${CUDA_INSTALL_DIR}/bin:$PATH
      export CUDA_SAMPLES=/data/software/cuda-samples

      export CMAKE_PREFIX_PATH="/cvmfs/larsoft.opensciencegrid.org/products/boost/v1_80_0/Linux64bit+3.10-2.17-e20-prof:${G4INSTALL}:${LOCAL_BASE}/opticks/externals:${OptiX_INSTALL_DIR}:${WORK_DIR}/opticks/cmake/Modules:${WORK_DIR}/local/opticks:${WORK_DIR}/local/opticks:${CLHEP_BASE_DIR}"

      ${OptiX_INSTALL_DIR}

      source /data/syjun/g4gpu/setup_test.sh
      mrb b -j 10 -DWITH_G4CXOPTICKS=ON  \
            -DCMAKE_PREFIX_PATH="${LOCAL_BASE}/opticks/externals;${LOCAL_BASE}/opticks;${OptiX_INSTALL_DIR}" \
            -DOPTICKS_PREFIX=${LOCAL_BASE}/opticks \
            -DCMAKE_MODULE_PATH=${OPTICKS_HOME}/cmake/Modules
