#***************************************************************************
#
#   File Name   [Makefile]
#
#   Synopsis    []
#
#   Description []
#
#   Revision    [0.1; Initial build from NVIDIA common.mk;
#                Xiao-Long Wu, ECE UIUC]
#   Date        [04/04/2010]
#
#***************************************************************************

# =========================================================================
# Source files
# =========================================================================

# Target executable name
TARGET     := xcpplib
LIBRARY    := lib$(TARGET).a

# Cuda source files (compiled with nvcc)
CUFILES    := xcpplib_typesGpu.cu

# C/C++ source files (compiled with g++)
CCFILES    := xcpplib_global.cpp \
              xcpplib_others.cpp \
              xcpplib_process.cpp \
              xcpplib_types.cpp

# =========================================================================
# Environment setting: Directories
# =========================================================================

# CUDA SDK code library path
ifeq ($(origin CUDA_SDK_DIR), undefined)
  CUDA_SDK_DIR  := ../cuda_sdk
else
  CUDA_SDK_DIR  := ../$(CUDA_SDK_DIR)
endif
CUDA_SDK_LIB    := -L$(CUDA_SDK_DIR)/lib/$(OSLOWER) -lcutil
CUDA_SDK_COMMON := $(CUDA_SDK_DIR)

SRCDIR ?= .
BINDIR ?= $(SRCDIR)/bin/$(OSLOWER)
OBJDIR ?= $(SRCDIR)/obj/$(OSLOWER)
LIBDIR ?= $(SRCDIR)/lib/$(OSLOWER)

#XCPPLIB_DIR := xcpplib
#XCPPLIB_LIB ?= -L$(XCPPLIB_DIR)/lib/$(OSLOWER) -lxcpplib

INCLUDES    ?= -I$(CUDA_SDK_DIR)/inc

# =========================================================================
# Environment setting: Compilers
# =========================================================================

# Compiler setup

# GCC/G++ 4.4 doesn't support CUDA
# GCC/G++ 4.3 supports both OpenMP and CUDA
# GCC/G++ 4.1 supports only CUDA

CXX        ?= g++
CC         ?= gcc
LINKER     ?= g++ #-fPIC
AR         ?= ar
RANLIB     ?= ranlib

CUDA_INSTALL_PATH ?= /usr/local/cuda

ifdef cuda-install
	CUDA_INSTALL_PATH := $(cuda-install)
endif

# =========================================================================
# Rules
# =========================================================================

# Default architecture compute capability
# This will be overrided when DOUBLE_PRECISION is "true".
CUDA_ARCH := -arch sm_10

#ifeq ($(OPENMP), "true")
#  CXXFLAGS += -fopenmp -DENABLE_OPENMP
#  OPENMP_LIB := -fopenmp
#endif

#ifeq ($(DOUBLE_PRECISION), "true")
#  CXXFLAGS += -DENABLE_DOUBLE_PRECISION
#  NVCCFLAGS += -DENABLE_DOUBLE_PRECISION
#  # Only sm_13 supports double precision hardware.
#  CUDA_ARCH := -arch sm_13
#endif

# Override the options given above
CUDA_ARCH := -arch sm_10

#NVCCFLAGS += $(CUDA_ARCH)

# =========================================================================
# Environment setting: Others
# =========================================================================

.SUFFIXES : .cu .cu_dbg.o .c_dbg.o .cpp_dbg.o .cu_rel.o .c_rel.o .cpp_rel.o .cubin .ptx

# Add new SM Versions here as devices with new Compute Capability are released
SM_VERSIONS := sm_10 sm_11 sm_12 sm_13

# detect OS
OSUPPER = $(shell uname -s 2>/dev/null | tr [:lower:] [:upper:])
OSLOWER = $(shell uname -s 2>/dev/null | tr [:upper:] [:lower:])
# 'linux' is output for Linux system, 'darwin' for OS X
DARWIN = $(strip $(findstring DARWIN, $(OSUPPER)))

# detect if 32 bit or 64 bit system
HP_64 =	$(shell uname -m | grep 64)

# Compilers
ifeq "$(strip $(HP_64))" ""
   NVCC := $(CUDA_INSTALL_PATH)/bin/nvcc 
else
   NVCC := $(CUDA_INSTALL_PATH)/bin/nvcc 
endif

