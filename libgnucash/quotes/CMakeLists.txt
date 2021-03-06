


SET(PERL ${PERL_EXECUTABLE})

SET(_TMPDIR ${CMAKE_CURRENT_BINARY_DIR}${CMAKE_FILES_DIRECTORY})
FILE(WRITE ${_TMPDIR}/copy_with_perms.cmake
  "FILE(COPY \${SRC} DESTINATION \${DST}
   FILE_PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE GROUP_READ GROUP_EXECUTE WORLD_READ WORLD_EXECUTE)"
)

SET(_BIN_FILES "")
FOREACH(file gnc-fq-check.in gnc-fq-helper.in gnc-fq-update.in gnc-fq-dump)
  STRING(REPLACE ".in" "" _OUTPUT_FILE_NAME ${file})
  GNC_CONFIGURE2(${file}-target ${file} ${_OUTPUT_FILE_NAME})
  SET(_ABS_OUTPUT_FILE ${CMAKE_CURRENT_BINARY_DIR}/${_OUTPUT_FILE_NAME})
  LIST(APPEND _BIN_FILES ${_ABS_OUTPUT_FILE})
  ADD_CUSTOM_COMMAND(
    OUTPUT ${_ABS_OUTPUT_FILE}
    APPEND
    COMMAND ${CMAKE_COMMAND} -D SRC=${_ABS_OUTPUT_FILE}
                             -D DST=${BINDIR_BUILD}
                             -P ${_TMPDIR}/copy_with_perms.cmake
  )
ENDFOREACH(file)

SET(CMAKE_COMMAND_TMP "")
IF (${CMAKE_VERSION} VERSION_GREATER 3.1)
  SET(CMAKE_COMMAND_TMP ${CMAKE_COMMAND} -E env)
ENDIF()


SET(_MAN_FILES "")
FOREACH(file gnc-fq-dump gnc-fq-helper)
  SET(_POD_INPUT ${CMAKE_CURRENT_BINARY_DIR}/${file})
  SET(_MAN_OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/${file}.1)
  LIST(APPEND _MAN_FILES ${_MAN_OUTPUT})
  ADD_CUSTOM_COMMAND(
      OUTPUT ${_MAN_OUTPUT}
      COMMAND
        ${CMAKE_COMMAND_TMP} ${PERL_EXECUTABLE} ${POD2MAN_EXECUTABLE} ${_POD_INPUT} ${_MAN_OUTPUT}
      DEPENDS ${_POD_INPUT}
  )
ENDFOREACH(file)


ADD_CUSTOM_TARGET(quotes-man ALL DEPENDS ${_MAN_FILES})
ADD_CUSTOM_TARGET(quotes-bin ALL DEPENDS gnc-fq-check gnc-fq-update)
INSTALL(FILES ${_MAN_FILES} DESTINATION share/man/man1)
INSTALL(PROGRAMS ${_BIN_FILES} DESTINATION bin)

SET_DIST_LIST(quotes_DIST CMakeLists.txt gnc-fq-check.in gnc-fq-dump gnc-fq-helper.in gnc-fq-update.in
        gnc-value-portfolio Makefile.am Quote_example.pl README)
