#
# Makefile that builds btest and other helper programs for the CS:APP data lab
#
# The authentic CS:APP data lab is built in 32-bit mode (-m32).
# gcc-multilib is now installed, so we use the original 32-bit build.
# (If you ever need 64-bit again: `make ARCH=-m64`.)
#
CC = gcc
ARCH = -m32
CFLAGS = -O -Wall $(ARCH)
LIBS = -lm

all: btest fshow ishow

btest: btest.c bits.c decl.c tests.c btest.h bits.h
	$(CC) $(CFLAGS) $(LIBS) -o btest bits.c btest.c decl.c tests.c

fshow: fshow.c
	$(CC) $(CFLAGS) -o fshow fshow.c

ishow: ishow.c
	$(CC) $(CFLAGS) -o ishow ishow.c

# Forces a recompile. Used by the driver program.
btestexplicit:
	$(CC) $(CFLAGS) $(LIBS) -o btest bits.c btest.c decl.c tests.c

clean:
	rm -f *.o btest fshow ishow *~
