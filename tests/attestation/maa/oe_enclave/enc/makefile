# Detect C and C++ compiler options
# if not gcc and g++, default to clang-7
C_COMPILER=$(notdir $(CC))
ifeq ($(C_COMPILER), gcc)
        CXX_COMPILER=$(notdir $(CXX))
        USE_GCC = true
endif

ifeq ($(USE_GCC),)
        CC = clang-7
        CXX = clang++-7
        C_COMPILER=clang
        CXX_COMPILER=clang++
endif

CFLAGS=$(shell pkg-config oeenclave-$(C_COMPILER) --cflags)
CXXFLAGS=$(shell pkg-config oeenclave-$(CXX_COMPILER) --cflags)
LDFLAGS=$(shell pkg-config oeenclave-$(CXX_COMPILER) --libs)
INCDIR=$(shell pkg-config oeenclave-$(C_COMPILER) --variable=includedir)
COMMONS="../../../common"

.PHONY: all build clean run

all:
	$(MAKE) build
	$(MAKE) sign

build:
	@ echo "Compilers used: $(CC), $(CXX)"
	oeedger8r ../oeApp.edl --trusted \
		--search-path $(INCDIR) \
		--search-path $(INCDIR)/openenclave/edl/sgx
	$(CXX) -c $(CXXFLAGS) $(INCLUDES) -std=c++11 server.cpp
	$(CXX) -c $(CFLAGS) $(CINCLUDES) tlssrv.cpp
	$(CXX) -c $(CFLAGS) $(CINCLUDES) $(COMMONS)/gencreds.cpp
	$(CC) -c $(CFLAGS) $(CINCLUDES) -I$(COMMONS) $(COMMONS)/transport_utils.c
	$(CC) -c $(CFLAGS) $(CINCLUDES) -I$(COMMONS) $(COMMONS)/log_utils.c
	$(CC) -c $(CFLAGS) $(CINCLUDES) oeApp_t.c
	$(CXX) -o oeApp_enc server.o tlssrv.o gencreds.o oeApp_t.o log_utils.o transport_utils.o $(LDFLAGS) -lmbedtls -lmbedcrypto -loehostsock -loehostresolver -loecore -loelibc -loesyscall

sign:
	openssl genrsa -out private.pem -3 3072
	oesign sign -e oeApp_enc -c enc.conf -k private.pem

clean:
	rm -f *.o oeApp_enc oeApp_enc.signed oeApp_t.* oeApp_args.h private.pem
