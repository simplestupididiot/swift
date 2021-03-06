
if(SWIFT_BUILD_ONLY_SYNTAXPARSERLIB)
  set(SWIFTAST_LLVM_LINK_COMPONENTS)
else()
  set(SWIFTAST_LLVM_LINK_COMPONENTS
    bitreader bitwriter coroutines coverage irreader debuginfoDWARF
    profiledata instrumentation object objcarcopts mc mcparser
    bitreader bitwriter lto ipo option core support ${LLVM_TARGETS_TO_BUILD}
  )
endif()

add_swift_host_library(swiftAST STATIC
  AbstractSourceFileDepGraphFactory.cpp
  AccessRequests.cpp
  ASTContext.cpp
  ASTDemangler.cpp
  ASTDumper.cpp
  ASTMangler.cpp
  ASTNode.cpp
  ASTPrinter.cpp
  ASTScope.cpp
  ASTScopeCreation.cpp
  ASTScopeLookup.cpp
  ASTScopePrinting.cpp
  ASTScopeSourceRange.cpp
  ASTVerifier.cpp
  ASTWalker.cpp
  Attr.cpp
  AutoDiff.cpp
  Availability.cpp
  AvailabilitySpec.cpp
  Builtins.cpp
  CaptureInfo.cpp
  ClangSwiftTypeCorrespondence.cpp
  ClangTypeConverter.cpp
  ConcreteDeclRef.cpp
  ConformanceLookupTable.cpp
  Decl.cpp
  DeclContext.cpp
  DeclNameLoc.cpp
  DiagnosticConsumer.cpp
  DiagnosticEngine.cpp
  DiagnosticList.cpp
  DocComment.cpp
  Evaluator.cpp
  Expr.cpp
  FineGrainedDependencies.cpp
  FrontendSourceFileDepGraphFactory.cpp
  GenericEnvironment.cpp
  GenericSignature.cpp
  GenericSignatureBuilder.cpp
  Identifier.cpp
  ImportCache.cpp
  IncrementalRanges.cpp
  IndexSubset.cpp
  InlinableText.cpp
  LayoutConstraint.cpp
  Module.cpp
  ModuleDependencies.cpp
  ModuleLoader.cpp
  ModuleNameLookup.cpp
  NameLookup.cpp
  NameLookupRequests.cpp
  Parameter.cpp
  Pattern.cpp
  PlatformKind.cpp
  PrettyStackTrace.cpp
  ProtocolConformance.cpp
  RawComment.cpp
  RequirementEnvironment.cpp
  SyntaxASTMap.cpp
  SILLayout.cpp
  Stmt.cpp
  SubstitutionMap.cpp
  SwiftNameTranslation.cpp
  Type.cpp
  TypeCheckRequests.cpp
  TypeDeclFinder.cpp
  TypeJoinMeet.cpp
  TypeRefinementContext.cpp
  TypeRepr.cpp
  TypeWalker.cpp
  UnqualifiedLookup.cpp
  USRGeneration.cpp

  LLVM_LINK_COMPONENTS
  ${SWIFTAST_LLVM_LINK_COMPONENTS}
  )

if(SWIFT_FORCE_OPTIMIZED_TYPECHECKER)
  if(CMAKE_CXX_COMPILER_ID STREQUAL MSVC OR CMAKE_CXX_SIMULATE_ID STREQUAL MSVC)
    target_compile_options(swiftAST PRIVATE /O2 /Ob2)
  else()
    target_compile_options(swiftAST PRIVATE -O3)
  endif()
endif()
if(NOT SWIFT_BUILD_ONLY_SYNTAXPARSERLIB)
  target_link_libraries(swiftAST INTERFACE
    clangCodeGen
    clangIndex
    clangFormat
    clangToolingCore
    clangFrontendTool
    clangFrontend
    clangDriver
    clangSerialization
    clangParse
    clangSema
    clangAnalysis
    clangEdit
    clangRewriteFrontend
    clangRewrite
    clangAST
    clangLex
    clangAPINotes
    clangBasic)
endif()

target_link_libraries(swiftAST
  PUBLIC swiftBasic
  PRIVATE swiftSyntax)
if(SWIFT_BUILD_ONLY_SYNTAXPARSERLIB)
  # Add clangBasic as a single direct dependency to avoid bringing along some
  # llvm libraries that we don't need.
  if("${SWIFT_HOST_VARIANT_SDK}" STREQUAL "WINDOWS")
    target_link_libraries(swiftAST PRIVATE
      "${LLVM_LIBRARY_OUTPUT_INTDIR}/clangBasic.lib")
  else()
    target_link_libraries(swiftAST PRIVATE
      "${LLVM_LIBRARY_OUTPUT_INTDIR}/libclangBasic.a")
  endif()
  target_compile_definitions(swiftAST PRIVATE
    SWIFT_BUILD_ONLY_SYNTAXPARSERLIB=1)
else()
  target_link_libraries(swiftAST PRIVATE
    swiftMarkup)
endif()

# intrinsics_gen is the LLVM tablegen target that generates the include files
# where intrinsics and attributes are declared. swiftAST depends on these
# headers.
# For more information see the comment at the top of lib/CMakeLists.txt.
add_dependencies(swiftAST intrinsics_gen clang-tablegen-targets)

set(swift_ast_verifier_flag)
if(SWIFT_AST_VERIFIER)
  set(swift_ast_verifier_flag " -USWIFT_DISABLE_AST_VERIFIER")
else()
  set(swift_ast_verifier_flag " -DSWIFT_DISABLE_AST_VERIFIER=1")
endif()

set_property(SOURCE ASTVerifier.cpp APPEND_STRING PROPERTY COMPILE_FLAGS
  "${swift_ast_verifier_flag}")

