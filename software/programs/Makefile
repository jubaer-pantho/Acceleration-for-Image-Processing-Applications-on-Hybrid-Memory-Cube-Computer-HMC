TARGET = process
USER_SOURCES = process.cpp
ENABLE_HMC=y


# this is meant to be included by all makefiles. feel free to override any vars - it should still work fine.
# (this should really be in the same directory as Pico and PicoUtl (and Arm), but that's the root of a
#  drive on most people's computer, and not in an existing sourcesafe project)

# Notes on use:
# PROJDIR must be the path of the project relative to the parent of the Pico/software dir. no trailing slash needed.
#    (eg: picoDll's PROJDIR is picoDll)
#    object files aren't lumped into one dir - instead dir structure is maintained.
#      (so when Asmw.o (from Pico/CELibs/Asmw.c) is made for the debug version of RServer it's
#       put at Pico/RServer/Debug.$(os)/Pico/CELibs/Asmw.o)
# to support having both release and debug configurations, i have CFLAGSR and CFLAGSD, which contain additions
#    to CFLAGS that are specific to one configuration. (same thing for LDFLAGS.)
# the inclusion of this file may trigger the update of deps for all files in SOURCES - even if we're
#    just going to run 'make clean'. weird, but it works and doesn't take long
#

OS:=$(shell uname | tr a-z A-Z)
os:=$(shell uname)
# this PICOBASE fixup isn't much help since this file probably won't be found if PICOBASE isn't right.
ifeq ($(PICOBASE),)
ifeq "$(OS)" "DARWIN"
PICOBASE:=/usr/src/pico/dist
else
PICOBASE:=$(shell echo ~pico/svn)
endif
endif
#translate slashes to Unix style.
export PICOBASE:=$(subst \,/,$(PICOBASE))
ifndef BUILDDIRR
BUILDDIRR=Release.$(os)
endif
ifndef BUILDDIRD
BUILDDIRD=Debug.$(os)
endif

ifeq ($(TARGETD),)
TARGETD=$(TARGET)d
endif
TARGETR=$(TARGET)

PROJDIRR=$(BUILDDIRR)/$(PROJDIR)
PROJDIRD=$(BUILDDIRD)/$(PROJDIR)

INSTDIRS=$(PICOBASE)/bin
PICOERRORS=pico_errors
PICOSOFTWARE=$(PICOBASE)/software

#CFLAGS+=-fmessage-length=150 \
#		-Wreturn-type \
#		-fno-merge-constants \
#		-U__linux__

HOSTNAME:=$(shell hostname -s | sed -e 's/l-//' -e 's/hp-//' -e 's/debian-//')

ifeq "$(THREADS)" "true"
#CFLAGS += -DHAVE_WTHREADS
CFLAGS += -DHAVE_PTHREADS
endif

ifeq "$(WX)" "true"
ifeq "$(OS)" "DARWIN"
#CFLAGS += -DWX_WIDGETS  -DPICO_WIDGETS  $(shell wx-config --cppflags)
#LDFLAGS += $(shell wx-config --libs | sed -e 's/-pthread//g')
CFLAGS += -DWX_WIDGETS  -DPICO_WIDGETS  $(shell /usr/local/bin/wx-config --cppflags --debug=no)
LDFLAGS += $(shell /usr/local/bin/wx-config --libs --debug=no | sed -e 's/-pthread//g')
else
CFLAGS += -DWX_WIDGETS  -DPICO_WIDGETS  $(shell wx-config --cppflags --debug=no)
LDFLAGS += $(shell wx-config --libs --debug=no | sed -e 's/-pthread//g')
endif
endif
CFLAGS += \
		-Wno-multichar \
		-D$(OS) \
		-DPOSIX \
		-I. \
		-I$(PICOSOFTWARE)/include/linux \
		-I$(PICOSOFTWARE)/include

CFLAGS +=-fmessage-length=0
CFLAGS +=-fdiagnostics-show-option
CFLAGS +=-fms-extensions
CFLAGS +=-Wno-write-strings

# if NO_SWSIM is set to 1, don't include software driven simulation in project
SWSIM_SOURCES=
ifneq ($(NO_SWSIM),1)
SWSIM_SOURCES+=$(PICOBASE)/software/source/pico_drv_swsim.cpp
CFLAGS += -pthread
else
CFLAGS += -DNO_SWSIM
endif

