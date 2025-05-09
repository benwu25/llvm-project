lldb_tablegen(CoreProperties.inc -gen-lldb-property-defs
  SOURCE CoreProperties.td
  TARGET LLDBCorePropertiesGen)

lldb_tablegen(CorePropertiesEnum.inc -gen-lldb-property-enum-defs
  SOURCE CoreProperties.td
  TARGET LLDBCorePropertiesEnumGen)

set(LLDB_CURSES_LIBS)
set(LLDB_LIBEDIT_LIBS)

if (LLDB_ENABLE_CURSES)
  list(APPEND LLDB_CURSES_LIBS ${PANEL_LIBRARIES} ${CURSES_LIBRARIES})
  if (LLVM_BUILD_STATIC)
    list(APPEND LLDB_CURSES_LIBS gpm)
  endif()
endif()

if (LLVM_BUILD_TELEMETRY)
   set(TELEMETRY_SOURCES Telemetry.cpp)
   set(TELEMETRY_DEPS Telemetry)
endif()

# TODO: Add property `NO_PLUGIN_DEPENDENCIES` to lldbCore
add_lldb_library(lldbCore
  Address.cpp
  AddressRange.cpp
  AddressRangeListImpl.cpp
  AddressResolver.cpp
  AddressResolverFileLine.cpp
  Communication.cpp
  DataFileCache.cpp
  Debugger.cpp
  DebuggerEvents.cpp
  Declaration.cpp
  Disassembler.cpp
  DumpDataExtractor.cpp
  DumpRegisterValue.cpp
  DumpRegisterInfo.cpp
  DynamicLoader.cpp
  EmulateInstruction.cpp
  FileLineResolver.cpp
  FormatEntity.cpp
  Highlighter.cpp
  IOHandler.cpp
  IOHandlerCursesGUI.cpp
  Mangled.cpp
  Module.cpp
  ModuleChild.cpp
  ModuleList.cpp
  Opcode.cpp
  PluginManager.cpp
  Progress.cpp
  RichManglingContext.cpp
  SearchFilter.cpp
  Section.cpp
  SourceLocationSpec.cpp
  SourceManager.cpp
  StreamAsynchronousIO.cpp
  ThreadedCommunication.cpp
  UserSettingsController.cpp
  Value.cpp
  ${TELEMETRY_SOURCES}
  PARTIAL_SOURCES_INTENDED
  DEPENDS
    clang-tablegen-targets

  LINK_LIBS
    lldbBreakpoint
    lldbDataFormatters
    lldbExpression
    lldbHost
    lldbInterpreter
    lldbSymbol
    lldbTarget
    lldbUtility
    lldbValueObject
    lldbPluginCPlusPlusLanguage
    lldbPluginObjCLanguage
    ${LLDB_CURSES_LIBS}

  CLANG_LIBS
    clangDriver

  LINK_COMPONENTS
    Support
    Demangle
    TargetParser
    ${TELEMETRY_DEPS}
  )

add_dependencies(lldbCore
  LLDBCorePropertiesGen
  LLDBCorePropertiesEnumGen)

# Needed to properly resolve references in a debug build.
# TODO: Remove once we have better layering
set_target_properties(lldbCore PROPERTIES LINK_INTERFACE_MULTIPLICITY 5)

if (LLDB_ENABLE_CURSES)
  target_include_directories(lldbCore PRIVATE ${CURSES_INCLUDE_DIRS})
endif()
