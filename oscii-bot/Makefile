APPNAME=osciibot

WDL_PATH = ../WDL
vpath swell%.cpp $(WDL_PATH)/swell
vpath lice%.cpp $(WDL_PATH)/lice
vpath png%.c $(WDL_PATH)/libpng
vpath %.c $(WDL_PATH) $(WDL_PATH)/zlib $(WDL_PATH)/eel2
vpath %.cpp $(WDL_PATH) $(WDL_PATH)/wingui $(WDL_PATH)/jnetlib
vpath a2x64.php $(WDL_PATH)/eel2

###### Objects and resources (probably too many)
SWELL_OBJS = swell.o swell-ini.o swell-miscdlg-generic.o swell-wnd-generic.o \
             swell-menu-generic.o swell-kb-generic.o swell-dlg-generic.o \
             swell-gdi-generic.o swell-misc-generic.o swell-gdi-lice.o \
             swell-generic-gdk.o swell-appstub-generic.o swell-modstub-generic.o

LICE_OBJS = lice_image.o lice_arc.o lice_line.o lice_png.o lice_text.o \
            lice_textnew.o lice.o lice_colorspace.o

PNG_OBJS = png.o pngerror.o pngget.o pngmem.o pngpread.o pngread.o \
           pngrio.o pngrtran.o pngrutil.o pngset.o pngtrans.o pngwio.o \
           pngwrite.o pngwtran.o pngwutil.o

ZLIB_OBJS = adler32.o compress.o crc32.o deflate.o infback.o inffast.o inflate.o \
            inftrees.o trees.o uncompr.o zutil.o

JNET_OBJS = asyncdns.o connection.o listen.o util.o

EEL_OBJS = nseel-caltab.o nseel-cfunc.o nseel-compiler.o \
           nseel-eval.o nseel-lextab.o nseel-ram.o nseel-yylex.o

ifeq ($(shell uname -m),x86_64)
  EEL_OBJS += asm-nseel-x64.o
endif

OTHER_OBJS = wndsize.o filebrowse.o

RESFILES = res.rc_mac_dlg res.rc_mac_menu

OBJS += main.o midi2osc.o oscmsg.o midi_linux.o $(SWELL_OBJS) $(PNG_OBJS) \
        $(ZLIB_OBJS) $(JNET_OBJS) $(EEL_OBJS) $(OTHER_OBJS) 


###### Compiler/Linker flags
CFLAGS += -pipe -fvisibility=hidden -fno-math-errno -fPIC -DPIC -Wall -Wshadow \
          -Wno-unused-function -Wno-multichar -Wno-unused-result
          
ARCH := $(shell uname -m)
PKG_CONFIG = pkg-config

ifndef ALLOW_WARNINGS
  CFLAGS += -Werror
endif
ifndef DEPRECATED_WARNINGS
  CFLAGS +=  -Wno-deprecated-declarations
endif

ifneq ($(filter arm%,$(ARCH)),)
  CFLAGS += -fsigned-char -marm
endif

ifdef DEBUG
  CFLAGS += -O0 -g -D_DEBUG
else
  CFLAGS += -O2 -DNDEBUG
endif

LINKEXTRA =  -lpthread -ldl 

ifndef NOGDK
  ifdef GDK2
    CFLAGS += -DSWELL_TARGET_GDK=2 $(shell $(PKG_CONFIG) --cflags gdk-2.0)
    LINKEXTRA += $(shell $(PKG_CONFIG) --libs gdk-2.0)
    LINKEXTRA += -lX11 -lXi
  else
    ifdef SWELL_SUPPORT_GTK
      CFLAGS += -DSWELL_TARGET_GDK=3 $(shell $(PKG_CONFIG) --cflags gtk+-3.0) -DSWELL_SUPPORT_GTK
    else
      CFLAGS += -DSWELL_TARGET_GDK=3 $(shell $(PKG_CONFIG) --cflags gdk-3.0)
    endif
    LINKEXTRA += -lX11 -lXi
    ifdef SWELL_SUPPORT_GTK
      LINKEXTRA += $(shell $(PKG_CONFIG) --libs gtk+-3.0)
    else
      LINKEXTRA += $(shell $(PKG_CONFIG) --libs gdk-3.0)
    endif
  endif
  CFLAGS += -DSWELL_LICE_GDI
  OBJS += $(LICE_OBJS)

  ifndef NOFREETYPE
    CFLAGS += -DSWELL_FREETYPE $(shell $(PKG_CONFIG) --cflags freetype2)
    LINKEXTRA += $(shell $(PKG_CONFIG) --libs freetype2)
    ifndef NOFONTCONFIG
      CFLAGS += -DSWELL_FONTCONFIG
      LINKEXTRA += -lfontconfig
    endif
  endif
endif

##### Jack
CFLAGS += $(shell $(PKG_CONFIG) --cflags jack)
LINKEXTRA += $(shell $(PKG_CONFIG) --libs jack)

CXXFLAGS = $(CFLAGS)


default: $(APPNAME)

.PHONY: clean run

$(RESFILES): res.rc
	php $(WDL_PATH)/swell/mac_resgen.php $^
	
asm-nseel-x64.o: a2x64.php asm-nseel-x86-gcc.c
	cd $(WDL_PATH)/eel2 && php a2x64.php elf64 && cp $@ ../../oscii-bot
	
$(APPNAME): $(RESFILES) $(OBJS)
	$(CXX) -o $@ $(CFLAGS) $(OBJS) $(LINKEXTRA)

run: $(APPNAME)
	./$^

clean:
	-rm $(OBJS) $(APPNAME) $(RESFILES) $(WDL_PATH)/eel2/asm-nseel-x64.o $(WDL_PATH)/eel2/asm-nseel-x64.asm
