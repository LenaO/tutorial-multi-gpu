# Copyright (c) 2017-2024, NVIDIA CORPORATION. All rights reserved.
THIS_TASK := 06H-sol
OUTPUT_NAME := jacobi.$(THIS_TASK)__$(shell date '+%Y%m%d-%H%M')
NP ?= 1
NVCC=nvcc
MPICXX=mpicxx
JSC_SUBMIT_CMD ?= srun --gres=gpu:4 --ntasks-per-node 4
CUDA_HOME ?= /usr/local/cuda
_JSCCOURSE_GPU_ARCH?=80
GENCODE_SM30	:= -gencode arch=compute_30,code=sm_30
GENCODE_SM35	:= -gencode arch=compute_35,code=sm_35
GENCODE_SM37	:= -gencode arch=compute_37,code=sm_37
GENCODE_SM50	:= -gencode arch=compute_50,code=sm_50
GENCODE_SM52	:= -gencode arch=compute_52,code=sm_52
GENCODE_SM60    := -gencode arch=compute_60,code=sm_60
GENCODE_SM70    := -gencode arch=compute_70,code=sm_70
GENCODE_SM80    := -gencode arch=compute_80,code=sm_80 -gencode arch=compute_80,code=compute_80
GENCODE_SM90    := -gencode arch=compute_90,code=sm_90 -gencode arch=compute_90,code=compute_90
GENCODE_FLAGS	:= $(GENCODE_SM$(_JSCCOURSE_GPU_ARCH))
ifdef DISABLE_CUB
        NVCC_FLAGS = -Xptxas --optimize-float-atomics
else
        NVCC_FLAGS = -DHAVE_CUB
endif
NVCC_FLAGS += -lineinfo $(GENCODE_FLAGS) -std=c++14
MPICXX_FLAGS = -DUSE_NVTX -I$(CUDA_HOME)/include -std=c++14
LD_FLAGS = -L$(CUDA_HOME)/lib64 -lcudart -lnvToolsExt
jacobi: Makefile jacobi.cpp jacobi_kernels.o
	$(MPICXX) $(MPICXX_FLAGS) jacobi.cpp jacobi_kernels.o $(LD_FLAGS) -o jacobi

jacobi_kernels.o: Makefile jacobi_kernels.cu
	$(NVCC) $(NVCC_FLAGS) jacobi_kernels.cu -c

.PHONY.: clean
clean:
	rm -f jacobi jacobi_kernels.o *.nsys-rep jacobi.*.compute-sanitizer.log

sanitize: jacobi
	$(JSC_SUBMIT_CMD) -n $(NP) compute-sanitizer --log-file $(OUTPUT_NAME).%q{SLURM_PROCID}.compute-sanitizer.log ./jacobi -niter 10

run: jacobi
	$(JSC_SUBMIT_CMD) -n $(NP) ./jacobi

profile: jacobi
	$(JSC_SUBMIT_CMD) -n $(NP) nsys profile --trace=mpi,cuda,nvtx -o $(OUTPUT_NAME).%q{SLURM_PROCID} ./jacobi -niter 10
