
# Notes during slide creation

## Select celeritas version and build

```bash
spack env activate celeritas-a
cd /nobackup/projects/bdsheXX/$USER/aarch64/exatepp/celeritas
git fetch 
git checkout v0.4.2
mkdir -p build && cd build
cd build
```

```bash
cmake .. --preset full -DCELERITAS_USE_CUDA=ON -DCMAKE_CUDA_ARCHITECTURES=90 -DCELERITAS_USE_MPI=OFF -DCELERITAS_USE_SWIG=OFF -DCELERITAS_BUILD_DOCS=OFF -DCMAKE_BUILD_TYPE=Release -DCMAKE_CUDA_FLAGS="-lineinfo -Xcompiler -Wno-psabi" 

cmake --build . -j `nproc`
time ./bin/celer-sim ../../inputs/cms2018+field+msc.json |& tee log.txt
```

```
real    1m16.701s
user    1m7.003s
sys     0m0.799s
```

```bash
export CELER_ENABLE_PROFILING=1
nsys profile -c nvtx -p celer-sim@celeritas -o gh200-celer-sim-0.4.2-cms2018+field+msc.nsys-rep ./bin/celer-sim ../../inputs/cms2018+field+msc.json 

ncu --set=full --nvtx --nvtx-include "celeritas@celer-sim/step/*" --launch-skip 1000 --launch-count 50 -k regex:"launch_action_impl|_kernel_agent" -o gh200-celer-sim-0.4.2-cms2018+field+msc-1000-50.nsys-rep ./bin/celer-sim ../../inputs/cms2018+field+msc.json 

ncu --set=full --nvtx --nvtx-include "celeritas@celer-sim/step/*" --launch-skip-before-match 31000 --launch-count 100 -k regex:"launch_action_impl|_kernel_agent" -o gh200-celer-sim-0.4.2-cms2018+field+msc-31000-100.nsys-rep ./bin/celer-sim ../../inputs/cms2018+field+msc.json 

ncu --set=full --kernel-name launch_action_impl --launch-skip 11102 --launch-count 1 -o gh200-celer-sim-0.4.2-cms2018+field+msc-slowest.nsys-rep ./bin/celer-sim ../../inputs/cms2018+field+msc.json 

```

### With sync

```
 ncu --set=full --nvtx --nvtx-include "celeritas@celer-sim/step/*" --launch-skip-before-match 31000 --launch-count 100 -k regex:"launch_action_impl|_kernel_agent" -o gh200-celer-sim-0.4.2-cms2018+field+msc-31000-100.ncu-rep ./bin/celer-sim ../../inputs/cms2018+field+msc.json
```

## Building a fresh container 

1. make a bunch of changes to the celeritas repo
2. build the docker container via `./build.sh jammy-cuda11` (this takes ~ 2 hours)
3. Create tarball of the docker image `docker save celeritas/dev-jammy-cuda11:2024-03-25 -o temp.tar` (~4 mins)
4. Create apptainer image `apptainer build celeritas-dev-jammy-cuda11-2024-03-25.sif docker-archive:temp.tar` (? mins)
5. Copy container to appropriate location(s)


## mav

```bash
cd ~/code/exatepp/
apptainer run --cleanenv --nv --bind ./:/celeritas-project ./celeritas/scripts/docker/celeritas-dev-jammy-cuda11-2024-03-25.sif 
source /etc/profile.d/celeritas_spack_env.sh
cd celeritas
mkdir -p build-apptainer-run-2024-03 && cd build-apptainer-run-2024-03
cmake .. -DCELERITAS_BUILD_DEMOS=ON -DCELERITAS_BUILD_DOCS=OFF -DCELERITAS_BUILD_TESTS=ON -DCELERITAS_USE_Geant4=ON -DCELERITAS_USE_HepMC3=ON -DCELERITAS_USE_JSON=ON  -DCELERITAS_USE_MPI=OFF -DCELERITAS_USE_ROOT=ON -DCELERITAS_USE_SWIG=OFF -DCELERITAS_USE_VecGeom=ON -DCELERITAS_USE_CUDA=ON -DCMAKE_CUDA_ARCHITECTURES=70 -DCMAKE_BUILD_TYPE=Release -DCMAKE_CUDA_FLAGS="-lineinfo"
cmake --build . -j `nproc`
time ./bin/celer-sim ../../inputs/cms2018+field+msc.json
```

## stanage via apptainer-run

