project (bftengine LANGUAGES CXX)

add_subdirectory(src/preprocessor)

set(corebft_source_files
    src/bftengine/PrimitiveTypes.cpp
    src/bftengine/DebugStatistics.cpp
    src/bftengine/Digest.cpp
    src/bftengine/SeqNumInfo.cpp
    src/bftengine/ReadOnlyReplica.cpp
    src/bftengine/ReplicaBase.cpp
    src/bftengine/ReplicaForStateTransfer.cpp
    src/bftengine/ReplicaImp.cpp
    src/bftengine/ControllerBase.cpp
    src/bftengine/ControllerWithSimpleHistory.cpp
    src/bftengine/IncomingMsgsStorageImp.cpp
    src/bftengine/RetransmissionsManager.cpp
    src/bftengine/SigManager.cpp
    src/bftengine/ReplicasInfo.cpp
    src/bftengine/ViewChangeSafetyLogic.cpp
    src/bftengine/ViewsManager.cpp
    src/bftengine/CheckpointInfo.cpp
    src/bftengine/ClientsManager.cpp
    src/bftengine/Crypto.cpp
    src/bftengine/NullStateTransfer.cpp
    src/bftengine/BFTEngine.cpp
    src/bftengine/SimpleClientImp.cpp
    src/bftengine/PersistentStorageImp.cpp
    src/bftengine/PersistentStorageDescriptors.cpp
    src/bftengine/PersistentStorageWindows.cpp
    src/bftengine/DebugPersistentStorage.cpp
    src/bftengine/ReplicaLoader.cpp
    src/bftengine/SerializableActiveWindow.cpp
    src/bftengine/MsgsCommunicator.cpp
    src/bftengine/MsgReceiver.cpp
    src/bftengine/DbMetadataStorage.cpp
    src/bftengine/RequestsBatchingLogic.cpp
    src/bftengine/ReplicaStatusHandlers.cpp
    src/bcstatetransfer/BCStateTran.cpp
    src/bcstatetransfer/InMemoryDataStore.cpp
    src/bcstatetransfer/STDigest.cpp
    src/bcstatetransfer/DBDataStore.cpp
    src/bcstatetransfer/SourceSelector.cpp
    src/simplestatetransfer/SimpleStateTran.cpp
    src/bftengine/messages/PrePrepareMsg.cpp
    src/bftengine/messages/CheckpointMsg.cpp
    src/bftengine/messages/FullCommitProofMsg.cpp
    src/bftengine/messages/MessageBase.cpp
    src/bftengine/messages/PartialCommitProofMsg.cpp
    src/bftengine/messages/PartialProofsSet.cpp
    src/bftengine/messages/ClientReplyMsg.cpp
    src/bftengine/messages/ReqMissingDataMsg.cpp
    src/bftengine/messages/ClientRequestMsg.cpp
    src/bftengine/messages/StartSlowCommitMsg.cpp
    src/bftengine/messages/SignedShareMsgs.cpp
    src/bftengine/messages/SimpleAckMsg.cpp
    src/bftengine/messages/NewViewMsg.cpp
    src/bftengine/messages/ViewChangeMsg.cpp
    src/bftengine/messages/ReplicaStatusMsg.cpp
    src/bftengine/messages/StateTransferMsg.cpp
    src/bftengine/messages/ReplicaAsksToLeaveViewMsg.cpp
    src/bftengine/KeyManager.cpp
    src/bftengine/RequestHandler.cpp
    src/bftengine/ControlStateManager.cpp
    src/bftengine/InternalBFTClient.cpp
    src/bftengine/KeyStore.cpp
)

#
# pthread dependency
find_package(Threads REQUIRED)
#message("Threads library: ${CMAKE_THREAD_LIBS_INIT}")


#
# Targets
#
add_library(corebft STATIC ${corebft_source_files})
add_library(bftclient STATIC src/bftengine/SimpleClientImp)
add_library(bftclient_shared SHARED src/bftengine/SimpleClientImp)
add_library(bftheaders INTERFACE)
target_include_directories(bftheaders INTERFACE include)

find_package(cryptopp REQUIRED)

get_property(perf_include GLOBAL PROPERTY PERF_MANAGER_INCLUDE_DIR)
get_property(kvbc_include GLOBAL PROPERTY KVBC_INCLUDE_DIR)

target_compile_options(corebft PUBLIC "-Wno-extra-semi" "-Wno-undefined-var-template") # TODO tmp cryptopp
target_include_directories(corebft PUBLIC include/)
target_include_directories(corebft PUBLIC include/bftengine)
target_include_directories(corebft PUBLIC include/bcstatetransfer)
target_include_directories(corebft PUBLIC include/simplestatetransfer)
target_include_directories(corebft PUBLIC include/metadatastorage)
target_include_directories(corebft PRIVATE src/bftengine)
target_include_directories(corebft PRIVATE src/preprocessor)
target_include_directories(corebft PUBLIC ${CRYPTOPP_INCLUDE_DIRS})
target_include_directories(corebft PUBLIC ${perf_include}/performance/include)
target_link_libraries(corebft PUBLIC
  threshsign
  Threads::Threads
  util
  concordbft_storage
  preprocessor
  bftcommunication
  diagnostics
  concordbft_reconfiguration
  secretsmanager)


target_include_directories(bftclient PUBLIC include/bftengine)
target_include_directories(bftclient PUBLIC src/bftengine)
target_include_directories(bftclient PUBLIC src/preprocessor)
target_include_directories(bftclient PUBLIC ${perf_include})
target_include_directories(bftclient PUBLIC ${kvbc_include})
target_link_libraries(bftclient PUBLIC bftcommunication)

target_include_directories(bftclient_shared PUBLIC include/bftengine)
target_include_directories(bftclient_shared PUBLIC src/bftengine)
target_include_directories(bftclient_shared PUBLIC src/preprocessor)
target_include_directories(bftclient_shared PUBLIC ${CRYPTOPP_INCLUDE_DIRS})
target_include_directories(bftclient_shared PUBLIC ${perf_include})
target_include_directories(bftclient_shared PUBLIC ${kvbc_include})

target_link_libraries(bftclient_shared PUBLIC bftcommunication_shared ${CRYPTOPP_SHARED_LIBRARY})
install (TARGETS bftclient_shared DESTINATION lib${LIB_SUFFIX})

install(DIRECTORY include/bftengine DESTINATION include)

if(BUILD_SLOWDOWN)
    target_compile_definitions(bftclient PUBLIC USE_SLOWDOWN)
    target_compile_definitions(corebft PUBLIC USE_SLOWDOWN)
    target_compile_definitions(bftclient_shared PUBLIC USE_SLOWDOWN)
endif()