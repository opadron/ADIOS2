#------------------------------------------------------------------------------#
# Distributed under the OSI-approved Apache License, Version 2.0.  See
# accompanying file Copyright.txt for details.
#------------------------------------------------------------------------------#
  
add_library(taustubs
  toolkit/profiling/taustubs/tautimer.cpp
  toolkit/profiling/taustubs/tautimer.hpp
  toolkit/profiling/taustubs/taustubs.h
)

target_include_directories(taustubs PRIVATE
      $<BUILD_INTERFACE:${PROJECT_BINARY_DIR}/source>
)

add_library(adios2
  common/ADIOSTypes.cpp
  
  core/Attribute.cpp 
  core/AttributeBase.cpp
  core/ADIOS.cpp
  core/Engine.cpp
  core/IO.cpp core/IO.tcc
  core/Operator.cpp
  core/Stream.cpp core/Stream.tcc
  core/Variable.cpp core/Variable.tcc
  core/VariableBase.cpp
  core/VariableCompound.cpp core/VariableCompound.tcc

#operator callback
  operator/callback/Signature1.cpp
  operator/callback/Signature2.cpp

#helper
  helper/adiosComm.h  helper/adiosComm.cpp
  helper/adiosCommDummy.h  helper/adiosCommDummy.cpp
  helper/adiosDynamicBinder.h  helper/adiosDynamicBinder.cpp
  helper/adiosMath.cpp
  helper/adiosMemory.cpp
  helper/adiosNetwork.cpp
  helper/adiosString.cpp helper/adiosString.tcc
  helper/adiosSystem.cpp
  helper/adiosType.cpp
  helper/adiosXML.cpp
  helper/adiosYAML.cpp

#engine derived classes  
  engine/bp3/BP3Reader.cpp engine/bp3/BP3Reader.tcc
  engine/bp3/BP3Writer.cpp engine/bp3/BP3Writer.tcc

  engine/bp4/BP4Reader.cpp engine/bp4/BP4Reader.tcc
  engine/bp4/BP4Writer.cpp engine/bp4/BP4Writer.tcc

  engine/skeleton/SkeletonReader.cpp engine/skeleton/SkeletonReader.tcc
  engine/skeleton/SkeletonWriter.cpp engine/skeleton/SkeletonWriter.tcc

  engine/inline/InlineReader.cpp engine/inline/InlineReader.tcc
  engine/inline/InlineWriter.cpp engine/inline/InlineWriter.tcc
  
  engine/null/NullEngine.cpp

  engine/nullcore/NullCoreWriter.cpp engine/nullcore/NullCoreWriter.tcc
#toolkit
  toolkit/format/buffer/Buffer.cpp
  toolkit/format/buffer/heap/BufferSTL.cpp

  toolkit/format/bp/BPBase.cpp toolkit/format/bp/BPBase.tcc
  toolkit/format/bp/BPSerializer.cpp toolkit/format/bp/BPSerializer.tcc

  toolkit/format/bp/bp3/BP3Base.cpp
  toolkit/format/bp/bp3/BP3Serializer.cpp toolkit/format/bp/bp3/BP3Serializer.tcc
  toolkit/format/bp/bp3/BP3Deserializer.cpp toolkit/format/bp/bp3/BP3Deserializer.tcc

  toolkit/format/bp/bp4/BP4Base.cpp
  toolkit/format/bp/bp4/BP4Serializer.cpp toolkit/format/bp/bp4/BP4Serializer.tcc
  toolkit/format/bp/bp4/BP4Deserializer.cpp toolkit/format/bp/bp4/BP4Deserializer.tcc
  
  toolkit/format/bp/bpOperation/BPOperation.cpp 
  toolkit/format/bp/bpOperation/BPOperation.tcc
  toolkit/format/bp/bpOperation/compress/BPZFP.cpp 
  toolkit/format/bp/bpOperation/compress/BPZFP.tcc
  toolkit/format/bp/bpOperation/compress/BPSZ.cpp
  toolkit/format/bp/bpOperation/compress/BPMGARD.cpp
  toolkit/format/bp/bpOperation/compress/BPPNG.cpp
  toolkit/format/bp/bpOperation/compress/BPBZIP2.cpp 
  toolkit/format/bp/bpOperation/compress/BPBZIP2.tcc
  toolkit/format/bp/bpOperation/compress/BPBlosc.cpp
  
  toolkit/profiling/iochrono/Timer.cpp
  toolkit/profiling/iochrono/IOChrono.cpp

  toolkit/query/Query.cpp
  toolkit/query/Worker.cpp
  toolkit/query/XmlUtil.cpp
  toolkit/query/XmlWorker.cpp
  toolkit/query/BlockIndex.cpp

  toolkit/transport/Transport.cpp
  toolkit/transport/file/FileStdio.cpp
  toolkit/transport/file/FileFStream.cpp
  toolkit/transport/null/NullTransport.cpp

  toolkit/transportman/TransportMan.cpp

  toolkit/aggregator/mpi/MPIAggregator.cpp
  toolkit/aggregator/mpi/MPIChain.cpp
)
target_include_directories(adios2
  PUBLIC
    $<BUILD_INTERFACE:${ADIOS2_SOURCE_DIR}/source>
    $<BUILD_INTERFACE:${ADIOS2_BINARY_DIR}/source>
    $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)
