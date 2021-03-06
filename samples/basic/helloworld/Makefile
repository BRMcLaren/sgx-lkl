
APP_ROOT=app
PROG=${APP_ROOT}/helloworld
PROG_NONPIE=${APP_ROOT}/helloworld-nonpie
PROG_C=helloworld.c

DISK_IMAGE=sgxlkl-helloworld.img
IMAGE_SIZE=100M

SGXLKL_ROOT=../../..

MUSL_CC=${SGXLKL_ROOT}/build/host-musl/bin/musl-gcc

SGXLKL_STARTER=$(SGXLKL_ROOT)/build/sgx-lkl-run-oe

ifeq ($(SGXLKL_VERBOSE),)
SGXLKL_ENV=\
   SGXLKL_VERBOSE=1 SGXLKL_KERNEL_VERBOSE=0 SGXLKL_TRACE_SIGNAL=0\
   SGXLKL_TRACE_HOST_SYSCALL=0 SGXLKL_TRACE_LKL_SYSCALL=0 SGXLKL_TRACE_MMAP=0
else
SGXLKL_ENV=
endif

SGXLKL_DISK_TOOL=${SGXLKL_ROOT}/tools/sgx-lkl-disk
SGXLKL_GDB=${SGXLKL_ROOT}/tools/gdb/sgx-lkl-gdb

.DELETE_ON_ERROR:
.PHONY: all clean

all: $(DISK_IMAGE)

clean:
	rm -f $(DISK_IMAGE) $(PROG) $(PROG_NONPIE)

$(PROG): $(PROG_C)
	${MUSL_CC} -fPIE -pie -o $@ $(PROG_C)

# non-PIE executable are currently unsupported by SGX-LKL-OE
$(PROG_NONPIE): $(PROG_C)
	${MUSL_CC} -fno-pie -no-pie -o $@ $(PROG_C)

$(DISK_IMAGE): $(PROG) $(PROG_NONPIE)
	${SGXLKL_DISK_TOOL} create --size=${IMAGE_SIZE} --copy=./${APP_ROOT}/ ${DISK_IMAGE}

run: run-hw

run-hw: $(DISK_IMAGE)
	${SGXLKL_ENV} ${SGXLKL_STARTER} --hw-debug --enclave-config enclave_config.json $(DISK_IMAGE)

run-hw-gdb: $(DISK_IMAGE)
	${SGXLKL_ENV} ${SGXLKL_GDB} --args ${SGXLKL_STARTER} --hw-debug --enclave-config enclave_config.json $(DISK_IMAGE)

run-sw: $(DISK_IMAGE)
	${SGXLKL_ENV} ${SGXLKL_STARTER} --sw-debug --enclave-config enclave_config.json $(DISK_IMAGE)

run-sw-gdb: $(DISK_IMAGE)
	${SGXLKL_ENV} ${SGXLKL_GDB} --args ${SGXLKL_STARTER} --sw-debug --enclave-config enclave_config.json $(DISK_IMAGE)
