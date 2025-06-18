
SET(RULES_MK
cmlib.linux.rules.mk
cmlib.mingw32.rules.mk
)

# installation of transmission lines
INSTALL( FILES ${RULES_MK} DESTINATION ${DEPLOY_FILES_DIR}/${QUCS_NAME}/xspice_cmlib )
INSTALL(DIRECTORY include DESTINATION ${DEPLOY_FILES_DIR}/${QUCS_NAME}/xspice_cmlib )