# Includes
INCLUDES  += -I. -I$(CUDA_INSTALL_PATH)/include

# architecture flag for cubin build
CUBIN_ARCH_FLAG :=

# Warning flags
CXXWARN_FLAGS := \
	-W -Wall \
	-Wimplicit \
	-Wswitch \
	-Wformat \
	-Wchar-subscripts \
	-Wparentheses \
	-Wmultichar \
	-Wtrigraphs \
	-Wpointer-arith \
	-Wcast-align \
	-Wreturn-type \
	-Wno-unused-function \
	$(SPACE)

CWARN_FLAGS := $(CXXWARN_FLAGS) \
	-Wstrict-prototypes \
	-Wmissing-prototypes \
	-Wmissing-declarations \
	-Wnested-externs \
	-Wmain

# Compiler-specific flags
#NVCCFLAGS := 
CXXFLAGS  += $(CXXWARN_FLAGS)
CFLAGS    += $(CWARN_FLAGS)

# Common flags
COMMONFLAGS += $(INCLUDES) -DUNIX

# Debug/release configuration
ifeq ($(debug_xcpplib),1)
	COMMONFLAGS += -g
	BINSUBDIR   := debug
	LIBSUFFIX   := D
	NVCCFLAGS   += -DXCPPLIB_DEBUG --define-macro debug
	CXXFLAGS    += -DXCPPLIB_DEBUG
	CFLAGS      += -DXCPPLIB_DEBUG
else 
	COMMONFLAGS += -O3 # -O2
	BINSUBDIR   := release
	LIBSUFFIX   :=
	NVCCFLAGS   += --compiler-options -fno-strict-aliasing
	CXXFLAGS    += -fno-strict-aliasing
	CFLAGS      += -fno-strict-aliasing
endif

# append optional arch/SM version flags (such as -arch sm_11)
#NVCCFLAGS += $(SMVERSIONFLAGS)

# architecture flag for cubin build
CUBIN_ARCH_FLAG :=

# OpenGL is used or not (if it is used, then it is necessary to include GLEW)
ifeq ($(USEGLLIB),1)

	ifneq ($(DARWIN),)
		OPENGLLIB := -L/System/Library/Frameworks/OpenGL.framework/Libraries -lGL -lGLU $(CUDA_SDK_COMMON)/lib/$(OSLOWER)/libGLEW.a
	else
		OPENGLLIB := -lGL -lGLU -lX11 -lXi -lXmu

		ifeq "$(strip $(HP_64))" ""
			OPENGLLIB += -lGLEW -L/usr/X11R6/lib
		else
			OPENGLLIB += -lGLEW_x86_64 -L/usr/X11R6/lib64
		endif
	endif

	CUBIN_ARCH_FLAG := -m64
endif

ifeq ($(USEGLUT),1)
	ifneq ($(DARWIN),)
		OPENGLLIB += -framework GLUT
	else
		OPENGLLIB += -lglut
	endif
endif

ifeq ($(USEPARAMGL),1)
	PARAMGLLIB := -lparamgl$(LIBSUFFIX)
endif

ifeq ($(USERENDERCHECKGL),1)
	RENDERCHECKGLLIB := -lrendercheckgl$(LIBSUFFIX)
endif

ifeq ($(USECUDPP), 1)
	ifeq "$(strip $(HP_64))" ""
		CUDPPLIB := -lcudpp
	else
		CUDPPLIB := -lcudpp64
	endif

	CUDPPLIB := $(CUDPPLIB)$(LIBSUFFIX)

	ifeq ($(emu), 1)
		CUDPPLIB := $(CUDPPLIB)_emu
	endif
endif

# Libs
ifeq "$(strip $(HP_64))" ""
   LIB := -L$(CUDA_INSTALL_PATH)/lib -L$(CUDA_SDK_COMMON)/lib/$(OSLOWER) $(OPENMP_LIB)
else
   LIB := -L$(CUDA_INSTALL_PATH)/lib64 -L$(CUDA_SDK_COMMON)/lib/$(OSLOWER) $(OPENMP_LIB)
endif

# If dynamically linking to CUDA and CUDART, we exclude the libraries from the LIB
ifeq ($(USECUDADYNLIB),1)
     LIB += ${OPENGLLIB} $(PARAMGLLIB) $(RENDERCHECKGLLIB) $(CUDPPLIB) -ldl -rdynamic 
