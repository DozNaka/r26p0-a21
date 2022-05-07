# SPDX-License-Identifier: GPL-2.0 WITH Linux-syscall-note
#
# (C) COPYRIGHT 2012-2021 ARM Limited. All rights reserved.
#
# This program is free software and is provided to you under the terms of the
# GNU General Public License version 2 as published by the Free Software
# Foundation, and any use by you of this program is subject to the terms
# of such GNU license.
#
# This program is distributed in the hope that it will be useful,
# but WITHOUT ANY WARRANTY; without even the implied warranty of
# MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the
# GNU General Public License for more details.
#
# You should have received a copy of the GNU General Public License
# along with this program; if not, you can access it online at
# http://www.gnu.org/licenses/gpl-2.0.html.
#
#

# make $(src) as absolute path if it is not already, by prefixing $(srctree)
# This is to prevent any build issue due to wrong path.
src:=$(if $(patsubst /%,,$(src)),$(srctree)/$(src),$(src))

#
# Prevent misuse when Kernel configurations are not present by default
# in out-of-tree builds
#
ifneq ($(CONFIG_ANDROID),n)
ifeq ($(CONFIG_GPU_TRACEPOINTS),n)
    $(error CONFIG_GPU_TRACEPOINTS must be set in Kernel configuration)
endif
endif

ifeq ($(CONFIG_DMA_SHARED_BUFFER),n)
    $(error CONFIG_DMA_SHARED_BUFFER must be set in Kernel configuration)
endif

ifeq ($(CONFIG_PM_DEVFREQ),n)
    $(error CONFIG_PM_DEVFREQ must be set in Kernel configuration)
endif

ifeq ($(CONFIG_DEVFREQ_THERMAL),n)
    $(error CONFIG_DEVFREQ_THERMAL must be set in Kernel configuration)
endif

ifeq ($(CONFIG_DEVFREQ_GOV_SIMPLE_ONDEMAND),n)
    $(error CONFIG_DEVFREQ_GOV_SIMPLE_ONDEMAND must be set in Kernel configuration)
endif

ifeq ($(CONFIG_FW_LOADER), n)
    $(error CONFIG_FW_LOADER must be set in Kernel configuration)
endif

ifeq ($(CONFIG_MALI_PRFCNT_SET_SELECT_VIA_DEBUG_FS), y)
    ifneq ($(CONFIG_DEBUG_FS), y)
        $(error CONFIG_MALI_PRFCNT_SET_SELECT_VIA_DEBUG_FS depends on CONFIG_DEBUG_FS to be set in Kernel configuration)
    endif
endif

ifeq ($(CONFIG_MALI_FENCE_DEBUG), y)
    ifneq ($(CONFIG_SYNC), y)
        ifneq ($(CONFIG_SYNC_FILE), y)
            $(error CONFIG_MALI_FENCE_DEBUG depends on CONFIG_SYNC || CONFIG_SYNC_FILE to be set in Kernel configuration)
        endif
    endif
endif

#
# Configurations
#

# Driver version string which is returned to userspace via an ioctl
MALI_RELEASE_NAME ?= '"r37p0-01eac0"'
# Set up defaults if not defined by build system
ifeq ($(CONFIG_MALI_DEBUG), y)
    MALI_UNIT_TEST = 1
    MALI_CUSTOMER_RELEASE ?= 0
else
    MALI_UNIT_TEST ?= 0
    MALI_CUSTOMER_RELEASE ?= 1
endif
MALI_COVERAGE ?= 0

# Kconfig passes in the name with quotes for in-tree builds - remove them.
MALI_PLATFORM_DIR := $(shell echo $(CONFIG_MALI_PLATFORM_NAME))

ifeq ($(CONFIG_MALI_CSF_SUPPORT),y)
    MALI_JIT_PRESSURE_LIMIT_BASE = 0
    MALI_USE_CSF = 1
else
    MALI_JIT_PRESSURE_LIMIT_BASE ?= 1
    MALI_USE_CSF ?= 0
endif


ifneq ($(CONFIG_MALI_KUTF), n)
    MALI_KERNEL_TEST_API ?= 1