# If SOURCES is empty, grab the user's sources from USER_SOURCES, and add the required platform files.
# This spares the user the hassle of keeping track of what files a Pico project requires.
ifeq "$(SOURCES)" ""
	ifeq "$(OS)" "LINUX"
		LIBSOURCES = $(PICOBASE)/software/source/GString.cpp \
		$(PICOBASE)/software/source/pico_drv.cpp \
		$(PICOBASE)/software/source/pico_errors.cpp \
		$(PICOBASE)/software/source/linux/linux.cpp \
		$(PICOBASE)/software/source/pico_drv_linux.cpp \
		$(SWSIM_SOURCES)
		SOURCES:=$(USER_SOURCES) $(LIBSOURCES)
	endif
endif

# if we want to include the HMC source (not all users necessarily have this or want to include it)
ifdef ENABLE_HMC
CFLAGS+=-DENABLE_HMC
# for now we assume that our HMC code (if it exists) sits next to PICOBASE and the root of the HMC code is called HMC/
HMC_DIR:=$(PICOBASE)/../HMC
HMC_SOURCES:=\
		 PicoBusToDebug.cpp \
		 PicoBusToI2C.cpp \
		 hmc_ctrl.cpp \
		 hmc_ctrl_helpers.cpp \
		 hmc_ctrl_int.cpp \
		 platform.cpp \
		 sb800.cpp \
		 ac510.cpp
HMC_SOURCES_FULL_PATH:=$(addprefix $(HMC_DIR)/software/src/, $(HMC_SOURCES))

CFLAGS+=-I$(HMC_DIR)/software/include
LIB_SOURCES+=$(HMC_SOURCES_FULL_PATH)
SOURCES+=$(HMC_SOURCES_FULL_PATH)
endif

ifeq "$(OS)" "DARWIN"
LIBDL=
CFLAGS += $(POSIXCFLAGS) $(DARWINCFLAGS) -DBSD -DOSNAME="Darwin"
LDFLAGS += $(POSIXLDFLAGS) $(DARWINLDFLAGS)
SOURCES += $(POSIXSOURCES) $(DARWINSOURCES)
endif
ifeq "$(OS)" "OPENBSD"
LIBDL=
CFLAGS += $(POSIXCFLAGS) $(OPENBSDCFLAGS) -DBSD -DOSNAME="OpenBSD"
LDFLAGS += $(POSIXLDFLAGS) $(OPENBSDLDFLAGS)
SOURCES += $(POSIXSOURCES) $(OPENBSDSOURCES)
endif
ifeq "$(OS)" "LINUX"
CFLAGS += $(POSIXCFLAGS) $(LINUXCFLAGS) -DOSNAME="Linux"
LDFLAGS += $(POSIXLDFLAGS) $(LINUXLDFLAGS)
SOURCES += $(POSIXSOURCES) $(LINUXSOURCES)
OBJS = $(LIBSOURCES:.cpp=.o)
LIBNAME = libpico.so
LIB = $(PICOBASE)/software/source/$(LIBNAME)
endif
ifeq "$(OS)" "WINDOWS"
CFLAGS += $(WINDOWSCFLAGS) -DOSNAME="Windows"
LDFLAGS += $(WINDOWSLDFLAGS)
SOURCES += $(WINDOWSSOURCES)
endif

# If we have a Go path, try installing a Go package for our API
ifneq "$(GOPATH)" ""
GOPKG = $(GOPATH)/src/gopico/gopico.go
endif

CFLAGSR +=-O3 $(CFLAGS)
# -DNDEBUG is necescary to use the non-debug Wxwidgets libraries
CFLAGSD +=$(CFLAGS) -DNDEBUG -D_DEBUG -g3 -ggdb3 -DPICO_DEBUG -DDEBUGGER_PRESENT=1
LDFLAGSR+=$(LDFLAGS) $(LIBDL)
LDFLAGSD+=$(LDFLAGS) $(LIBDL)

ifeq "$(CPU)" "mb"
PATH +=:/opt/crosstool/microblaze/bin
CROSS_COMPILE=mb-
endif
ifeq "$(CPU)" "ppc"
PATH +=:/opt/crosstool/gcc-3.4.1-glibc-2.3.3/powerpc-405-linux-gnu/bin
CROSS_COMPILE=powerpc-405-linux-gnu-
endif
#ifndef CC
#CC           =${CROSS_COMPILE}gcc
CC           =${CROSS_COMPILE}g++
#endif
ifndef LD
LD           =${CROSS_COMPILE}ld
LD           =${CROSS_COMPILE}g++
LD           =${CROSS_COMPILE}gcc
endif
ifndef AR
AR           =${CROSS_COMPILE}ar
endif
ifndef SIZE
SIZE         =${CROSS_COMPILE}size
endif
ifndef STRIP
STRIP        =${CROSS_COMPILE}strip
endif
ifndef READELF
READELF      =${CROSS_COMPILE}readelf
endif


