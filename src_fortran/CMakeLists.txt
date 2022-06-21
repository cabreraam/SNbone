################################################################################
#
# CMakeLists.txt
#
# Author: Anthony Cabrera
# Contact: cabreraam AT ieee DOT org
# Description: CMakeLists.txt replacement for SNaCFE.x/SNBone
#
################################################################################


################################################################################
#
# Options and Variables
#
################################################################################

option(THREAD "Enable OpenMP for multi-threaded version" OFF)
option(PAPI "Enable PAPI for performance monitoring" OFF)
option(IBMHPM "Enable the IBM hpm library" OFF)
option(USEMETIS "Enable the metis library" OFF)
option(USEGREEDY "Enable a greedy choice(?)" OFF)

set(OPT "3" CACHE STRING "Optimization Level")
#TODO: how do they use this flag debug?
set(DEBUG "" CACHE STRING "Don't yet know what they mean by this")
# Not using the FF flag, since we specify the compiler through the CMake 
# Variable CMAKE_Fortran_COMPILER
# They set the linker through a variable LD, but I don't think we need to do 
# that here.

# TODO:
# Figure out how to handle PAPI flags
# There are PAPI inc and lib flags, as well as pound defines
# Same 
#set(PAPIFLAGS "" CACHE STRING)

# TODO:
# IBMFLAGS
# IBMFLAGS gets set if they are using an IBM compiler. In this case, it's bgq,
# or mpixlf90_r.
# The only thing that IBMFLAGS gets set to is -WF,-DWITHBGQHPM
# The -WF flag is a way to pass things to the IBM preprocessor

# TODO:
# IBMLIB
# IBMLIB also only gets set if they are using an IBM compiler. The directories
# for the location of libmpihpm and libbgpm are hard-coded, so I'm assuming the
# user would need to specify those if they wanted to use them

# TODO:
# EXTFLAGS is for extra values to be pound defined.

set(TARGET_NAME
	SNaCFE.x
	CACHE
	STRING
	"The name of the driver build target for SNBone"
)

set(TARGET_SRC_FILES
	ApplyA_AVE1_Tet_SUPG.F90
	ApplyA_AVE2_Tet_SUPG.F90
	AssembleNZmatrix.F90
	Basic_Abort.F90
	Basic_CheckError.F90
	Basic_CommonBlock.F90
	BuildAngleCubature.F90
	CommonBlock.F90
	Driver.F90
	FGMRES_Threaded.F90
	GenerateXb.F90
	Import_PMesh.F90
	Makefile
	Method_Krylov.F90
	PrintSummary.F90
	SolveWGS.F90
	SolveWGS_PassThrough_AVE1.F90
	SolveWGS_PassThrough_AVE2.F90
	SolveWGS_PassThrough_AVE3.F90
	SolveWGS_PassThrough_PC.F90
	StencilNZmatrix.F90
	Verify.F90
	Zero.F90
	CACHE
	STRING
	"List of target source files"
)

################################################################################
#
# Setting up build target
#
################################################################################
add_executable(${TARGET_NAME})
target_sources(${TARGET_NAME}
	PRIVATE
		${TARGET_SRC_FILES}
)

################################################################################
#
# Find Packages
#
################################################################################
# Let CMake Handle finding the OpenMP implementation
if(THREAD)
	find_package(OpenMP REQUIRED)
	target_link_libraries(${TARGET} 
		PUBLIC
		OpenMP::OpenMP_Fortran
	)
endif()


################################################################################
#
# Compile Options for Different Compilers
#
################################################################################
# ------------------------------
# INTEL INTEL INTEL INTEL
# ------------------------------
set(SNACFE_COMPILE_DEFNS_INTEL
	$<$<BOOL:${THREAD}>:WITHOMP>
	CACHE
	STRING
	"Intel compiler definitions"
)
set(SNACFE_COMPILE_OPTIONS_INTEL
	-O${OPT}
	${DEBUG}
	-Bstatic
	$<$<BOOL:${THREAD}>:-Bstatic>
	CACHE
	STRING
	"Intel compiler options"
)
set(INTEL_PAPI_DIR
	/software/papi-5.1.1/O-intel-12.1
)
set(SNACFE_LINK_LIBS_INTEL
	$<$<BOOL:${PAPI}>:${INTEL_PAPI_DIR}/lib/libpapi.a>
	$<$<BOOL:${PAPI}>:${INTEL_PAPI_DIR}/lib/libpfm.a>
	CACHE
	STRING
	"Intel compiler libraries to link"
)
set(SNACFE_INCLUDE_DIRS_INTEL
	$<$<BOOL:${PAPI}>:${INTEL_PAPI_DIR}/include>
	CACHE
	STRING
	"Intel compiler include directories"
)

