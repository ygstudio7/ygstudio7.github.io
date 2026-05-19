# Chipyard 설치 흐름

참고: https://chipyard.readthedocs.io/en/latest/Chipyard-Basics/Initial-Repo-Setup.html#prerequisites

## clone

```
git clone https://github.com/ucb-bar/chipyard.git
cd chipyard
```

## submodule (1)

```
./build-setup.sh
```

이 단계가 꽤 오래 걸립니다.



## [Q] build-setup.sh에서 conda가 없다고 나오면?

참고: https://github.com/conda-forge/miniforge/#download



```bash
curl -L -O "https://github.com/conda-forge/miniforge/releases/latest/download/Miniforge3-$(uname)-$(uname -m).sh"
bash Miniforge3-$(uname)-$(uname -m).sh
```

(1)

ubuntu 재시작후 위 (1) build-setup 재실행

```bash
./build-setup.sh
```





(1) 다음 오류가 나서 다시 설치

``` bash
FileNotFoundError: [Errno 2] No such file or directory: 'guestmount'

ERROR: Failed to build workload br-base.json
Log available at: /home/ygkim/dev/chipyard_260518/software/firemarshal/logs/br-base-build-2026-05-18--23-26-19-FM43Y4UEVBHVNL13.log
ERROR: FAILURE: 1 builds failed
build-setup.sh: Build script failed with exit code 1 at step 9: Pre-compiling FireMarshal buildroot sources
```



```bash
sudo apt update
sudo apt install -y libguestfs-tools qemu-utils fuse

which guestmount
guestmount --version

```



WSL/modern Ubuntu에서는 보통 `fuse3`만 사용하면 됩니다.

즉, `fuse`를 빼고 설치하세요.

```
sudo apt update

sudo apt install -y \
    libguestfs-tools \
    qemu-utils
```

이미 `fuse` 설치 시도가 꼬였으면 먼저 정리:

```
sudo apt remove -y fuse
sudo apt autoremove -y
```

그 다음 확인:

```
which guestmount
```



 위 실행되었으면, conda설치 이어가기 (-u)

```bash
bash Miniforge3-$(uname)-$(uname -m).sh -u
```



마지막으로 기본 환경 활성화

```bash
conda activate base
```



```bash
git checkout main
```



# 1. 기존 fuse/libguestfs 정리

```
sudo apt remove -y fuse fuse3 libfuse2 libfuse3-3 guestmount libguestfs-tools
sudo apt autoremove -y
```

------

# 2. universe repository 활성화

```
sudo add-apt-repository universe
sudo apt update
```

------

# 3. 필요한 패키지 재설치

WSL Ubuntu 22.04/24.04 기준:

```
sudo apt install -y \
    libguestfs-tools \
    guestmount \
    qemu-utils \
    qemu-system-x86 \
    fuse3 \
    libfuse2
```

중요:

- `fuse` 말고 `fuse3`
- `libfuse2` 같이 필요

# 4. WSL에서 FUSE 활성화 확인

```
ls -l /dev/fuse
```

정상 예:

```
crw-rw-rw- 1 root root ...
```



## 재설치

```bash
./build-setup.sh -s 1
```



# 2. RTL 생성 테스트

가장 먼저 정상 동작 확인:

```
cd sims/verilator
make CONFIG=RocketConfig
```



## 2.1.4. Simulating The Default Example[](https://chipyard.readthedocs.io/en/latest/Simulation/Software-RTL-Simulation.html#simulating-the-default-example)

To compile the example design, run `make` in the selected verilator or VCS directory. This will elaborate the `RocketConfig` in the example project.

Note

The elaboration of `RocketConfig` requires about 6.5 GB of main memory. Otherwise the process will fail with `make: *** [firrtl_temp] Error 137` which is most likely related to limited resources. Other configurations might require even more main memory.

An executable called `simulator-chipyard.harness-RocketConfig` will be produced. This executable is a simulator that has been compiled based on the design that was built. You can then use this executable to run any compatible RV64 code. For instance, to run one of the riscv-tools assembly tests.

```
./simulator-chipyard.harness-RocketConfig $RISCV/riscv64-unknown-elf/share/riscv-tests/isa/rv64ui-p-simple
```

Note

In a VCS simulator, the simulator name will be `simv-chipyard.harness-RocketConfig` instead of `simulator-chipyard.harness-RocketConfig`.

The makefiles have a `run-binary` rule that simplifies running the simulation executable. It adds many of the common command line options for you and redirects the output to a file.

```
make run-binary BINARY=$RISCV/riscv64-unknown-elf/share/riscv-tests/isa/rv64ui-p-simple
```

Alternatively, we can run a pre-packaged suite of RISC-V assembly or benchmark tests, by adding the make target `run-asm-tests` or `run-bmark-tests`. For example:

```
make run-asm-tests
make run-bmark-tests
```

Note

Before running the pre-packaged suites, you must run the plain `make` command, since the elaboration command generates a `Makefile` fragment that contains the target for the pre-packaged test suites. Otherwise, you will likely encounter a `Makefile` target error.
