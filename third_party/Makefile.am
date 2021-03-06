# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License

# This Makefile is for building third_party packages from
# tarballs. For autotools-based packages, we configure each of the
# packages to build static PIC binaries which we can safely link into
# a shared libmesos, and build it in-place without installing it (even
# if one runs 'make install' in this directory). Non-autotools based
# packages may be special cases; this Makefile is responsible for
# passing any special make or configure flags that might be required.

BUILT_SOURCES = # Initialized to enable using +=.

SUBDIRS = libprocess

# We need to add '--srcdir=.' needed because 'make distcheck' adds
#  '--srcdir=...' when configuring.
CONFIGURE_ARGS = @CONFIGURE_ARGS@ --enable-shared=no --with-pic --srcdir=.

include versions.am

BOTO = boto-$(BOTO_VERSION)
DISTRIBUTE = distribute-$(DISTRIBUTE_VERSION)
LEVELDB = leveldb
ZOOKEEPER = zookeeper-$(ZOOKEEPER_VERSION)


EXTRA_DIST =		\
  $(BOTO).zip		\
  $(DISTRIBUTE).tar.gz	\
  $(LEVELDB).tar.gz	\
  $(ZOOKEEPER).tar.gz

CLEAN_EXTRACTED =	\
  $(BOTO)		\
  $(DISTRIBUTE)		\
  $(LEVELDB)		\
  $(ZOOKEEPER)


# This is where the magic happens: we use stamp files as dependencies
# which cause the packages to get extracted as necessary. We also
# apply any patches as appropriate.
%-stamp: %.tar.gz
	gzip -d -c $^ | tar xf -
	test ! -e $(srcdir)/$*.patch || patch -d $* -p1 <$(srcdir)/$*.patch
	touch $@


if HAS_PYTHON
DISTRIBUTE_EGG = \
  $(DISTRIBUTE)/dist/$(DISTRIBUTE)$(PYTHON_EGG_PUREPY_POSTFIX).egg

$(DISTRIBUTE_EGG): $(DISTRIBUTE)-stamp
	cd $(DISTRIBUTE) && $(PYTHON) setup.py bdist_egg
endif


# TODO(charles): Figure out PIC options in our configure.ac or create
# a configure.ac for leveldb.
$(LEVELDB)/libleveldb.a: $(LEVELDB)-stamp
	cd $(LEVELDB) && \
          $(MAKE) $(AM_MAKEFLAGS) CC="$(CXX)" OPT="$(CXXFLAGS) -fPIC"

$(ZOOKEEPER)/src/c/libzookeeper_mt.la: $(ZOOKEEPER)-stamp
	cd $(ZOOKEEPER)/src/c && ./configure $(CONFIGURE_ARGS) && \
          $(MAKE) $(AM_MAKEFLAGS)

# Dependencies for all-local.
ALL_LOCAL = $(LEVELDB)/libleveldb.a

if HAS_PYTHON
  ALL_LOCAL += $(DISTRIBUTE_EGG)
endif

if WITH_INCLUDED_ZOOKEEPER
  ALL_LOCAL += $(ZOOKEEPER)/src/c/libzookeeper_mt.la
else
  # Now matter what we need to extract ZooKeeper so that we can run
  # 'make check' (some of our tests need the ZooKeeper JAR).
  ALL_LOCAL += $(ZOOKEEPER)-stamp
endif

all-local: $(ALL_LOCAL)

clean-local:
	rm -r -f $(CLEAN_EXTRACTED)
	rm -f *-stamp
