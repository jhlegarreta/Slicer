
#-----------------------------------------------------------------------------
set(MODULE_NAME PETStandardUptakeValueComputation)

#-----------------------------------------------------------------------------

#
# SlicerExecutionModel
#
find_package(SlicerExecutionModel REQUIRED)
include(${SlicerExecutionModel_USE_FILE})

#
# DCMTK
#
find_package(DCMTK REQUIRED)

#
# ITK
#
set(${PROJECT_NAME}_ITK_COMPONENTS
  ITKCommon
  ITKIODCMTK
  ITKIOGDCM
  ITKIOImageBase
  ${ModuleDescriptionParser_ITK_COMPONENTS}
  )
find_package(ITK 4.6 COMPONENTS ${${PROJECT_NAME}_ITK_COMPONENTS} REQUIRED)
if(ITK_VERSION VERSION_GREATER_EQUAL "5.3")
  foreach(factory_uc IN ITEMS "IMAGEIO" "MESHIO" "TRANSFORMIO")
    set(ITK_NO_${factory_uc}_FACTORY_REGISTER_MANAGER 1)
  endforeach()
else()
  set(ITK_NO_IO_FACTORY_REGISTER_MANAGER 1) # See Libs/ITKFactoryRegistration/CMakeLists.txt
endif()
include(${ITK_USE_FILE})

#-----------------------------------------------------------------------------
set(${MODULE_NAME}_TARGET_LIBRARIES
  vtkITK ModuleDescriptionParser
  MRMLCore SlicerBaseLogic SlicerBaseCLI
  ${ITK_LIBRARIES}
  ${VTK_LIBRARIES}
  )
foreach(lib IN LISTS DCMTK_LIBRARIES)
  if(TARGET "DCMTK::${lib}")
    list(APPEND ${MODULE_NAME}_TARGET_LIBRARIES DCMTK::${lib})
  else()
    list(APPEND ${MODULE_NAME}_TARGET_LIBRARIES ${lib})
  endif()
endforeach()

#-----------------------------------------------------------------------------
SEMMacroBuildCLI(
  NAME ${MODULE_NAME}
  LOGO_HEADER ${Slicer_SOURCE_DIR}/Resources/NAMICLogo.h
  # LOGO_HEADER ${Slicer_SOURCE_DIR}/Resources/CTSCLogo.h
  TARGET_LIBRARIES ${${MODULE_NAME}_TARGET_LIBRARIES}
  INCLUDE_DIRECTORIES
    ${vtkITK_INCLUDE_DIRS}
    ${MRMLCore_INCLUDE_DIRS}
    ${SlicerBaseLogic_SOURCE_DIR}  ${SlicerBaseLogic_BINARY_DIR}
  EXECUTABLE_ONLY
  )

#-----------------------------------------------------------------------------
if(BUILD_TESTING)
  if(NOT WIN32)
    add_subdirectory(Testing)
  endif()
endif()
