project('Cutter', 'cpp', default_options: 'cpp_std=c++11', meson_version: '>=0.47.0')

qt5_mod = import('qt5')
py3_exe = import('python').find_installation('python3')

qt_modules = []
feature_define_args = []
if get_option('enable_jupyter')
  message('Jupyter support enabled')
  add_project_arguments('-DCUTTER_ENABLE_JUPYTER', language: 'cpp')
  feature_define_args += ['-DCUTTER_ENABLE_JUPYTER']
  if get_option('enable_webengine')
    message('QtWebEngine support enabled')
    add_project_arguments('-DCUTTER_ENABLE_QTWEBENGINE', language: 'cpp')
    feature_define_args += ['-DCUTTER_ENABLE_QTWEBENGINE']
    qt_modules += 'WebEngineWidgets'
  endif
endif

parse_cmd = [py3_exe, join_paths(meson.current_source_dir(), '../scripts/meson_parse_qmake.py')]

qt_modules += run_command(parse_cmd + ['QT'], check: true).stdout().split(';')
sources = run_command(parse_cmd + ['SOURCES'], check: true).stdout().split(';')
headers = run_command(parse_cmd + ['HEADERS'], check: true).stdout().split(';')
ui_files = run_command(parse_cmd + ['FORMS'], check: true).stdout().split(';')
qresources = run_command(parse_cmd + ['RESOURCES'], check: true).stdout().split(';')
version_major = run_command(parse_cmd + ['CUTTER_VERSION_MAJOR'], check: true).stdout()
version_minor = run_command(parse_cmd + ['CUTTER_VERSION_MINOR'], check: true).stdout()
version_patch = run_command(parse_cmd + ['CUTTER_VERSION_PATCH'], check: true).stdout()

conf_json = '''{"CUTTER_VERSION_MAJOR":@0@,
                "CUTTER_VERSION_MINOR":@1@,
                "CUTTER_VERSION_PATCH":@2@}'''.format(
                version_major, version_minor, version_patch)
configure_file(input: 'CutterConfig.h.in',
               output: 'CutterConfig.h',
               command: [py3_exe, join_paths(meson.current_source_dir(), '../scripts/meson_configure_qmake_in.py'), '@INPUT@', '@OUTPUT0@', conf_json])
conf_inc = include_directories('.')

sp_dir = join_paths(meson.source_root(), 'subprojects')
sp_r2_dir = join_paths(sp_dir, 'radare2')
exists_cmd = '__import__("sys").exit(__import__("os").path.exists("@0@"))'.format(sp_r2_dir)
if run_command(py3_exe, '-c', exists_cmd).returncode() == 0
  r2_src_dir = join_paths(meson.source_root(), '..', 'radare2')
  if host_machine.system() == 'windows'
    sp_dir = '\\'.join(sp_dir.split('/'))
    sp_r2_dir = '\\'.join(sp_r2_dir.split('/'))
    r2_src_dir = '\\'.join(r2_src_dir.split('/'))
    link_cmd = ['CMD', '/C', 'MKDIR', sp_dir, '&', 'MKLINK', '/D', sp_r2_dir, r2_src_dir]
  else
    link_cmd = ['sh', '-c', 'mkdir @0@ ; ln -s @1@ @2@'.format(sp_dir, r2_src_dir, sp_r2_dir)]
  endif
  run_command(link_cmd, check: true)
endif

r2 = subproject('radare2')
libr2_dep = r2.get_variable('libr2_dep')

qt5dep = dependency('qt5', modules: qt_modules)

deps = [libr2_dep, qt5dep]
if get_option('enable_jupyter')
  deps += [dependency('python3')]
endif

moc_files = qt5_mod.preprocess(
  moc_headers: headers,
  ui_files: ui_files,
  qresources: qresources,
  moc_extra_arguments: feature_define_args
)

cpp = meson.get_compiler('cpp')

platform_inc = []
if host_machine.system() == 'windows'
  add_project_arguments('-D_CRT_NONSTDC_NO_DEPRECATE', language: 'cpp')
  add_project_arguments('-D_CRT_SECURE_NO_WARNINGS', language: 'cpp')
  platform_inc = include_directories('../radare2/libr/include/msvc')
  # Workaround for https://github.com/mesonbuild/meson/issues/2327
  qt_host_libs = run_command('qmake', '-query', 'QT_HOST_LIBS').stdout().strip()
  if get_option('buildtype').startswith('debug')
    qtmain_libname = 'qtmaind.lib'
  else
    qtmain_libname = 'qtmain.lib'
  endif
  add_project_link_arguments(join_paths(qt_host_libs, qtmain_libname), language: 'cpp')
endif

cutter_exe = executable(
  'Cutter',
  moc_files,
  gui_app: true,
  sources: sources,
  include_directories: [platform_inc, conf_inc],
  dependencies: deps,
)
