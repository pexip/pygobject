gnome = import('gnome')

host_system = host_machine.system()

cc = meson.get_compiler('c')

visibility_args = []
if get_option('default_library') != 'static'
  if host_system == 'windows'
    visibility_args += ['-DDLL_EXPORT']
    if cc.get_id() == 'msvc'
      visibility_args += ['-D_GI_EXTERN=__declspec(dllexport) extern']
    elif cc.has_argument('-fvisibility=hidden')
      visibility_args += ['-D_GI_EXTERN=__attribute__((visibility("default"))) __declspec(dllexport) extern']
      visibility_args += ['-fvisibility=hidden']
    endif
  elif cc.has_argument('-fvisibility=hidden')
    visibility_args += ['-D_GI_EXTERN=__attribute__((visibility("default"))) extern']
    visibility_args += ['-fvisibility=hidden']
  endif
endif

if gi_dep.type_name() == 'pkgconfig'
  gi_datadir = gi_dep.get_pkgconfig_variable('gidatadir')
  regress_sources = [join_paths(gi_datadir, 'tests', 'regress.c')]
  regress_headers = [join_paths(gi_datadir, 'tests', 'regress.h')]
  regress_incdir = include_directories(join_paths(gi_datadir, 'tests'))
  marshalling_sources = [join_paths(gi_datadir, 'tests', 'gimarshallingtests.c')]
  marshalling_headers = [join_paths(gi_datadir, 'tests', 'gimarshallingtests.h')]
else
  gi_subproject = subproject('gobject-introspection')
  regress_sources = gi_subproject.get_variable('test_regress_sources')
  regress_headers = gi_subproject.get_variable('test_regress_headers')
  regress_incdir = gi_subproject.get_variable('test_regress_incdirs')
  marshalling_sources = gi_subproject.get_variable('test_marshalling_sources')
  marshalling_headers = gi_subproject.get_variable('test_marshalling_headers')
  gi_datadir = join_paths(meson.source_root(), 'subprojects', 'gobject-introspection', 'tests')
  message('OK so @0@'.format(gi_datadir))
endif

marshalling_sources += ['gimarshallingtestsextra.c']

marshalling_headers += ['gimarshallingtestsextra.h']

message (visibility_args)

marshalling_lib = library(
  'gimarshallingtests',
  sources : marshalling_sources,
  dependencies : [glib_dep, gobject_dep, gio_dep, gmodule_dep],
  include_directories : regress_incdir,
  c_args : visibility_args,
)

gnome.generate_gir(
  marshalling_lib,
  sources : marshalling_sources + marshalling_headers,
  nsversion : '1.0',
  namespace : 'GIMarshallingTests',
  dependencies : [glib_dep, gobject_dep, gio_dep, gmodule_dep],
  symbol_prefix : 'gi_marshalling_tests',
  includes : ['Gio-2.0'],
  build_by_default : true,
  gir_dep : gi_dep,
)

regress_sources += ['regressextra.c']

regress_headers += ['regressextra.h']

regress_lib = library(
  'regress',
  sources : regress_sources,
  dependencies : [glib_dep, gobject_dep, gio_dep, gmodule_dep],
  include_directories : regress_incdir,
  c_args: ['-D_GI_DISABLE_CAIRO'] + visibility_args
)

gnome.generate_gir(
  regress_lib,
  sources : regress_sources + regress_headers,
  nsversion : '1.0',
  namespace : 'Regress',
  includes : ['Gio-2.0'],
  build_by_default : true,
  dependencies : [glib_dep, gobject_dep, gio_dep, gmodule_dep],
  extra_args : ['-D_GI_DISABLE_CAIRO'],
  gir_dep : gi_dep
)

helper_sources = [
  'testhelpermodule.c',
  'test-floating.c',
  'test-thread.c',
  'test-unknown.c']

helperext = python.extension_module('testhelper', helper_sources,
  dependencies : [python_dep, glib_dep, gobject_dep],
  include_directories: include_directories(join_paths('..', 'gi'))
)

schemas = gnome.compile_schemas(build_by_default: true)

envdata = environment()
envdata.append('GI_TYPELIB_PATH', meson.current_build_dir(), separator : ':')
if host_machine.system() == 'linux'
    envdata.prepend('LD_LIBRARY_PATH', meson.current_build_dir())
endif
if host_machine.system() == 'windows'
    envdata.prepend('PATH', join_paths(get_option('prefix'), get_option('bindir')))
endif
envdata.append('PYTHONPATH', join_paths(meson.current_build_dir(), '..'))
envdata.append('TESTS_BUILDDIR', meson.current_build_dir())

message(join_paths(meson.current_build_dir(), '..'))

test('pygobject-test-suite', python,
  args : [join_paths(meson.current_source_dir(), 'runtests.py')],
  env : envdata)