target_link_libraries(adios2 PRIVATE adios2sys_interface pugixml_private taustubs yaml-cpp)
target_link_libraries(adios2 PUBLIC ${CMAKE_THREAD_LIBS_INIT})
target_compile_features(adios2 PUBLIC ${ADIOS2_CXX11_FEATURES})

if(UNIX)
  target_sources(adios2 PRIVATE toolkit/transport/file/FilePOSIX.cpp)
endif()

if(ADIOS2_HAVE_SysVShMem)
  target_sources(adios2 PRIVATE toolkit/format/buffer/ipc/BufferSystemV.cpp)
  
  target_sources(adios2 PRIVATE toolkit/transport/shm/ShmSystemV.cpp)
endif()

if(ADIOS2_HAVE_ZeroMQ)
    target_sources(adios2 PRIVATE
        toolkit/zmq/zmqreqrep/ZmqReqRep.cpp
        toolkit/zmq/zmqpubsub/ZmqPubSub.cpp
        )
    target_link_libraries(adios2 PRIVATE ZeroMQ::ZMQ)
endif()

#if (ADIOS2_HAVE_JSON)
if (ADIOS2_HAVE_DataMan)
    target_sources(adios2 PRIVATE
	toolkit/query/JsonWorker.cpp			
   	)
    target_link_libraries(adios2 PRIVATE nlohmann_json)
endif()

if(ADIOS2_HAVE_DataMan OR ADIOS2_HAVE_Table)
    target_sources(adios2 PRIVATE
        toolkit/format/dataman/DataManSerializer.cpp
        toolkit/format/dataman/DataManSerializer.tcc
    )
    target_link_libraries(adios2 PRIVATE nlohmann_json)
endif()


if(ADIOS2_HAVE_DataMan)
    target_sources(adios2 PRIVATE
        engine/dataman/DataManCommon.cpp
        engine/dataman/DataManReader.cpp
        engine/dataman/DataManReader.tcc
        engine/dataman/DataManWriter.cpp
        engine/dataman/DataManWriter.tcc
        )
endif()

if(ADIOS2_HAVE_SSC)
    target_sources(adios2 PRIVATE
        engine/ssc/SscReader.cpp
        engine/ssc/SscReader.tcc
        engine/ssc/SscWriter.cpp
        engine/ssc/SscWriter.tcc
        engine/ssc/SscHelper.cpp
        )
    target_link_libraries(adios2 PRIVATE nlohmann_json)
endif()

if(ADIOS2_HAVE_Table)
    target_sources(adios2 PRIVATE
        engine/table/TableWriter.cpp
        engine/table/TableWriter.tcc
    )
endif()

if(ADIOS2_HAVE_SST)
  add_subdirectory(toolkit/sst)
  target_sources(adios2 PRIVATE
    engine/sst/SstReader.cpp
    engine/sst/SstWriter.cpp
    engine/sst/SstParamParser.cpp
  )
  target_link_libraries(adios2 PRIVATE sst)
endif()