SOURCESR +=$(SOURCES)
SOURCESD +=$(SOURCES)
DEPSR := $(SOURCESR:.c=.d)
DEPSR := $(DEPSR:.cpp=.d)
DEPSR := $(DEPSR:.S=.d)
DEPSR := $(DEPSR:.s=.d)

DEPSD := $(SOURCESD:.c=.d)
DEPSD := $(DEPSD:.cpp=.d)
DEPSD := $(DEPSD:.S=.d)
DEPSD := $(DEPSD:.s=.d)

OBJSR =$(addprefix $(PROJDIRR)/, $(DEPSR:.d=.o))
OBJSD =$(addprefix $(PROJDIRD)/, $(DEPSD:.d=.o))

# these includes will create/update the dep files (*.d) if needed, and also create the dirs they are in
#        (which are the same as the dirs the object files will be created in)
#include $(addprefix $(PROJDIRR)/, $(DEPS))
#include $(addprefix $(PROJDIRD)/, $(DEPS))


LD_RUN_PATH=/usr/local/lib
#LIB_PANTHO1 = ../libim/  #-lpthread
#LIB_PANTHO2 = ../jpeg/
LIB_OTHERS = ../libim/libim.a ../jpeg/libjpeg.a -lpthread

all: Release $(GOTARGET) #Debug Errors
	

Release: $(TARGETR)

Debug: $(TARGETD)

#Errors: ../source/builderrors/$(PICOERRORS).cpp

Errors:

rund:
	cgdb $(INSTDIRS)/$(TARGETD) 

# no longer generating cpp from xml. kept for historical trivia
# ../source/$(PICOERRORS).cpp: ../source/$(PICOERRORS).xml

$(TARGETR): $(SOURCESR)
	$(CC) $(CFLAGSR) -o $(TARGETR) $(SOURCESR) $(LDFLAGSR) $(LIB_OTHERS) 
	@$(STRIP) $(TARGETR)
#	for dir in $(INSTDIRS); do echo "Copying $(TARGETR) to $$dir" && cp $(TARGETR) $$dir; done;

$(TARGETD): $(SOURCESD)
	$(CC) $(CFLAGSD) -o $(TARGETD) $(SOURCESD) $(LDFLAGSD) $(LIB_OTHERS) 
#	for dir in $(INSTDIRS); do echo "Copying $(TARGETD) to $$dir" && cp $(TARGETD) $$dir; done;

$(TARGET)-XRC.cpp: $(TARGET).xrc
	wxrc -c $(TARGET).xrc -o $(TARGET)-XRC.cpp

# If the user wants to build a Go version of their program GOPATH must be
# defined.
ifneq "$(GOPATH)" ""
$(GOTARGET): $(GOPKG) $(USER_GO_SOURCES)
	CGO_CFLAGS="-I$(PICOBASE)/software/include" \
	CGO_LDFLAGS="-L$(PICOBASE)/software/source -lpico" \
	go build -o $@ $(USER_GO_SOURCES)
endif

# This just links our Go bindings into the user's Go path
$(GOPKG): $(PICOBASE)/software/source/gopico/gopico.go
	@echo Installing Go package to $(GOPATH)/src
	@cp -r $(PICOBASE)/software/source/gopico $(GOPATH)/src/

%.o: %.cpp
	@echo Building $@ for $(LIB)
	@$(CC) -c -o $@ $^ -fPIC $(CFLAGSR) $(LDFLAGSR) $(LIB_OTHERS) 

.PHONY: libpico
libpico: $(LIB)

$(LIB): $(OBJS)
	@echo Building library: $(LIB)
	@$(CC) -shared -o $@ $(OBJS) -Wl,-soname,$(LIBNAME) $(LIB_OTHERS) 
	@-$(STRIP) --strip-unneeded $@
	@-rm -f $(OBJS)

../source/builderrors/$(PICOERRORS).cpp: ../source/$(PICOERRORS).xml
	../../bin/builderrors ../source/$(PICOERRORS).xml ../builderrors/$(PICOERRORS).h ../builderrors/$(PICOERRORS).cpp

clean:
	@-rm -f *.o *.core $(TARGETR) $(TARGETD) ../source/*.o ../source/linux/*.o
	@-rm -rf $(GOTARGET)
