
CFLAGS = -Wall -O2  -I /tmp/spooles.2.2 -DARCH="Linux" -DSPOOLES -DARPACK -DMATRIXSTORAGE -DNETWORKOUT
FFLAGS = -O2

CC=cc
FC=ifort

.c.o :
	$(CC) $(CFLAGS) -c $<
.f.o :
	$(FC) $(FFLAGS) -c $<

include Makefile.inc

SCCXMAIN = ccx_2.17.c

OCCXF = $(SCCXF:.f=.o)
OCCXC = $(SCCXC:.c=.o)
OCCXMAIN = $(SCCXMAIN:.c=.o)

DIR=/tmp/spooles.2.2

LIBS = \
       $(DIR)/spooles.a \
	/tmp/ARPACK/libarpack_INTEL.a \
       -lpthread -lm -lc

ccx_2.17: $(OCCXMAIN) ccx_2.17.a  $(LIBS)
	./date.pl; $(CC) $(CFLAGS) -c ccx_2.17.c; $(FC) -nofor-main -Wall -O2 -o $@ $(OCCXMAIN) ccx_2.17.a $(LIBS)

ccx_2.17.a: $(OCCXF) $(OCCXC)
	ar vr $@ $?