# ------------------------------
# GFORTRAN GFORTRAN GFORTRAN 
# ------------------------------
set(SNACFE_COMPILE_DEFNS_GNU
	$<$<BOOL:${THREAD}>:WITHOMP>
	$<$<BOOL:${USEGREEDY}>:USEGREEDY>
	$<$<BOOL:${USEMETIS}>:USEMETIS>
	CACHE
	STRING
	"GNU compiler definitions"
)
set(SNACFE_COMPILE_OPTIONS_GNU
	-O${OPT}
	CACHE
	STRING
	"GNU compiler options"
)
set(SNACFE_LINK_LIBS_GNU
	$<$<BOOL:${USEMETIS}>:../metis/libmetis.a>
	CACHE
	STRING
	"GNU link options"
)

# ------------------------------
# IBM IBM IBM IBM IBM IBM IBM
# ------------------------------
set(SNACFE_COMPILE_DEFNS_IBM
	$<$<BOOL:${USEMETIS}>:-DUSEMETIS>
	CACHE
	STRING
	"IBM compile definitions"
)

set(SNACFE_COMPILE_OPTIONS_IBM
	-O${OPT}
	${DEBUG}
	$<$<BOOL:${THREAD}>:-qsmp=omp>
	$<$<BOOL:${THREAD}>:-WF,-DWITHOMP>
	$<$<BOOL:${IBMHPM}>:-WF,-DWITHBGQHPM>
	$<$<BOOL:${USEGREEDY}>:-WF,-DUSEGREEDY>
	$<$<BOOL:${USEMETIS}>:-WF,-DUSEMETIS>
	CACHE
	STRING
	"IBM compile options"
)
set(SNACFE_COMPILE_OPTIONS_IBM
	$<$<BOOL:${IBMHPM}>:/home/morozov/lib/libmpihpm.so>
	$<$<BOOL:${IBMHPM}>:/bgsys/drivers/ppcfloor/bgpm/lib/libbgpm.so>
	$<$<BOOL:${USEMETIS}>:../metis/libmetis.a>
)

target_compile_definitions(${TARGET_NAME}
	PRIVATE
		$<$<Fortran_COMPILER_ID:Intel>:${SNACFE_COMPILE_DEFNS_INTEL}>
		$<$<Fortran_COMPILER_ID:GNU>:${SNACFE_COMPILE_DEFNS_GNU}>
		$<$<Fortran_COMPILER_ID:XL>:${SNACFE_COMPILE_DEFNS_XL}>
)

target_compile_options(${TARGET_NAME}
	PRIVATE
		$<$<Fortran_COMPILER_ID:Intel>:${SNACFE_COMPILE_OPTIONS_INTEL}>
		$<$<Fortran_COMPILER_ID:GNU>:${SNACFE_COMPILE_OPTIONS_GNU}>
		$<$<Fortran_COMPILER_ID:XL>:${SNACFE_COMPILE_OPTIONS_XL}>
)

target_link_libraries(${TARGET_NAME}
	PRIVATE
		$<$<Fortran_COMPILER_ID:Intel>:${SNACFE_LINK_LIBS_INTEL}>
		$<$<Fortran_COMPILER_ID:GNU>:${SNACFE_LINK_LIBS_GNU}>
		$<$<Fortran_COMPILER_ID:XL>:${SNACFE_LINK_LIBS_XL}>
)
target_include_directories(${TARGET_NAME}
	PRIVATE
		$<$<Fortran_COMPILER_ID:Intel>:${SNACFE_INCLUDE_DIRS_INTEL}>
)
