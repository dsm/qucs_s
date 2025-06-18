# python scripts

SET(BASICS
  parse_result.py
  parse_result_example.py
  rc_ac_sweep.dat
  rc_ac_sweep.net
)

INSTALL(FILES ${BASICS} DESTINATION ${DEPLOY_FILES_DIR}/${QUCS_NAME}/python/)

