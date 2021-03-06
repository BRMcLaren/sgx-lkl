
SGXLKL_ROOT=../../..

CC_APP=/usr/bin/python3
CC_APP_CMDLINE=${CC_APP} -c 'print("Hello SGX World from Encrypted Confidential Container!")'
CC_APP_CONFIG_TEMPLATE=sgxlkl-encrypted-python-config-template.json
CC_APP_CONFIG=sgxlkl-encrypted-python-config.json

CC_IMAGE_SIZE=128M

CC_IMAGE_VERITY=sgxlkl-alpine-crypt-verity.img
CC_IMAGE_VERITY_KEY_FILE=$(CC_IMAGE_VERITY).key
CC_IMAGE_VERITY_ROOTHASH=$(CC_IMAGE_VERITY).roothash
CC_IMAGE_VERITY_OFFSET=$(CC_IMAGE_VERITY).hashoffset

CC_IMAGE_INTEGRITY=sgxlkl-alpine-crypt-integrity.img
CC_IMAGE_INTEGRITY_KEY_FILE=$(CC_IMAGE_INTEGRITY).key

CC_IMAGE_DATA=data-crypt-integrity.img

SGXLKL_ENV_VERITY=\
SGXLKL_HD_KEY=${CC_IMAGE_VERITY_KEY_FILE}\
SGXLKL_HD_VERITY=$(shell cat ${CC_IMAGE_VERITY_ROOTHASH})\
SGXLKL_HD_VERITY_OFFSET=$(shell cat ${CC_IMAGE_VERITY_OFFSET})\
SGXLKL_HD_RO=1

SGXLKL_ENV_INTEGRITY=\
SGXLKL_HD_KEY=${CC_IMAGE_INTEGRITY_KEY_FILE}\
SGXLKL_HD_RO=1

SGXLKL_ENV_APP_CONFIG=\
SGXLKL_HDS=${CC_IMAGE_DATA}:/data:0

VERBOSE_OPTS=SGXLKL_VERBOSE=1 SGXLKL_KERNEL_VERBOSE=1

ifeq ($(SGXLKL_VERBOSE),)
	SGXLKL_ENV_VERITY+=${VERBOSE_OPTS}
	SGXLKL_ENV_INTEGRITY+=${VERBOSE_OPTS}
	SGXLKL_ENV_APP_CONFIG+=${VERBOSE_OPTS}
endif

SGXLKL_STARTER=$(SGXLKL_ROOT)/build/sgx-lkl-run-oe
SGXLKL_DISK_TOOL=${SGXLKL_ROOT}/tools/sgx-lkl-disk
SGXLKL_GDB=${SGXLKL_ROOT}/tools/gdb/sgx-lkl-gdb

.DELETE_ON_ERROR:
.PHONY: all clean run run-hw-verity run-sw-verity run-hw-integrity run-sw-integrity

all: $(CC_IMAGE_VERITY) $(CC_IMAGE_INTEGRITY) ${CC_APP_CONFIG}

clean:
	rm -f $(CC_IMAGE_VERITY) $(CC_IMAGE_VERITY_KEY_FILE) $(CC_IMAGE_VERITY_OFFSET) \
	$(CC_IMAGE_VERITY_ROOTHASH) $(CC_IMAGE_INTEGRITY) $(CC_IMAGE_INTEGRITY_KEY_FILE) \
	${CC_APP_CONFIG} ${CC_IMAGE_DATA}

$(CC_IMAGE_VERITY):
	${SGXLKL_DISK_TOOL} create --size=${CC_IMAGE_SIZE} --encrypt --key-file --verity --alpine="python3" ${CC_IMAGE_VERITY}

$(CC_IMAGE_INTEGRITY):
	${SGXLKL_DISK_TOOL} create --size=${CC_IMAGE_SIZE} --encrypt --key-file --integrity --alpine="python3" ${CC_IMAGE_INTEGRITY}

${CC_APP_CONFIG}:
	cp ${CC_IMAGE_INTEGRITY} ${CC_IMAGE_DATA}
	cp ${CC_APP_CONFIG_TEMPLATE} ${CC_APP_CONFIG}
	$(eval CC_IMAGE_INTEGRITY_KEY=$(shell xxd -ps -c100 ${CC_IMAGE_INTEGRITY_KEY_FILE}))

	sed -i "/\"key\":/c\\"\"key\"": \"${CC_IMAGE_INTEGRITY_KEY}\"" ${CC_APP_CONFIG}

run: run-hw-verity

run-hw-verity: $(CC_IMAGE_VERITY)
	${SGXLKL_ENV_VERITY} ${SGXLKL_STARTER} --hw-debug $(CC_IMAGE_VERITY) $(CC_APP_CMDLINE)

run-sw-verity: $(CC_IMAGE_VERITY)
	${SGXLKL_ENV_VERITY} ${SGXLKL_STARTER} --sw-debug $(CC_IMAGE_VERITY) $(CC_APP_CMDLINE)

run-hw-integrity: $(CC_IMAGE_INTEGRITY)
	${SGXLKL_ENV_INTEGRITY} ${SGXLKL_STARTER} --hw-debug $(CC_IMAGE_INTEGRITY) $(CC_APP_CMDLINE)

run-sw-integrity: $(CC_IMAGE_INTEGRITY)
	${SGXLKL_ENV_INTEGRITY} ${SGXLKL_STARTER} --sw-debug $(CC_IMAGE_INTEGRITY) $(CC_APP_CMDLINE)

run-hw-app-config: $(CC_IMAGE_INTEGRITY)
	${SGXLKL_ENV_APP_CONFIG} ${SGXLKL_STARTER} --app-config=${CC_APP_CONFIG} --hw-debug $(CC_IMAGE_INTEGRITY) $(CC_APP_CMDLINE)

run-sw-app-config: $(CC_IMAGE_INTEGRITY)
	${SGXLKL_ENV_APP_CONFIG} ${SGXLKL_STARTER} --app-config=${CC_APP_CONFIG} --sw-debug $(CC_IMAGE_INTEGRITY) $(CC_APP_CMDLINE)