else
# static linking, we will statically link against CUDA and CUDART
  ifeq ($(USEDRVAPI),1)
     LIB += -lcuda   ${OPENGLLIB} $(PARAMGLLIB) $(RENDERCHECKGLLIB) $(CUDPPLIB)
  else
     LIB += -lcudart ${OPENGLLIB} $(PARAMGLLIB) $(RENDERCHECKGLLIB) $(CUDPPLIB)
  endif
endif

ifeq ($(USECUFFT),1)
  ifeq ($(emu),1)
    LIB += -lcufftemu
  else
    LIB += -lcufft
  endif
endif

ifeq ($(USECUBLAS),1)
  ifeq ($(emu),1)
    LIB += -lcublasemu
  else
    LIB += -lcublas
  endif
endif

# Device emulation configuration
ifeq ($(emu), 1)
	NVCCFLAGS   += -deviceemu --define-macro emu
	CUDACCFLAGS += 
	BINSUBDIR   := emu$(BINSUBDIR)
	# consistency, makes developing easier
	CXXFLAGS    += -D__DEVICE_EMULATION__ -Demu
	CFLAGS      += -D__DEVICE_EMULATION__ -Demu
endif

TARGETDIR := $(BINDIR)/$(BINSUBDIR)
TARGET    := $(TARGETDIR)/$(TARGET)
LINKLINE   = $(LINKER) -o $(TARGET) $(OBJS) $(LIB)

# check if verbose 
ifeq ($(verbose), 1)
	VERBOSE :=
else
	VERBOSE := @
endif

# =========================================================================
# Check for input flags and set compiler flags appropriately
# =========================================================================

ifeq ($(fastmath), 1)
	NVCCFLAGS += -use_fast_math
endif

ifeq ($(keep), 1)
	NVCCFLAGS += -keep
	NVCC_KEEP_CLEAN := *.i* *.cubin *.cu.c *.cudafe* *.fatbin.c *.ptx
endif

ifdef maxregisters
	NVCCFLAGS += -maxrregcount $(maxregisters)
endif

# Add cudacc flags
NVCCFLAGS += $(CUDACCFLAGS)

# Add common flags
NVCCFLAGS += $(COMMONFLAGS)
CXXFLAGS  += $(COMMONFLAGS)
CFLAGS    += $(COMMONFLAGS)

ifeq ($(nvcc_warn_verbose),1)
	NVCCFLAGS += $(addprefix --compiler-options ,$(CXXWARN_FLAGS)) 
	NVCCFLAGS += --compiler-options -fno-strict-aliasing
endif

# =========================================================================
# Set up object files
# =========================================================================

OBJDIR := $(OBJDIR)/$(BINSUBDIR)
OBJS += $(patsubst %.cpp,$(OBJDIR)/%.cpp.o,$(notdir $(CCFILES)))
OBJS += $(patsubst %.c,$(OBJDIR)/%.c.o,$(notdir $(CFILES)))
OBJS += $(patsubst %.cu,$(OBJDIR)/%.cu.o,$(notdir $(CUFILES)))

# =========================================================================
# Set up cubin output files
# =========================================================================

CUBINDIR := $(SRCDIR)/data
CUBINS += $(patsubst %.cu,$(CUBINDIR)/%.cubin,$(notdir $(CUBINFILES)))

# =========================================================================
# Set up PTX output files
# =========================================================================

PTXDIR := $(SRCDIR)/data
PTXBINS += $(patsubst %.cu,$(PTXDIR)/%.ptx,$(notdir $(PTXFILES)))

# =========================================================================
# Make Rules
# =========================================================================

$(OBJDIR)/%.c.o : $(SRCDIR)/%.c $(C_DEPS)
	$(VERBOSE)@echo
	@echo Making $< at $(BINSUBDIR) mode ...
	$(VERBOSE)$(CC) $(CFLAGS) -o $@ -c $<

$(OBJDIR)/%.cpp.o : $(SRCDIR)/%.cpp $(C_DEPS)
	$(VERBOSE)@echo
	@echo Making $< at $(BINSUBDIR) mode ...
	$(VERBOSE)$(CXX) $(CXXFLAGS) -o $@ -c $<

