env_tests = environment()
env_tests.set('XDP_UNINSTALLED', '1')
env_tests.set('XDG_DATA_DIRS', meson.current_build_dir() / 'share')
env_tests.set('G_TEST_SRCDIR', meson.current_source_dir())
env_tests.set('G_TEST_BUILDDIR', meson.current_build_dir())
env_tests.set('G_DEBUG', 'gc-friendly')  # from glib-tap.mk

subdir('dbs')
subdir('portals')
subdir('services')
subdir('share')

test_db = executable(
  'testdb',
  ['testdb.c'] + db_sources,
  dependencies: [common_deps],
  include_directories: [common_includes],
  install: enable_installed_tests,
  install_dir: installed_tests_dir,
)
test(
  'testdb',
  test_db,
  env: env_tests,
  is_parallel: false,
  protocol: 'tap',
)

if enable_installed_tests
  install_data('dbs/no_tables', install_dir: installed_tests_dir / 'dbs')
endif

test_doc_portal = executable(
  'test-doc-portal',
  'can-use-fuse.c',
  'test-doc-portal.c',
  'utils.c',
  document_portal_built_sources,
  dependencies: [common_deps, fuse3_dep],
  include_directories: [common_includes],
  install: enable_installed_tests,
  install_dir: installed_tests_dir,
)
test(
  'test-doc-portal',
  test_doc_portal,
  env: env_tests,
  is_parallel: false,
  protocol: 'tap'
)

test_backends_sources = files(
  'backend/test-backends.c',
  'backend/request.c',
  'backend/session.c',
  'backend/access.c',
  'backend/account.c',
  'backend/appchooser.c',
  'backend/background.c',
  'backend/email.c',
  'backend/filechooser.c',
  'backend/inhibit.c',
  'backend/lockdown.c',
  'backend/notification.c',
  'backend/print.c',
  'backend/screenshot.c',
  'backend/settings.c',
  'backend/wallpaper.c',
  'glib-backports.c',
)

# We build this in the tests/ subdirectory so that it can be accessed
# via G_TEST_BUILT
test_backends = executable(
  'test-backends',
  test_backends_sources,
  document_portal_built_sources,
  document_portal_built_sources,
  impl_built_sources,
  dependencies: [common_deps],
  include_directories: [common_includes, src_includes],
  install: enable_installed_tests,
  install_dir: installed_tests_dir,
)

test_portals_sources = files(
  'test-portals.c',
)

if have_libportal
  test_portals_sources += files(
    'account.c',
    'background.c',
    'camera.c',
    'email.c',
    'filechooser.c',
    'inhibit.c',
    'location.c',
    'notification.c',
    'openuri.c',
    'print.c',
    'screenshot.c',
    'trash.c',
    'utils.c',
    'wallpaper.c',
    'glib-backports.c',
  )
endif

test_portals = executable(
  'test-portals',
  test_portals_sources + portal_built_sources + impl_built_sources + xdp_utils_sources + sd_escape_sources,
  dependencies: [common_deps, libportal_dep, libsystemd_dep],
  include_directories: [common_includes, xdp_utils_includes],
  c_args: ['-DXDG_DP_BUILDDIR="src"',  '-DXDG_PS_BUILDDIR="document-portal"'],
  install: enable_installed_tests,
  install_dir: installed_tests_dir,
)

# Split the portal tests into one test per portal, this makes debugging a lot
# easier.
# Keep in sync with test-portals.c
portal_tests = [
  'account',
  'background',
  'camera',
  'color',
  'email',
  'inhibit',
  'location',
  'notification',
  'openfile',
  'openuri',
  'prepareprint',
  'print',
  'savefile',
  'screenshot',
  'trash',
  'wallpaper',
]
foreach p : portal_tests
    test(
      'test-portals-@0@'.format(p),
      test_portals,
      args: ['--keep-going', '--tap', '-p', '/portal/@0@'.format(p)],
      depends: [test_backends, test_portals],
      env: env_tests,
      is_parallel: false,
      protocol: 'tap',
      suite: 'portals',
    )
endforeach

if enable_installed_tests
  install_data(
    'session.conf.in',
    'test-document-fuse.sh',
    'test-document-fuse.py',
    install_dir: installed_tests_dir
  )
endif

test_permission_store = executable(
  'test-permission-store',
  'test-permission-store.c',
  'utils.c',
  permission_store_built_sources,
  xdp_utils_sources,
  sd_escape_sources,
  dependencies: [common_deps, libsystemd_dep],
  include_directories: [common_includes, xdp_utils_includes],
  install: enable_installed_tests,
  install_dir: installed_tests_dir,
)
test(
  'test-permission-store',
  test_permission_store,
  env: env_tests,
  is_parallel: false,
  protocol: 'tap',
)

test_xdp_utils = executable(
  'test-xdp-utils',
  'test-xdp-utils.c',
  'utils.c',
  xdp_utils_sources,
  sd_escape_sources,
  dependencies: [common_deps, libsystemd_dep],
  include_directories: [common_includes, xdp_utils_includes],
  install: enable_installed_tests,
  install_dir: installed_tests_dir,
)
test(
  'test-xdp-utils',
  test_xdp_utils,
  env: env_tests,
  is_parallel: false,
  protocol: 'tap',
)

pytest = find_program('pytest-3', 'pytest', required: false)
pymod = import('python')
python = pymod.find_installation(
  'python3',
  modules: ['dbus', 'dbusmock', 'gi'],
  required: false,
)

if pytest.found() and python.found()
  test_env = environment()
  test_env.set('XDP_UNINSTALLED', '1')

  test(
    'pytest',
    pytest,
    args: ['--verbose', '--log-level=DEBUG'],
    env: test_env,
    workdir: meson.project_source_root(),
    suite: ['pytest'],
  )
endif

if enable_installed_tests
  # autotools used to symlink to the host files, here we just install our version
  install_data(
    doc_portal_service_file,
    permission_portal_service_file,
    install_dir: installed_tests_dir / 'services',
  )

  testfiles = [
    'testdb',
    'test-doc-portal',
    'test-document-fuse.sh',
    'test-permission-store',
    'test-portals',
    'test-xdp-utils',
  ]
  foreach tf : testfiles
      data = configuration_data()
      data.set('installed_testdir', installed_tests_dir)
      data.set('exec', tf)
      configure_file(
        input: 'template.test.in',
        output: '@0@.test'.format(tf),
        configuration: data,
        install: true,
        install_dir: installed_tests_data_dir,
      )
  endforeach
endif
