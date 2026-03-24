# Copyright Claus Klein, 2025-2026
# Distributed under the Boost Software License, Version 1.0.
# See accompanying file LICENSE_1_0.txt or copy at http://www.boost.org/LICENSE_1_0.txt

# Standard stuff

.SUFFIXES:

MAKEFLAGS+= --no-builtin-rules  # Disable the built-in implicit rules.
MAKEFLAGS+= --warn-undefined-variables        # Warn when an undefined variable is referenced.

export hostSystemName=$(shell uname)

ifeq (${hostSystemName},Darwin)
  export LLVM_PREFIX:=$(shell brew --prefix llvm)
  export LLVM_DIR:=$(shell realpath ${LLVM_PREFIX})
  export PATH:=${LLVM_DIR}/bin:${PATH}

  export CMAKE_CXX_STDLIB_MODULES_JSON:=${LLVM_DIR}/lib/c++/libc++.modules.json
  export CXXFLAGS:=-stdlib=libc++
  export LDFLAGS:=-L$(LLVM_DIR)/lib/c++ # XXX -lc++abi
  export CXX:=clang++
  # FIXME: export GCOV:="llvm-cov gcov"

  ### TODO: to test g++-15:
  export GCC_PREFIX:=$(shell brew --prefix gcc)
  export GCC_DIR:=$(shell realpath ${GCC_PREFIX})

  # export CMAKE_CXX_STDLIB_MODULES_JSON=${GCC_DIR}/lib/gcc/current/libstdc++.modules.json
  # export CXXFLAGS:=-stdlib=libstdc++
  # export CXX:=g++-15
  # export GCOV="gcov"
else ifeq (${hostSystemName},Linux)
	export LLVM_DIR:=/usr/lib/llvm-23
  export PATH:=${LLVM_DIR}/bin:${PATH}
  export CXX:=clang++-23
endif

.PHONY: all install distclean format demo

all: build/compile_commands.json
	ln -sf $< .
	ninja -C build

build/compile_commands.json: CMakeLists.txt GNUmakefile
	/usr/local/bin/cmake --version
	/usr/local/bin/cmake -S . -B build -G Ninja \
	 -D CMAKE_CXX_STDLIB_MODULES_JSON=${CMAKE_CXX_STDLIB_MODULES_JSON} \
	 -D CMAKE_CXX_STANDARD=23 -D CMAKE_CXX_EXTENSIONS=YES -D CMAKE_CXX_STANDARD_REQUIRED=YES \
	 -D CMAKE_CXX_MODULE_STD=YES \
	 -D BOOST_USE_MODULES=YES \
	 -D BUILD_TESTING=YES \
	 -D CMAKE_BUILD_TYPE=Release \
	 -D CMAKE_INSTALL_MESSAGE=LAZY \
	 -D CMAKE_SKIP_INSTALL_RULES=NO \
	 --log-level=VERBOSE --fresh \
	 # --trace-expand --trace-source=use-fetch-content.cmake \
	 # --debug-find-pkg=GTest

install: build/cmake_install.cmake
	cmake --install build

distclean: # XXX clean
	rm -rf build stagedir compile_commands.json
	find . -name '*~' -delete

format: distclean
	gersemi -i --line-length 102 --no-warn-about-unknown-commands \
  cmake/*.cmake CMakeLists.txt

# Anything we don't know how to build will use this rule.
% ::
	ninja -C build $(@)
