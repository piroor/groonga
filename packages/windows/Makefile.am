SUBDIRS =					\
	patches					\
	language-files
EXTRA_DIST = Rakefile
LICENSE_DIR = dist-x64/share/license
ARCHITECTURES = x86 x64
SUFFIX =

all:

release: build package installer upload

ensure-rsync-path:
	@if test -z "$(RSYNC_PATH)"; then				\
	  echo "--with-rsync-path configure option must be specified.";	\
	  false;							\
	fi

download: ensure-rsync-path
	rsync -avz --progress --delete $(RSYNC_PATH)/windows/groonga/ files

upload: ensure-rsync-path
	rsync -avz --progress --delete files/ $(RSYNC_PATH)/windows/groonga

upload-to-tmp:
	for architecture in $(ARCHITECTURES); do			\
	  scp files/$(PACKAGE)-$(VERSION)-$${architecture}$(SUFFIX).zip	\
	    packages@packages.groonga.org:public/tmp/;			\
	done

build: source
	vagrant destroy --force
	@(							\
	  echo "VERSION='$(VERSION)'";				\
	  echo "SOURCE='$(SOURCE)'";				\
	  echo "DEBUG_BUILD='$(DEBUG_BUILD)'";			\
	  echo "MEMORY_DEBUG_BUILD='$(MEMORY_DEBUG_BUILD)'";	\
	  echo "ARCHITECTURES='$(ARCHITECTURES)'";		\
	) > tmp/env.sh
	vagrant up
	vagrant destroy --force

build-groonga: source
	for architecture in $(ARCHITECTURES); do			\
	  $(RUBY) -S rake build:groonga build:post $(build_options)	\
	     ARCHITECTURE=$${architecture};				\
	done

LICENSE:
	@(echo "Groonga";				\
	 echo "=======";				\
	 echo;						\
	 echo "AUTHORS";				\
	 echo "-------";				\
	 echo;						\
	 cat $(LICENSE_DIR)/groonga/AUTHORS;		\
	 echo;						\
	 echo "COPYING";				\
	 echo "-------";				\
	 echo;						\
	 cat $(LICENSE_DIR)/groonga/COPYING;		\
	 echo;						\
	 echo;						\
	 echo "MeCab";					\
	 echo "=====";					\
	 echo;						\
	 echo "AUTHORS";				\
	 echo "-------";				\
	 echo;						\
	 cat $(LICENSE_DIR)/mecab/AUTHORS;		\
	 echo;						\
	 echo "COPYING";				\
	 echo "-------";				\
	 echo;						\
	 cat $(LICENSE_DIR)/mecab/COPYING;		\
	 echo;						\
	 echo "BSD";					\
	 echo "---";					\
	 echo;						\
	 echo; cat $(LICENSE_DIR)/mecab/BSD;		\
	 echo;						\
	 echo "GPL";					\
	 echo "---";					\
	 echo;						\
	 cat $(LICENSE_DIR)/mecab/GPL;			\
	 echo;						\
	 echo "LGPL";					\
	 echo "----";					\
	 echo;						\
	 cat $(LICENSE_DIR)/mecab/LGPL;			\
	 echo;						\
	 echo;						\
	 echo "NAIST-jdic";				\
	 echo "==========";				\
	 echo;						\
	 echo "AUTHORS";				\
	 echo "-------";				\
	 echo;						\
	 cat $(LICENSE_DIR)/naist-jdic/AUTHORS;		\
	 echo;						\
	 echo "COPYING";				\
	 echo "-------";				\
	 echo;						\
	 cat $(LICENSE_DIR)/naist-jdic/COPYING;		\
	) > LICENSE

installer: installer-x86 installer-x64

setup-x86.nsi: setup-x64.nsi
	sed						\
	  -e 's/64/86/g'				\
	  -e 's/PROGRAMFILES86/PROGRAMFILES32/g'	\
	  setup-x64.nsi > setup-x86.nsi

installer-x86: LICENSE setup-x86.nsi
	makensis setup-x86.nsi

installer-x64: LICENSE setup-x64.nsi
	makensis setup-x64.nsi

package:
	for architecture in $(ARCHITECTURES); do			\
	  mkdir -p files;						\
	  rm -rf files/$(PACKAGE)-$(VERSION)-$${architecture}$(SUFFIX);	\
	  cp -a dist-$${architecture}					\
	    files/$(PACKAGE)-$(VERSION)-$${architecture}$(SUFFIX);	\
	  (cd files &&							\
	    rm -f $(PACKAGE)-$(VERSION)-$${architecture}$(SUFFIX).zip;	\
	    zip -r $(PACKAGE)-$(VERSION)-$${architecture}$(SUFFIX).zip	\
	           $(PACKAGE)-$(VERSION)-$${architecture}$(SUFFIX));	\
	  rm -rf files/$(PACKAGE)-$(VERSION)-$${architecture}$(SUFFIX);	\
	done

SOURCE=tmp/$(PACKAGE)-$(VERSION).tar.gz

source: $(SOURCE)

$(SOURCE): $(abs_top_builddir)/$(PACKAGE)-$(VERSION).tar.gz
	cp $(abs_top_builddir)/$(PACKAGE)-$(VERSION).tar.gz $(SOURCE)

diff-mecab:
	(							\
	  cd tmp/mecab;						\
	  diff -u mecab-$(MECAB_VERSION){.orig,}/configure.in;	\
	  diff -u mecab-$(MECAB_VERSION){.orig,}/src/common.h;	\
	  diff -u mecab-$(MECAB_VERSION){.orig,}/src/mmap.h;	\
	  diff -u mecab-$(MECAB_VERSION){.orig,}/src/utils.cpp;	\
	  diff -u mecab-$(MECAB_VERSION){.orig,}/src/winmain.h;	\
	  :							\
	) > patches/mecab-$(MECAB_VERSION).diff