$(OBJDIR)/%.cu.o : $(SRCDIR)/%.cu $(CU_DEPS)
	$(VERBOSE)@echo
	@echo Making $< at $(BINSUBDIR) mode ...
	$(VERBOSE)$(NVCC) $(NVCCFLAGS) $(SMVERSIONFLAGS) -o $@ -c $<

$(CUBINDIR)/%.cubin : $(SRCDIR)/%.cu cubindirectory
	$(VERBOSE)@echo
	@echo Making $< at $(BINSUBDIR) mode ...
	$(VERBOSE)$(NVCC) $(CUBIN_ARCH_FLAG) $(NVCCFLAGS) $(SMVERSIONFLAGS) -o $@ -cubin $<

$(PTXDIR)/%.ptx : $(SRCDIR)/%.cu ptxdirectory
	$(VERBOSE)@echo
	@echo Making $< at $(BINSUBDIR) mode ...
	$(VERBOSE)$(NVCC) $(CUBIN_ARCH_FLAG) $(NVCCFLAGS) $(SMVERSIONFLAGS) -o $@ -ptx $<

# The following definition is a template that gets instantiated for each SM
# version (sm_10, sm_13, etc.) stored in SMVERSIONS.  It does 2 things:
# 1. It adds to OBJS a .cu_sm_XX.o for each .cu file it finds in CUFILES_sm_XX.
# 2. It generates a rule for building .cu_sm_XX.o files from the corresponding 
#    .cu file.
#
# The intended use for this is to allow Makefiles that use common.mk to compile
# files to different Compute Capability targets (aka SM arch version).  To do
# so, in the Makefile, list files for each SM arch separately, like so:
#
# CUFILES_sm_10 := mycudakernel_sm10.cu app.cu
# CUFILES_sm_12 := anothercudakernel_sm12.cu
#
define SMVERSION_template
OBJS += $(patsubst %.cu,$(OBJDIR)/%.cu_$(1).o,$(notdir $(CUFILES_$(1))))
$(OBJDIR)/%.cu_$(1).o : $(SRCDIR)/%.cu $(CU_DEPS)
	$(VERBOSE)$(NVCC) -o $$@ -c $$< $(NVCCFLAGS) -arch $(1)
endef

# This line invokes the above template for each arch version stored in
# SM_VERSIONS.  The call funtion invokes the template, and the eval
# function interprets it as make commands.
$(foreach smver,$(SM_VERSIONS),$(eval $(call SMVERSION_template,$(smver))))

$(TARGET): makedirectories $(OBJS) $(CUBINS) $(PTXBINS) library Makefile
	$(VERBOSE)@echo
	@echo All XCPPLIB files are compiled.
#	$(VERBOSE)@echo
#	@echo Making $(TARGET) ...
#	$(VERBOSE)$(LINKLINE)

library: $(OBJS)
	$(VERBOSE)@ echo
	@ echo Making '"'$(LIBDIR)/$(LIBRARY)'"' for $(OSLOWER) platform at $(BINSUBDIR) mode ...
	$(VERBOSE)$(AR) r $(LIBDIR)/$(LIBRARY) $(OBJS)
	$(VERBOSE)$(RANLIB) $(LIBDIR)/$(LIBRARY)

cubindirectory:
	$(VERBOSE)mkdir -p $(CUBINDIR)

ptxdirectory:
	$(VERBOSE)mkdir -p $(PTXDIR)

makedirectories:
	$(VERBOSE)mkdir -p $(LIBDIR)
	$(VERBOSE)mkdir -p $(OBJDIR)
	$(VERBOSE)mkdir -p $(TARGETDIR)

tidy :
	$(VERBOSE)find . | egrep "#" | xargs rm -f
	$(VERBOSE)find . | egrep "\~" | xargs rm -f

clean : tidy
	$(VERBOSE)rm -f $(TARGET)
	$(VERBOSE)rm -f $(OBJS)
	$(VERBOSE)rm -f $(LIBDIR)/$(LIBRARY)
	$(VERBOSE)rm -f $(CUBINS)
	$(VERBOSE)rm -f $(PTXBINS)
	$(VERBOSE)rm -f $(NVCC_KEEP_CLEAN)

clobber : clean
	$(VERBOSE)rm -rf $(ROOTOBJDIR)

# =========================================================================
#  End of Makefile
# =========================================================================

