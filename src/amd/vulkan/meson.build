# Copyright © 2017 Intel Corporation

# Permission is hereby granted, free of charge, to any person obtaining a copy
# of this software and associated documentation files (the "Software"), to deal
# in the Software without restriction, including without limitation the rights
# to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
# copies of the Software, and to permit persons to whom the Software is
# furnished to do so, subject to the following conditions:

# The above copyright notice and this permission notice shall be included in
# all copies or substantial portions of the Software.

# THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
# IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
# FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
# AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
# LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
# OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
# SOFTWARE.

radv_entrypoints = custom_target(
  'radv_entrypoints',
  input : [vk_entrypoints_gen, vk_api_xml],
  output : ['radv_entrypoints.h', 'radv_entrypoints.c'],
  command : [
    prog_python, '@INPUT0@', '--xml', '@INPUT1@', '--proto', '--weak',
    '--out-h', '@OUTPUT0@', '--out-c', '@OUTPUT1@', '--prefix', 'radv',
    '--device-prefix', 'sqtt', '--device-prefix', 'metro_exodus',
  ],
  depend_files : vk_entrypoints_gen_depend_files,
)

libradv_files = files(
  'layers/radv_metro_exodus.c',
  'layers/radv_sqtt_layer.c',
  'winsys/null/radv_null_bo.c',
  'winsys/null/radv_null_bo.h',
  'winsys/null/radv_null_cs.c',
  'winsys/null/radv_null_cs.h',
  'winsys/null/radv_null_winsys.c',
  'winsys/null/radv_null_winsys_public.h',
  'radv_acceleration_structure.c',
  'radv_acceleration_structure.h',
  'radv_android.c',
  'radv_cmd_buffer.c',
  'radv_cs.h',
  'radv_debug.c',
  'radv_debug.h',
  'radv_device.c',
  'radv_descriptor_set.c',
  'radv_descriptor_set.h',
  'radv_formats.c',
  'radv_image.c',
  'radv_meta.c',
  'radv_meta.h',
  'radv_meta_blit.c',
  'radv_meta_blit2d.c',
  'radv_meta_buffer.c',
  'radv_meta_bufimage.c',
  'radv_meta_clear.c',
  'radv_meta_copy.c',
  'radv_meta_copy_vrs_htile.c',
  'radv_meta_dcc_retile.c',
  'radv_meta_decompress.c',
  'radv_meta_etc_decode.c',
  'radv_meta_fast_clear.c',
  'radv_meta_fmask_copy.c',
  'radv_meta_fmask_expand.c',
  'radv_meta_resolve.c',
  'radv_meta_resolve_cs.c',
  'radv_meta_resolve_fs.c',
  'radv_nir_lower_ycbcr_textures.c',
  'radv_pass.c',
  'radv_perfcounter.c',
  'radv_pipeline.c',
  'radv_pipeline_cache.c',
  'radv_pipeline_rt.c',
  'radv_private.h',
  'radv_radeon_winsys.h',
  'radv_shader.c',
  'radv_shader.h',
  'radv_shader_args.c',
  'radv_shader_args.h',
  'radv_shader_info.c',
  'radv_spm.c',
  'radv_sqtt.c',
  'radv_query.c',
  'radv_util.c',
  'radv_wsi.c',
  'si_cmd_buffer.c',
  'vk_format.h',
)

if not with_platform_windows
  libradv_files += files(
    'winsys/amdgpu/radv_amdgpu_bo.c',
    'winsys/amdgpu/radv_amdgpu_bo.h',
    'winsys/amdgpu/radv_amdgpu_cs.c',
    'winsys/amdgpu/radv_amdgpu_cs.h',
    'winsys/amdgpu/radv_amdgpu_surface.c',
    'winsys/amdgpu/radv_amdgpu_surface.h',
    'winsys/amdgpu/radv_amdgpu_winsys.c',
    'winsys/amdgpu/radv_amdgpu_winsys.h',
    'winsys/amdgpu/radv_amdgpu_winsys_public.h',
  )