else
    MALI_KERNEL_TEST_API ?= 0
endif

# Experimental features (corresponding -D definition should be appended to
# ccflags-y below, e.g. for MALI_EXPERIMENTAL_FEATURE,
# -DMALI_EXPERIMENTAL_FEATURE=$(MALI_EXPERIMENTAL_FEATURE) should be appended)
#
# Experimental features must default to disabled, e.g.:
# MALI_EXPERIMENTAL_FEATURE ?= 0
MALI_INCREMENTAL_RENDERING_JM ?= 0

#
# ccflags
#
ccflags-y = \
    -DMALI_CUSTOMER_RELEASE=$(MALI_CUSTOMER_RELEASE) \
    -DMALI_USE_CSF=$(MALI_USE_CSF) \
    -DMALI_KERNEL_TEST_API=$(MALI_KERNEL_TEST_API) \
    -DMALI_UNIT_TEST=$(MALI_UNIT_TEST) \
    -DMALI_COVERAGE=$(MALI_COVERAGE) \
    -DMALI_RELEASE_NAME=$(MALI_RELEASE_NAME) \
    -DMALI_JIT_PRESSURE_LIMIT_BASE=$(MALI_JIT_PRESSURE_LIMIT_BASE) \
    -DMALI_INCREMENTAL_RENDERING_JM=$(MALI_INCREMENTAL_RENDERING_JM) \
    -DMALI_PLATFORM_DIR=$(MALI_PLATFORM_DIR)


# MALI_SEC_INTEGRATION : rename CONFIG_MALI_PLATFORM_NAME to CONFIG_MALI_PLATFORM_THIRDPARTY_NAME
ifeq ($(KBUILD_EXTMOD),)
# in-tree
    ccflags-y +=-DMALI_KBASE_PLATFORM_PATH=../../$(src)/platform/$(CONFIG_MALI_PLATFORM_THIRDPARTY_NAME)
else
# out-of-tree
    ccflags-y +=-DMALI_KBASE_PLATFORM_PATH=$(src)/platform/$(CONFIG_MALI_PLATFORM_THIRDPARTY_NAME)
endif

ccflags-y += \
    -I$(srctree)/include/linux \
    -I$(srctree)/drivers/staging/android \
    -I$(src) \
    -I$(src)/platform/$(MALI_PLATFORM_DIR) \
    -I$(src)/../../../base \
    -I$(src)/../../../../include

subdir-ccflags-y += $(ccflags-y)

#
# Kernel Modules
#
obj-$(CONFIG_MALI_MIDGARD) += mali_kbase.o
obj-$(CONFIG_MALI_ARBITRATION) += arbitration/
obj-$(CONFIG_MALI_KUTF)    += tests/

mali_kbase-y := \
    mali_kbase_cache_policy.o \
    mali_kbase_ccswe.o \
    mali_kbase_mem.o \
    mali_kbase_mem_pool_group.o \
    mali_kbase_native_mgm.o \
    mali_kbase_ctx_sched.o \
    mali_kbase_gpuprops.o \
    mali_kbase_pm.o \
    mali_kbase_config.o \
    mali_kbase_kinstr_prfcnt.o \
    mali_kbase_vinstr.o \
    mali_kbase_hwcnt.o \
    mali_kbase_hwcnt_gpu.o \
    mali_kbase_hwcnt_gpu_narrow.o \
    mali_kbase_hwcnt_types.o \
    mali_kbase_hwcnt_virtualizer.o \
    mali_kbase_hwcnt_watchdog_if_timer.o \
    mali_kbase_softjobs.o \
    mali_kbase_hw.o \
    mali_kbase_debug.o \
    mali_kbase_gpu_memory_debugfs.o \
    mali_kbase_mem_linux.o \
    mali_kbase_core_linux.o \
    mali_kbase_mem_profile_debugfs.o \
    mali_kbase_disjoint_events.o \
    mali_kbase_debug_mem_view.o \
    mali_kbase_debug_mem_zones.o \
    mali_kbase_smc.o \
    mali_kbase_mem_pool.o \
    mali_kbase_mem_pool_debugfs.o \
    mali_kbase_debugfs_helper.o \
    mali_kbase_strings.o \
    mali_kbase_as_fault_debugfs.o \
    mali_kbase_regs_history_debugfs.o \
    mali_kbase_dvfs_debugfs.o \
    mali_power_gpu_frequency_trace.o \
    mali_kbase_trace_gpu_mem.o \
    mali_kbase_pbha.o

