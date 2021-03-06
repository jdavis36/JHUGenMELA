# All targets with # symbol are self-documenting, i.e. make help or simply make will
# show the targets among available options
#
# User targets are at the bottom
#
ifndef ROOTSYS
all:
	@echo "ROOTSYS is not set. Please set ROOT environment properly"; echo
else
ifndef ROOFITSYS
all:
	@echo "ROOFITSYS is not set. Please set ROOT environment properly"; echo
else

RM  = /bin/rm

MELADIR = $(shell pwd)
MELASRCDIR = $(MELADIR)/src
MELADATADIR = $(MELADIR)/data
# Modify MELALIBDIR for the gcc version as needed
MELALIBDIR = $(MELADATADIR)/${SCRAM_ARCH}
# _melapkgpath_ should refer to the root compilation path just like MELADIR with an extra '/'.
# If environment variables need to be inserted without expansion for portability,
# you can use '.oODOLLAROo..oOOPEN_BRACKETOo.[YOUR_ENV_VARIABLE].oOCLOSE_BRACKETOo.',
# e.g. '.oODOLLAROo..oOOPEN_BRACKETOo.CMSSW_BASE.oOCLOSE_BRACKETOo.' in place of '${CMSSW_BASE}' as used in the CMS-specific compilation.
MELAPKGPATH = $(MELADIR)/
LIB = libJHUGenMELAMELA.so

ROOTCFLAGS = $(shell root-config --cflags)
ROOTLIBS = $(shell root-config --libs) -Lrootlib
NLIBS = $(ROOTLIBS)
NLIBS += -L$(ROOFITSYS)/lib -lMinuit -lRooFitCore -lRooFit -lgfortran
LIBS = $(filter-out -lNew, $(NLIBS))

ROOFITINC = -I$(ROOFITSYS)/include
ROOTINC = $(ROOFITINC)

CC = g++
CPPINC = -I$(MELADIR)/interface $(ROOTINC)
CPPLOAD = -L$(MELALIBDIR) -lmcfm_707 -ljhugenmela $(LIBS)
CPPOPTS =  -fPIC -O2 -ftree-vectorize -fipa-pta -felide-constructors -fvisibility-inlines-hidden -fno-math-errno \
	--param vect-max-version-for-alias-checks=50 -fmessage-length=0 -fdiagnostics-show-option \
	-Werror=unused-value \
	 $(shell root-config --cflags) -D_melapkgpath_=$(MELAPKGPATH)
CPPFLAGS = $(CPPOPTS) $(CPPINC) $(CPPLOAD)

LINKER = g++
LINKERFLAGS = -Wl,-rpath=$(MELALIBDIR),-soname,$(LIB) $(CPPLOAD)

SOURCES = $(wildcard $(MELASRCDIR)/*.cc)
OBJECTS = $(SOURCES:.cc=.o)

.PHONY: all help compile clean

all: $(LIB) python/__init__.py

$(MELASRCDIR)/LinkDef_out.o: $(MELASRCDIR)/LinkDef.h
	$(QUIET) echo "Making CINT dictionaries"; \
        rootcint -f $(MELASRCDIR)/LinkDef_out.cc -c -p $(CPPINC) $(MELASRCDIR)/LinkDef.h; \
        cat $(MELASRCDIR)/LinkDef.h $(MELASRCDIR)/LinkDef_out.cc > $(MELASRCDIR)/LinkDef_out.cxx; \
        rm $(MELASRCDIR)/LinkDef_out.cc; mv $(MELASRCDIR)/LinkDef_out_rdict.pcm $(MELALIBDIR)/LinkDef_out_rdict.pcm; \
        $(CC) $(CPPOPTS) $(CPPINC) -c $(MELASRCDIR)/LinkDef_out.cxx -o $(MELASRCDIR)/LinkDef_out.o

# General rule for making object files
%.d:	$(MELASRCDIR)/%.cc $(MELASRCDIR)/%.cxx
	$(QUIET) echo "Checking dependencies for $<"; \
	$(CC) -MM -MT $@ -MT ${@:.d=.o} $(CPPFLAGS) $< > $@; \
                     [ -s $@ ] || rm -f $@

%.o: 	$(MELASRCDIR)/%.cc
	$(QUIET) echo "Compiling $<"; \
	$(CC) $(CPPFLAGS) $< -c -o $@

$(LIB):	$(OBJECTS) $(MELASRCDIR)/LinkDef_out.o
	$(QUIET) echo "Linking $(LIB)"; \
	$(LINKER) $(LINKERFLAGS) -shared $(OBJECTS) $(MELASRCDIR)/LinkDef_out.o -o $@; \
	mv $(LIB) $(MELALIBDIR)/$(LIB)

clean:
	$(QUIET) rm -v -f \
	$(MELASRCDIR)/*.o $(MELASRCDIR)/*.d $(MELASRCDIR)/LinkDef_out* $(MELALIBDIR)/$(LIB) $(MELALIBDIR)/LinkDef_out_rdict.pcm; \
	echo "Done cleaning"

python/__init__.py:
	touch python/__init__.py

endif
endif

