# Integrators
if (NOT MTS_VERSION)
  message(FATAL_ERROR "Use the top level configuration file")
endif()

macro(add_integrator)
  add_mts_plugin(${ARGN} TYPE integrator)
endmacro()

macro(add_bidir)
  add_integrator(${ARGN} MTS_BIDIR)
endmacro()
find_package(OpenMP REQUIRED)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fopenmp")
if (MTS_CUDA)
    list(APPEND CUDA_NVCC_FLAGS "-arch=sm_52;-std=c++11")
    SET(CUDA_PROPAGATE_HOST_FLAGS OFF)
    INCLUDE_DIRECTORIES(${CUDA_INCLUDE_DIRS})
    macro(add_recons)
        add_mtsCUDA_plugin(${ARGN} MTS_BIDIR TYPE integrator)
    endmacro()
    SET(RECONSFILE gradient/poisson_solver/BackendCUDA.cu)
else()
    macro(add_recons)
        add_bidir(${ARGN})
    endmacro()
    SET(RECONSFILE gradient/poisson_solver/BackendOpenMP.cpp)
endif()
macro(link_omp)
  if (OpenMP_CXX_FOUND)
    target_link_libraries(${ARGN} PUBLIC OpenMP::OpenMP_CXX)
  endif()
endmacro()

add_integrator(ao       direct/ao.cpp)
add_integrator(direct   direct/direct.cpp)
add_integrator(path     path/path.cpp)
add_integrator(volpath  path/volpath.cpp)
add_integrator(volpath_simple path/volpath_simple.cpp)
add_integrator(guided_path  path/guided_path.cpp)
add_integrator(ptracer  ptracer/ptracer.cpp
                        ptracer/ptracer_proc.h ptracer/ptracer_proc.cpp)

# Photon mapping-based techniques
add_integrator(photonmapper photonmapper/photonmapper.cpp photonmapper/bre.cpp)
add_integrator(ppm          photonmapper/ppm.cpp)
add_integrator(sppm         photonmapper/sppm.cpp)

# Miscellaneous
add_integrator(vpl          vpl/vpl.cpp MTS_HW)
add_integrator(adaptive     misc/adaptive.cpp)
add_integrator(irrcache     misc/irrcache.cpp
                            misc/irrcache_proc.h misc/irrcache_proc.cpp)
add_integrator(multichannel misc/multichannel.cpp)
add_integrator(field        misc/field.cpp)

# Bidirectional techniques
add_bidir(bdpt          bdpt/bdpt.h      bdpt/bdpt.cpp
                        bdpt/bdpt_proc.h bdpt/bdpt_proc.cpp
                        bdpt/bdpt_wr.h   bdpt/bdpt_wr.cpp)

add_bidir(pssmlt        pssmlt/pssmlt.h         pssmlt/pssmlt.cpp
                        pssmlt/pssmlt_proc.h    pssmlt/pssmlt_proc.cpp
                        pssmlt/pssmlt_sampler.h pssmlt/pssmlt_sampler.cpp)

add_bidir(mlt           mlt/mlt.h      mlt/mlt.cpp
                        mlt/mlt_proc.h mlt/mlt_proc.cpp)

add_bidir(erpt          erpt/erpt.h      erpt/erpt.cpp
                        erpt/erpt_proc.h erpt/erpt_proc.cpp)


add_bidir(drmlt         drmlt/drmlt.h         drmlt/drmlt.cpp
                        drmlt/drmlt_proc.h    drmlt/drmlt_proc.cpp
                        drmlt/drmlt_sampler.h drmlt/drmlt_sampler.cpp
                        drmlt/tools/transition.h)

add_recons(gpt      gradient/gpt/gpt.cpp   gradient/gpt/gpt_wr.cpp
        gradient/gpt/gpt_proc.cpp

        gradient/poisson_solver/Solver.cpp
        gradient/poisson_solver/Backend.cpp
        ${RECONSFILE}
        gradient/poisson_solver/Defs.cpp
  
        denoiser/nfor/Regression.cpp
        denoiser/nfor/nfor.cpp
        denoiser/nfor/core/Logging.cpp
        denoiser/nfor/core/thread/ThreadPool.cpp
        denoiser/nfor/core/thread/ThreadUtils.cpp
        )
link_omp(gpt)
add_recons(gbdpt        gradient/gbdpt/gbdpt.cpp       gradient/gbdpt/gbdpt.h
        gradient/gbdpt/gbdpt_proc.cpp  gradient/gbdpt/gbdpt_proc.h
        gradient/gbdpt/gbdpt_wr.cpp    gradient/gbdpt/gbdpt_wr.h

        gradient/poisson_solver/Solver.cpp
        gradient/poisson_solver/Backend.cpp
        ${RECONSFILE}
        gradient/poisson_solver/Defs.cpp)
link_omp(gbdpt)
add_bidir(gvcm           gradient/gvcm/gdvcm.h      gradient/gvcm/gdvcm.cpp
        gradient/gvcm/gdvcm_proc.h gradient/gvcm/gdvcm_proc.cpp
        gradient/gvcm/gdvcm_wr.h   gradient/gvcm/gdvcm_wr.cpp vcm/vcm_basics.h

        gradient/poisson_solver/Solver.cpp
        gradient/poisson_solver/Backend.cpp
        ${RECONSFILE}
        gradient/poisson_solver/Defs.cpp

        vcm/nanoflann/nanoflann.hpp
        )
link_omp(gvcm)
add_recons(gvpm        
      gradient/gvpm/gvpm.cpp
      gradient/gvpm/gvpm_proc.cpp
      gradient/gvpm/gvpm_accel.cpp
      gradient/gvpm/gvpm_gatherpoint.h
      gradient/gvpm/gvpm_struct.h

      gradient/poisson_solver/Solver.cpp
      gradient/poisson_solver/Backend.cpp
      ${RECONSFILE}
      gradient/poisson_solver/Defs.cpp

      # Helpers
      volume_utils.h
      gradient/gvpm/gvpm_geoOps.h

      # Beam-beam implementation
      photonmapper/beams.cpp
      photonmapper/beams_accel.cpp
      photonmapper/beams_struct.h

      gradient/gvpm/gvpm_beams.h

      # Photon plane implementation
      photonmapper/plane_struct.h
      photonmapper/plane_accel.h
      photonmapper/plane_accel.cpp
      gradient/gvpm/gvpm_plane.h

      # Shifts
      gradient/gvpm/shift/shift_surface.cpp
      gradient/gvpm/shift/shift_volume_photon.cpp
      gradient/gvpm/shift/shift_volume_beams.cpp
      gradient/gvpm/shift/shift_volume_planes.h
      gradient/gvpm/shift/shift_utilities.h

      # Special shift
      gradient/gvpm/shift/operation/shift_ME.cpp
      gradient/gvpm/shift/operation/shift_diffuse.h
      gradient/gvpm/shift/operation/shift_diffuse.cpp
      gradient/gvpm/shift/operation/shift_medium.h

      # Denoiser 
      denoiser/nfor/Regression.cpp
      denoiser/nfor/nfor.cpp
      denoiser/nfor/core/Logging.cpp
      denoiser/nfor/core/thread/ThreadPool.cpp
      denoiser/nfor/core/thread/ThreadUtils.cpp
)
link_omp(gvpm)