```bash
cd ~/code/exatepp/
apptainer run --cleanenv --nv --bind ./:/celeritas-project /mnt/parscratch/users/$USER/celeritas-dev-jammy-cuda11-2024-03-25.sif
source /etc/profile.d/celeritas_spack_env.sh
cd celeritas
mkdir -p build-apptainer-run-2024-03 && cd build-apptainer-run-2024-03
cmake .. -DCELERITAS_BUILD_DEMOS=ON -DCELERITAS_BUILD_DOCS=OFF -DCELERITAS_BUILD_TESTS=ON -DCELERITAS_USE_Geant4=ON -DCELERITAS_USE_HepMC3=ON -DCELERITAS_USE_JSON=ON  -DCELERITAS_USE_MPI=OFF -DCELERITAS_USE_ROOT=ON -DCELERITAS_USE_SWIG=OFF -DCELERITAS_USE_VecGeom=ON -DCELERITAS_USE_CUDA=ON -DCMAKE_CUDA_ARCHITECTURES=70 -DCMAKE_BUILD_TYPE=Release -DCMAKE_CUDA_FLAGS="-lineinfo" 
cmake --build . -j `nproc`
time ./bin/celer-sim ../../cms2018/apptainer-run-cms2018+msv+field-input.json
```
## gh tests 

```console
99% tests passed, 2 tests failed out of 203

Label Time Summary:
app           = 108.80 sec*proc (11 tests)
gpu           = 101.33 sec*proc (43 tests)
nomemcheck    = 107.88 sec*proc (9 tests)
unit          =  33.99 sec*proc (191 tests)

Total Test time (real) = 140.78 sec

The following tests FAILED:
        158 - celeritas/mat/Material (Failed)
        160 - celeritas/phys/Particle (SEGFAULT)
Errors while running CTest
Output from these tests are in: /nobackup/projects/bdsheXX/$USER/aarch64/exatepp/celeritas/build/Testing/Temporary/LastTest.log
Use "--rerun-failed --output-on-failure" to re-run the failed cases verbosely.
```