if(ADIOS2_HAVE_DataSpaces)
  target_sources(adios2 PRIVATE
     toolkit/dataspaces/ds_writer.c
     engine/dataspaces/DataSpacesWriter.cpp engine/dataspaces/DataSpacesWriter.tcc 
     engine/dataspaces/DataSpacesReader.cpp engine/dataspaces/DataSpacesReader.tcc 
  )
  target_link_libraries(adios2 PRIVATE DataSpaces::DataSpaces)
endif()

if(ADIOS2_HAVE_Blosc)
  target_sources(adios2 PRIVATE operator/compress/CompressBlosc.cpp)
  target_link_libraries(adios2 PRIVATE Blosc::Blosc)
endif()

if(ADIOS2_HAVE_BZip2)
  target_sources(adios2 PRIVATE operator/compress/CompressBZIP2.cpp)
  target_link_libraries(adios2 PRIVATE BZip2::BZip2)
endif()

if(ADIOS2_HAVE_ZFP)
  target_sources(adios2 PRIVATE operator/compress/CompressZFP.cpp)
  target_link_libraries(adios2 PRIVATE zfp::zfp)
endif()

if(ADIOS2_HAVE_SZ)
  target_sources(adios2 PRIVATE operator/compress/CompressSZ.cpp)
  target_link_libraries(adios2 PRIVATE SZ::SZ)
endif()

if(ADIOS2_HAVE_MGARD)
  target_sources(adios2 PRIVATE operator/compress/CompressMGARD.cpp)
  target_link_libraries(adios2 PRIVATE MGARD::MGARD)
endif()

if(ADIOS2_HAVE_PNG)
  target_sources(adios2 PRIVATE operator/compress/CompressPNG.cpp)
  target_link_libraries(adios2 PRIVATE PNG::PNG)
endif()

if(ADIOS2_HAVE_MPI)
  target_sources(adios2 PRIVATE
    core/IOMPI.cpp
    helper/adiosCommMPI.h  helper/adiosCommMPI.cpp
    helper/adiosMpiHandshake.h  helper/adiosMpiHandshake.cpp
    engine/insitumpi/InSituMPIWriter.cpp engine/insitumpi/InSituMPIWriter.tcc
    engine/insitumpi/InSituMPIReader.cpp engine/insitumpi/InSituMPIReader.tcc
    engine/insitumpi/InSituMPIFunctions.cpp engine/insitumpi/InSituMPISchedules.cpp
  )
  target_link_libraries(adios2 PUBLIC MPI::MPI_C)
endif()

if(ADIOS2_HAVE_HDF5)
  target_sources(adios2 PRIVATE
    core/IOHDF5.cpp
    engine/hdf5/HDF5ReaderP.cpp
    engine/hdf5/HDF5WriterP.cpp
    toolkit/interop/hdf5/HDF5Common.cpp toolkit/interop/hdf5/HDF5Common.tcc
    )
  if(NOT HDF5_VERSION VERSION_LESS 1.11)
    target_sources(adios2 PRIVATE
      engine/mixer/HDFMixer.cpp
      engine/mixer/HDFMixer.tcc
      engine/mixer/HDFMixerWriter.cpp
      )
  endif()
  if(HDF5_IS_PARALLEL)
    set_property(SOURCE core/IOMPI.cpp APPEND PROPERTY COMPILE_DEFINITIONS ADIOS2_HAVE_HDF5_PARALLEL)
    target_sources(adios2 PRIVATE
      toolkit/interop/hdf5/HDF5CommonMPI.cpp
      )
  endif()

  target_link_libraries(adios2 PRIVATE ${HDF5_C_LIBRARIES})

  if(HDF5_C_INCLUDE_DIRS)
    target_include_directories(adios2 PRIVATE ${HDF5_C_INCLUDE_DIRS})
  else()
    target_include_directories(adios2 PRIVATE ${HDF5_INCLUDE_DIRS})
  endif()
endif()

# Set library version information
set_target_properties(adios2 PROPERTIES
  VERSION ${ADIOS2_LIBRARY_VERSION}
  SOVERSION ${ADIOS2_VERSION_MAJOR}
)

install(FILES common/ADIOSMacros.h common/ADIOSTypes.h common/ADIOSTypes.inl
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/adios2/common
)

# Library installation
install(TARGETS adios2 taustubs EXPORT adios2Exports
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)