mali_kbase-$(CONFIG_DEBUG_FS) += mali_kbase_pbha_debugfs.o

mali_kbase-$(CONFIG_MALI_CINSTR_GWT) += mali_kbase_gwt.o

# Kconfig passes in the name with quotes for in-tree builds - remove them.
# MALI_SEC_INTEGRATION : rename CONFIG_MALI_PLATFORM_NAME to CONFIG_MALI_PLATFORM_THIRDPARTY_NAME
platform_name := $(shell echo $(CONFIG_MALI_PLATFORM_THIRDPARTY_NAME))
MALI_PLATFORM_DIR := platform/$(platform_name)
ccflags-y += -I$(src)/$(MALI_PLATFORM_DIR)
#include $(src)/$(MALI_PLATFORM_DIR)/Kbuild
obj-$(CONFIG_MALI_MIDGARD) += platform/
#mali_kbase-y += $(PLATFORM_THIRDPARTY:.c=.o)

mali_kbase-$(CONFIG_SYNC) += \
    mali_kbase_sync_android.o \
    mali_kbase_sync_common.o

mali_kbase-$(CONFIG_SYNC_FILE) += \
    mali_kbase_fence_ops.o \
    mali_kbase_sync_file.o \
    mali_kbase_sync_common.o

ifeq ($(CONFIG_MALI_CSF_SUPPORT),y)
    mali_kbase-y += \
        mali_kbase_hwcnt_backend_csf.o \
        mali_kbase_hwcnt_backend_csf_if_fw.o
else
    mali_kbase-y += \
        mali_kbase_jm.o \
        mali_kbase_hwcnt_backend_jm.o \
        mali_kbase_hwcnt_backend_jm_watchdog.o \
        mali_kbase_dummy_job_wa.o \
        mali_kbase_debug_job_fault.o \
        mali_kbase_event.o \
        mali_kbase_jd.o \
        mali_kbase_jd_debugfs.o \
        mali_kbase_js.o \
        mali_kbase_js_ctx_attr.o \
        mali_kbase_kinstr_jm.o

    mali_kbase-$(CONFIG_MALI_DMA_FENCE) += \
        mali_kbase_fence_ops.o \
        mali_kbase_dma_fence.o \
        mali_kbase_fence.o

    mali_kbase-$(CONFIG_SYNC_FILE) += \
        mali_kbase_fence_ops.o \
        mali_kbase_fence.o
endif


INCLUDE_SUBDIR = \
    $(src)/context/Kbuild \
    $(src)/debug/Kbuild \
    $(src)/device/Kbuild \
    $(src)/backend/gpu/Kbuild \
    $(src)/mmu/Kbuild \
    $(src)/tl/Kbuild \
    $(src)/gpu/Kbuild \
    $(src)/thirdparty/Kbuild \
    $(src)/platform/$(MALI_PLATFORM_DIR)/Kbuild

ifeq ($(CONFIG_MALI_CSF_SUPPORT),y)
    INCLUDE_SUBDIR += $(src)/csf/Kbuild
endif

ifeq ($(CONFIG_MALI_ARBITER_SUPPORT),y)
    INCLUDE_SUBDIR += $(src)/arbiter/Kbuild
endif

ifeq ($(CONFIG_MALI_DEVFREQ),y)
    ifeq ($(CONFIG_DEVFREQ_THERMAL),y)
        INCLUDE_SUBDIR += $(src)/ipa/Kbuild
    endif
endif

ifeq ($(KBUILD_EXTMOD),)
# in-tree
    -include $(INCLUDE_SUBDIR)
else
# out-of-tree
    include $(INCLUDE_SUBDIR)
endif