```
ctest --rerun-failed --output-on-failure
Test project /nobackup/projects/bdsheXX/$USER/aarch64/exatepp/celeritas/build
    Start 158: celeritas/mat/Material
1/2 Test #158: celeritas/mat/Material ...........***Failed  Error regular expression found in output. Regex=[tests FAILED]  0.68 sec
Celeritas version 0.4.2
[==========] Running 9 tests from 4 test suites.
[----------] Global test environment set-up.
[----------] 2 tests from MaterialUtils
[ RUN      ] MaterialUtils.coulomb_correction
[       OK ] MaterialUtils.coulomb_correction (0 ms)
[ RUN      ] MaterialUtils.radiation_length
[       OK ] MaterialUtils.radiation_length (0 ms)
[----------] 2 tests from MaterialUtils (0 ms total)

[----------] 5 tests from MaterialTest
[ RUN      ] MaterialTest.params
[       OK ] MaterialTest.params (1 ms)
[ RUN      ] MaterialTest.material_view
[       OK ] MaterialTest.material_view (1 ms)
[ RUN      ] MaterialTest.element_view
[       OK ] MaterialTest.element_view (1 ms)
[ RUN      ] MaterialTest.isotope_view
[       OK ] MaterialTest.isotope_view (1 ms)
[ RUN      ] MaterialTest.output
/nobackup/projects/bdsheXX/$USER/aarch64/exatepp/celeritas/test/celeritas/mat/Material.test.cc:297: Failure
Expected:
  R"json({"_units":{"atomic_mass":"amu","mean_excitation_energy":"MeV","nuclear_mass":"MeV/c^2"},"elements":{"atomic_mass":[1.008,26.9815385,22.98976928,126.90447],"atomic_number":[1,13,11,53],"coulomb_correction":[6.400821803338426e-05,0.010734632775699565,0.00770256745342534,0.15954439947436763],"isotope_fractions":[[0.9,0.1],[0.7,0.3],[1.0],[0.05,0.15,0.8]],"isotope_ids":[[0,1],[2,3],[4],[5,6,7]],"label":["H","Al","Na","I"],"mass_radiation_coeff":[0.0158611264432063,0.04164723292591279,0.03605392839455309,0.11791841505608874]},"isotopes":{"atomic_mass_number":[1,2,27,28,23,125,126,127],"atomic_number":[1,1,13,13,11,53,53,53],"label":["1H","2H","27Al","28Al","23Na","125I","126I","127I"],"nuclear_mass":[938.272,1875.61,25126.5,26058.3,21409.2,116321.0,117253.0,118184.0]},"materials":{"density":[3.6700020622594716,0.0,0.00017976000000000003,0.00017943386624303615],"electron_density":[9.4365282069664e+23,0.0,1.073948435904467e+20,1.072e+20],"element_frac":[[0.5,0.5],[],[1.0],[1.0]],"element_id":[[2,3],[],[0],[0]],"label":["NaI","hard vacuum","H2@1","H2@2"],"matter_state":["solid","unspecified","gas","gas"],"mean_excitation_energy":[0.00040000760709482647,0.0,1.9199999999999986e-05,1.9199999999999986e-05],"number_density":[2.948915064677e+22,0.0,1.073948435904467e+20,1.072e+20],"radiation_length":[3.5393292693170424,null,350729.99844063615,351367.4750467326],"temperature":[293.0,0.0,100.0,110.0],"zeff":[32.0,0.0,1.0,1.0]}})json"
Actual:
  R"json({"_units":{"atomic_mass":"amu","mean_excitation_energy":"MeV","nuclear_mass":"MeV/c^2"},"elements":{"atomic_mass":[1.008,26.9815385,22.98976928,126.90447],"atomic_number":[1,13,11,53],"coulomb_correction":[6.400821803338426e-05,0.010734632775699565,0.00770256745342534,0.15954439947436763],"isotope_fractions":[[0.9,0.1],[0.7,0.3],[1.0],[0.05,0.15,0.8]],"isotope_ids":[[0,1],[2,3],[4],[5,6,7]],"label":["H","Al","Na","I"],"mass_radiation_coeff":[0.0158611264432063,0.04164723292591279,0.0360539283945531,0.11791841505608874]},"isotopes":{"atomic_mass_number":[1,2,27,28,23,125,126,127],"atomic_number":[1,1,13,13,11,53,53,53],"label":["1H","2H","27Al","28Al","23Na","125I","126I","127I"],"nuclear_mass":[938.272,1875.61,25126.5,26058.3,21409.2,116321.0,117253.0,118184.0]},"materials":{"density":[3.6700020622594716,0.0,0.00017976000000000003,0.00017943386624303615],"electron_density":[9.4365282069664e+23,0.0,1.073948435904467e+20,1.072e+20],"element_frac":[[0.5,0.5],[],[1.0],[1.0]],"element_id":[[2,3],[],[0],[0]],"label":["NaI","hard vacuum","H2@1","H2@2"],"matter_state":["solid","unspecified","gas","gas"],"mean_excitation_energy":[0.00040000760709482647,0.0,1.9199999999999986e-05,1.9199999999999986e-05],"number_density":[2.948915064677e+22,0.0,1.073948435904467e+20,1.072e+20],"radiation_length":[3.5393292693170424,null,350729.99844063615,351367.4750467326],"temperature":[293.0,0.0,100.0,110.0],"zeff":[32.0,0.0,1.0,1.0]}})json"

[  FAILED  ] MaterialTest.output (1 ms)
[----------] 5 tests from MaterialTest (6 ms total)

[----------] 1 test from MaterialParamsImportTest
[ RUN      ] MaterialParamsImportTest.import_materials
[       OK ] MaterialParamsImportTest.import_materials (213 ms)
[----------] 1 test from MaterialParamsImportTest (213 ms total)

[----------] 1 test from MaterialDeviceTest
[ RUN      ] MaterialDeviceTest.all
[       OK ] MaterialDeviceTest.all (36 ms)
[----------] 1 test from MaterialDeviceTest (36 ms total)

[----------] Global test environment tear-down
[==========] 9 tests from 4 test suites ran. (256 ms total)
[  PASSED  ] 8 tests.
[  FAILED  ] 1 test, listed below:
[  FAILED  ] MaterialTest.output

 1 FAILED TEST
/nobackup/projects/bdsheXX/$USER/aarch64/exatepp/celeritas/build/test/celeritas_mat_Material: tests FAILED

    Start 160: celeritas/phys/Particle
2/2 Test #160: celeritas/phys/Particle ..........   Passed    0.61 sec

50% tests passed, 1 tests failed out of 2

Label Time Summary:
gpu     =   1.29 sec*proc (2 tests)
unit    =   1.29 sec*proc (2 tests)

Total Test time (real) =   1.33 sec

The following tests FAILED:
        158 - celeritas/mat/Material (Failed)
Errors while running CTest
```


## Gpuutiliz

https://github.com/willfurnass/gpuutiliz
https://github.com/ptheywood/gpuutiliz-plotting


must patch nvml go binding pacakge to -2 for newer go's to avoid a sigsegv. for go 1.22.1 atleast, e.g.

+ https://github.com/GoogleCloudPlatform/container-engine-accelerators/pull/352/files
+ https://github.com/NVIDIA/go-nvml/issues/36

```
${HOME}/aarch64/gpuutiliz/gpuutiliz --help
```

```
${HOME}/aarch64/gpuutiliz/gpuutiliz -frequency 1 &
gupid=$!
./bin/celer-sim ../../inputs/cms2018+field+msc.json
echo ${gupid}
kill ${gupid}
```