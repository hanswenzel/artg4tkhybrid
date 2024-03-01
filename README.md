      #
      # Directory
      #
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
      setup cmake v3_25_2
      export MRB_PROJECT=artg4tk
      setup mrb 
      # do this once
      mkdir artg4tktest
      cd artg4tktest
      mrb newDev  -v v12_00 -q e20:prof
      source ${TOP_DIR}/artg4tktest/localProducts_artg4tk_v12_00_e20_prof/setup
      cd $MRB_SOURCE
      mrb g artg4tk
      mrbsetenv
      cd $MRB_BUILDDIR
      source /data/syjun/g4gpu/setup_test.sh
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
