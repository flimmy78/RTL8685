
TOPDIR = ../../
EXTRA_LOCALE_DIR:=${shell cd $(TOPDIR)/extra/locale && pwd}
include $(TOPDIR)Rules.mak

CFLAGS_wc8bit=-DCTYPE_PACKED=1
CFLAGS_mmap=
ifeq ($(strip $(HAS_WCHAR)),true)
    CFLAGS_wc8bit += -DDO_WIDE_CHAR=1
    CFLAGS_mmap += -D__WCHAR_ENABLED=1
endif

all: links-target

codesets.txt:
	@if [ ! -f codesets.txt ] ; then \
	    set -e; \
	    echo " "; \
	    echo "You do not have a codesets.txt file.  Please create this "; \
	    echo "file in the extra/locale directory by running something like: "; \
	    echo -e "  find $(EXTRA_LOCALE_DIR)/charmaps -name \"*.pairs\" > \\"; \
	    echo -e "        $(EXTRA_LOCALE_DIR)/codesets.txt"; \
	    echo "and then edit that file to disable/enable the codesets you wish to support. "; \
	    echo " "; \
	    /bin/false; \
	fi;

gen_wc8bit: gen_wc8bit.c
	gcc -Os -Wall $(CFLAGS_wc8bit) gen_wc8bit.c -o gen_wc8bit

c8tables.h: gen_wc8bit codesets.txt
	./gen_wc8bit `cat codesets.txt`

gen_wctype: gen_wctype.c
	gcc -Os -Wall gen_wctype.c -o gen_wctype

# Warning! Beware tr_TR toupper/tolower exceptions!
# Warning! Ignore the width table.  It will go away.
wctables.h: gen_wctype
	./gen_wctype en_US

gen_locale: gen_locale.c c8tables.h
	gcc -Os -Wall gen_locale.c -o gen_locale

# TODO: if no wide char support, we should auto-disable all UTF-8 locales
locale_tables.h: gen_locale locales.txt
	./gen_locale locales.txt

lt_defines.h: locale_tables.h
	grep "^#define" locale_tables.h > lt_defines.h

locales.txt:
	@if [ ! -f locales.txt ] ; then \
	    set -e; \
	    echo " "; \
	    echo "You do not have a locales.txt file in the extra/locale "; \
	    echo "directory, Please copy the LOCALES file to locales.txt by "; \
	    echo "running something like: "; \
	    echo "  cp $(EXTRA_LOCALE_DIR)/LOCALES \\ "; \
	    echo "      $(EXTRA_LOCALE_DIR)/locales.txt "; \
	    echo "then edit locales.txt to disable/enable the locales you wish "; \
	    echo "to support. "; \
	    echo " "; \
	    /bin/false; \
	fi;

ifneq ($(TARGET_ARCH),$(NATIVE_ARCH))

locale.mmap: gen_mmap
	echo Your locale.mmap file is either missing or out of date.
	echo The developmental code can only generate one for the
	echo target arch == native arch case.  Sorry.
else

gen_mmap: gen_mmap.c c8tables.h wctables.h locale_tables.h
	gcc $(CFLAGS_mmap) -Os -Wall -static gen_mmap.c -o gen_mmap

locale.mmap: gen_mmap
	./gen_mmap

endif

lmmtolso: lmmtolso.c
	gcc -Os -Wall lmmtolso.c -o lmmtolso

locale_data.c: lmmtolso locale.mmap
	./lmmtolso

locale_data.o: locale_data.c lt_defines.h
	$(CC) -c locale_data.c
	$(STRIPTOOL) -x -R .note -R .comment locale_data.o

uClibc_locale_data.h: c8tables.h wctables.h lt_defines.h locale_mmap.h
	grep -v "define __LC" lt_defines.h > uClibc_locale_data.h
	cat c8tables.h wctables.h locale_mmap.h >> uClibc_locale_data.h

links-target: locale_data.o uClibc_locale_data.h
	ln -sf ../../../extra/locale/locale_data.o ../../libc/misc/locale
	cp uClibc_locale_data.h ../../libc/sysdeps/linux/common/bits/

clean:
	rm -f *.[oa] *~ core
	rm -f gen_wc8bit gen_wctype gen_locale
	rm -f c8tables.h wctables.h locale_tables.h lt_defines.h
	rm -f gen_mmap locale.mmap lmmtolso
	rm -f locale_data.c locale_data.o  uClibc_locale_data.h

