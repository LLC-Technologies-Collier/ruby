
SHELL = /bin/sh
NULLCMD = :

#### Start of system configuration section. ####

VPATH = $(srcdir):$(srcdir)/missing

PURIFY =
AUTOCONF = autoconf
@SET_MAKE@
MKFILES = @MAKEFILES@

exec_prefix = @exec_prefix@

TESTUI = console
TESTS =

EXTOUT = @EXTOUT@
RIDATADIR = $(DESTDIR)$(datadir)/ri/$(MAJOR).$(MINOR)/system

empty =
OUTFLAG = @OUTFLAG@$(empty)
CFLAGS = @CFLAGS@ @XCFLAGS@ @ARCH_FLAG@
debugflags = @debugflags@
CPPFLAGS = -I. -I$(srcdir) @CPPFLAGS@
LDFLAGS = @STATIC@ $(CFLAGS) @LDFLAGS@
EXTLDFLAGS = 
XLDFLAGS = @XLDFLAGS@ $(EXTLDFLAGS)
EXTLIBS = 
LIBS = @LIBS@ $(EXTLIBS)
MISSING = @LIBOBJS@ @ALLOCA@
LDSHARED = @LIBRUBY_LDSHARED@
DLDFLAGS = @LIBRUBY_DLDFLAGS@ $(EXTLDFLAGS) @ARCH_FLAG@

EXEEXT = @EXEEXT@
PROGRAM=$(RUBY_INSTALL_NAME)$(EXEEXT)
RUBY = $(RUBY_INSTALL_NAME)
MINIRUBY = @MINIRUBY@ $(MINIRUBYOPT)
RUNRUBY = @RUNRUBY@ $(RUNRUBYOPT) --

DISTCLEANFILES = \
        ext/dl/call.func \
        ext/dl/callback.func \
        ext/dl/cbtable.func \
        ext/dl/dlconfig.h \
        ext/dl/dlconfig.rb \
        ext/tk/config_list \
        ext/win32ole/.document


#### End of system configuration section. ####

SETUP         =

RM            = rm -f
NM            = @NM@
AR            = @AR@
ARFLAGS       = rcu
RANLIB        = @RANLIB@
AS            = @AS@
ASFLAGS       = @ASFLAGS@

OBJEXT        = @OBJEXT@

INSTALLED_LIST= .installed.list
#### End of variables

all:

.DEFAULT: all

# Prevent GNU make v3 from overflowing arg limit on SysV.
.NOEXPORT:

miniruby$(EXEEXT):
		@$(RM) $@
		$(PURIFY) $(CC) $(LDFLAGS) $(XLDFLAGS) $(MAINLIBS) $(MAINOBJ) $(MINIOBJS) $(LIBRUBY_A) $(LIBS) $(OUTFLAG)$@

$(PROGRAM):
		@$(RM) $@
		$(PURIFY) $(CC) $(LDFLAGS) $(XLDFLAGS) $(MAINLIBS) $(MAINOBJ) $(EXTOBJS) $(LIBRUBYARG) $(LIBS) $(OUTFLAG)$@

# We must `rm' the library each time this rule is invoked because "updating" a
# MAB library on Apple/NeXT (see --enable-fat-binary in configure) is not
# supported.
$(LIBRUBY_A):
		@$(RM) $@
		$(AR) $(ARFLAGS) $@ $(OBJS) $(DMYEXT)
		@-$(RANLIB) $@ 2> /dev/null || true

$(LIBRUBY_SO):
		@-$(PRE_LIBRUBY_UPDATE)
		$(LDSHARED) $(DLDFLAGS) $(OBJS) $(DLDOBJS) $(SOLIBS) $(OUTFLAG)$@
		@-$(MINIRUBY) -e 'ARGV.each{|link| File.delete link if File.exist? link; \
						  File.symlink "$(LIBRUBY_SO)", link}' \
				$(LIBRUBY_ALIASES) || true

fake.rb:	Makefile
		@echo ' \
		class Object; \
		  CROSS_COMPILING = RUBY_PLATFORM; \
		  remove_const :RUBY_PLATFORM; \
		  remove_const :RUBY_VERSION; \
		  RUBY_PLATFORM = "@arch@"; \
		  RUBY_VERSION = "@MAJOR@.@MINOR@.@TEENY@"; \
		end; \
		if RUBY_PLATFORM =~ /mswin|bccwin|mingw/; \
		  class File; \
		    remove_const :ALT_SEPARATOR; \
		    ALT_SEPARATOR = "\\\\"; \
		  end; \
		end; \
		prehook = proc do |e|; \
		  builddir = File.expand_path(File.dirname(__FILE__)); \
		  Config::MAKEFILE_CONFIG["top_srcdir"] = $$top_srcdir = \
		    File.expand_path("@top_srcdir@", builddir) unless e; \
		  untrace_var(:$$extmk, prehook); \
		end; \
		trace_var(:$$extmk, prehook); \
		' > $@

Makefile:	$(srcdir)/Makefile.in

$(MKFILES): config.status $(srcdir)/common.mk
		MAKE=$(MAKE) $(SHELL) ./config.status
		@{ \
		    echo "all:; -@rm -f conftest.mk"; \
		    echo "conftest.mk: .force; @echo AUTO_REMAKE"; \
		    echo ".force:"; \
		} > conftest.mk || exit 1; \
		$(MAKE) -f conftest.mk | grep '^AUTO_REMAKE$$' >/dev/null 2>&1 || \
		{ echo "Makefile updated, restart."; exit 1; }

CONFIGURE_DEPENDENCIES = $(srcdir)/configure.ac

lex.c: keywords
	( gperf -C -p -j1 -i 1 -g -o -t -N rb_reserved_word -k1,3,$$ $? > $@.tmp && mv $@.tmp $@ ) || \
	if test -f $@; then \
	  touch $@ && echo $@ touched.; \
	else \
	  cp $(srcdir)/lex.c $@ && echo $@ copied.; \
	fi

.y.c:
	$(YACC) $<
	sed '/^#/s|y\.tab\.c|$@|' y.tab.c > $@
	rm -f y.tab.c

.c.@OBJEXT@:
	$(CC) $(CFLAGS) $(CPPFLAGS) -c $<

.s.@OBJEXT@:
	$(AS) $(ASFLAGS) -o $@ $<

clean-local::
	@$(RM) ext/extinit.c ext/extinit.$(OBJEXT)

distclean-local::
	@$(RM) ext/config.cache $(RBCONFIG)

ext/extinit.$(OBJEXT): ext/extinit.c $(SETUP)
	$(CC) $(CFLAGS) $(XCFLAGS) $(CPPFLAGS) $(OUTFLAG)$@ -c ext/extinit.c

update-rubyspec: 
	if [ -d $(srcdir)/rubyspec ]; then \
	  cd $(srcdir)/rubyspec/mspec; \
	  git pull; \
	  cd ../spec; \
	  git pull; \
	else \
	  mkdir $(srcdir)/rubyspec; \
	  git clone $(MSPEC_GIT_URL) $(srcdir)/rubyspec/mspec; \
	  git clone $(RUBYSPEC_GIT_URL) $(srcdir)/rubyspec/spec; \
	fi

test-rubyspec:
	@if [ ! -d $(srcdir)/rubyspec ]; then echo No rubyspec here.  make update-rubyspec first.; exit 1; fi
	RUBY_EXE="$(RUNRUBY)" $(RUNRUBY) $(srcdir)/rubyspec/mspec/bin/mspec-run --background --prefix $(srcdir)/rubyspec/spec -B $(srcdir)/rubyspec/spec/ruby.$(MAJOR).$(MINOR).mspec