endif

if with_llvm
  libradv_files += files(
    'radv_llvm_helper.cpp',
    'radv_llvm_helper.h',
    'radv_nir_to_llvm.c',
  )
endif

radv_deps = []
radv_flags = cc.get_supported_arguments(['-Wimplicit-fallthrough', '-Wshadow'])

if with_platform_x11
  radv_deps += dep_xcb_dri3
endif

if with_platform_wayland
  radv_deps += dep_wayland_client
endif

if system_has_kms_drm and not with_platform_android
  libradv_files += files('radv_wsi_display.c')
endif

if with_xlib_lease
  radv_deps += [dep_xlib_xrandr]
endif

if with_platform_android
  radv_deps += dep_android
endif

# When static linking LLVM, all its symbols are public API.
# That may cause symbol collision, so explicitly demote everything.
libvulkan_radeon_ld_args = []
libvulkan_radeon_link_depends = []

if with_ld_version_script
  libvulkan_radeon_ld_args += ['-Wl,--version-script', join_paths(meson.current_source_dir(), 'vulkan.sym')]
  libvulkan_radeon_link_depends += files('vulkan.sym')
endif

vulkan_radv_def = 'vulkan_radv.def'

libvulkan_radeon = shared_library(
  'vulkan_radeon',
  [libradv_files, radv_entrypoints, sha1_h],
  vs_module_defs : vulkan_radv_def,
  include_directories : [
    inc_include, inc_src, inc_mapi, inc_mesa, inc_gallium, inc_gallium_aux, inc_amd, inc_amd_common, inc_amd_common_llvm, inc_compiler, inc_util,
  ],
  link_with : [
    libamd_common, libamd_common_llvm, libamdgpu_addrlib,
  ],
  dependencies : [
    dep_llvm, dep_libdrm_amdgpu, dep_thread, dep_elf, dep_dl, dep_m,
    dep_valgrind, radv_deps, idep_aco,
    idep_mesautil, idep_nir, idep_vulkan_util, idep_vulkan_wsi,
    idep_vulkan_runtime, idep_amdgfxregs_h, idep_xmlconfig,
  ],
  c_args : [no_override_init_args, radv_flags, c_msvc_compat_args],
  cpp_args : [radv_flags, cpp_msvc_compat_args],
  link_args : [
    ld_args_build_id, ld_args_bsymbolic, ld_args_gc_sections, libvulkan_radeon_ld_args,
  ],
  link_depends : [libvulkan_radeon_link_depends,],
  gnu_symbol_visibility : 'hidden',
  install : true,
)

if with_symbols_check
  test(
    'radv symbols check',
    symbols_check,
    args : [
      '--lib', libvulkan_radeon,
      '--symbols-file', vulkan_icd_symbols,
      '--ignore-symbol', 'ac_init_shared_llvm_once',
      symbols_check_args,
    ],
    suite : ['amd'],
  )
endif

icd_lib_path = join_paths(get_option('prefix'), get_option('libdir'))
icd_file_name = 'libvulkan_radeon.so'
if with_platform_windows
  icd_lib_path = '.'
  icd_file_name = 'vulkan_radeon.dll'
endif

icd_command = [
  prog_python, '@INPUT0@',
  '--api-version', '1.2', '--xml', '@INPUT1@',
  '--lib-path', join_paths(icd_lib_path, icd_file_name),
  '--out', '@OUTPUT@',
]
if with_platform_windows
  icd_command += '--use-backslash'
endif

radeon_icd = custom_target(
  'radeon_icd',
  input : [vk_icd_gen, vk_api_xml],
  output : 'radeon_icd.@0@.json'.format(host_machine.cpu()),
  command : icd_command,
  build_by_default : true,
  install_dir : with_vulkan_icd_dir,
  install : true,
)

