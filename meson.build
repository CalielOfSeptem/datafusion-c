# Licensed to the Apache Software Foundation (ASF) under one
# or more contributor license agreements.  See the NOTICE file
# distributed with this work for additional information
# regarding copyright ownership.  The ASF licenses this file
# to you under the Apache License, Version 2.0 (the
# "License"); you may not use this file except in compliance
# with the License.  You may obtain a copy of the License at
#
#   http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing,
# software distributed under the License is distributed on an
# "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
# KIND, either express or implied.  See the License for the
# specific language governing permissions and limitations
# under the License.

project('datafusion-c', 'rust', 'c',
        default_options: [
          'c_std=c99',
        ],
        license: 'Apache-2.0',
        version: '8.0.0')

version = meson.project_version()
version_numbers = version.split('.')
version_major = version_numbers[0].to_int()
version_minor = version_numbers[1].to_int()
version_micro = version_numbers[2].to_int()

api_version = '@0@.@1@'.format(version_major, version_minor)
so_version = '@0@'.format(version_major)
library_version = version

prefix = get_option('prefix')
data_dir = get_option('datadir')
include_dir = get_option('includedir')
lib_dir = get_option('libdir')
pkg_config_dir = lib_dir / 'pkgconfig'
full_data_dir = prefix / data_dir
full_include_dir = prefix / include_dir
full_lib_dir = prefix / lib_dir
full_pkg_config_dir = prefix / pkg_config_dir

fs = import('fs')
rustc_target_libdir = run_command(find_program('rustc'), '--print=target-libdir',
                                  capture: true,
                                  check: true)
libdatafusion_output_base_dir = fs.name(fs.parent(rustc_target_libdir.stdout()))
build_type = get_option('buildtype')
libdatafusion_output_dir = libdatafusion_output_base_dir / build_type
if build_machine.system() == 'darwin'
  libdatafusion_so_path = 'libdatafusion.dylib'
  libdatafusion_so_install_dir = lib_dir
elif build_machine.system() == 'windows'
  libdatafusion_so_path = 'datafusion.dll'
  libdatafusion_so_install_dir = lib_dir
else
  libdatafusion_so_path = 'libdatafusion.so'
  libdatafusion_so_install_dir = false
endif
libdatafusion_output = [
  libdatafusion_so_path,
  'datafusion.h',
  'datafusion.pc',
  libdatafusion_output_base_dir,
]
libdatafusion_install_dir = [
  libdatafusion_so_install_dir,
  include_dir,
  pkg_config_dir,
  false,
]
if libdatafusion_so_path.endswith('.so')
  libdatafusion_so_version_path = \
    '@0@.@1@'.format(libdatafusion_so_path, version)
  libdatafusion_so_version_major_path = \
    '@0@.@1@'.format(libdatafusion_so_path, version_major)
  libdatafusion_output += [
    libdatafusion_so_version_path,
    libdatafusion_so_version_major_path,
  ]
  libdatafusion_install_dir += [
    lib_dir,
    false,
  ]
  install_symlink(libdatafusion_so_version_major_path,
                  install_dir: lib_dir,
                  pointing_to: libdatafusion_so_version_path)
  install_symlink(libdatafusion_so_path,
                  install_dir: lib_dir,
                  pointing_to: libdatafusion_so_version_major_path)
  cbuild_libdir = '${exec_prefix}' / lib_dir
elif libdatafusion_so_path.endswith('.dylib')
  libdatafusion_so_base_path = fs.replace_suffix(libdatafusion_so_path, '')
  libdatafusion_so_version_major_path = \
    '@0@.@1@.dylib'.format(libdatafusion_so_base_path, version_major)
  libdatafusion_output = [
    libdatafusion_so_version_major_path,
    libdatafusion_output[1],
    libdatafusion_output[2],
    libdatafusion_output[3],
    libdatafusion_so_path,
  ]
  libdatafusion_install_dir += [
    false,
  ]
  install_symlink(libdatafusion_so_path,
                  install_dir: lib_dir,
                  pointing_to: libdatafusion_so_version_major_path)
  cbuild_libdir = prefix / lib_dir
else
  cbuild_libdir = prefix / lib_dir
endif
libdatafusion_command = [
  find_program('python3'),
  '@CURRENT_SOURCE_DIR@' / 'build.py',
  '--file', libdatafusion_output_dir / 'datafusion.pc',
  '--file', libdatafusion_output_dir / 'include' / 'datafusion.h',
  '--shared-object', libdatafusion_output_dir / libdatafusion_so_path,
  '--version', version,
  '--',
  find_program('cargo'),
  'cbuild',
  '--color', 'always',
  '--includedir', '${prefix}/@0@'.format(include_dir),
  '--libdir', cbuild_libdir,
  '--manifest-path', '@INPUT0@',
  '--prefix', prefix,
  '--target-dir','@OUTDIR@',
]
if build_type == 'release'
  libdatafusion_command += ['--release']
endif
libdatafusion = custom_target('libdatafusion',
                              build_by_default: true,
                              command: libdatafusion_command,
                              input: [
                                'Cargo.toml',
                                'src/capi.rs',
                                'src/lib.rs',
                              ],
                              install: true,
                              install_dir: libdatafusion_install_dir,
                              output: libdatafusion_output)
libdatafusion_so = libdatafusion[0]
datafusion_h = libdatafusion[1]
datafusion_pc = libdatafusion[2]
datafusion = declare_dependency(link_with: [libdatafusion_so],
                                sources: [datafusion_h])

have_gi = dependency('gobject-introspection-1.0', required: false).found()
arrow_glib = dependency('arrow-glib')
generate_vapi = have_gi and get_option('vapi')
if generate_vapi
  add_languages('vala')
endif

subdir('datafusion-glib')
subdir('examples')
if get_option('doc')
  subdir('doc/datafusion-glib')
endif

test_run = find_program('test/run.sh')
test('unit test',
     test_run,
     env: [
       'BUILD=no'
     ])
