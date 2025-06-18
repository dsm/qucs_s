# octave scripts

SET(BASICS
getQucsVariable.m	plotCartesian2D.m	plotFFT2V.m
loadQucsDataSet.m	plotCartesian2D2V.m	plotTimeFFT.m
loadQucsVariable.m	plotFFT.m		showQucsDataSet.m
)

INSTALL(FILES ${BASICS} DESTINATION ${DEPLOY_FILES_DIR}/${QUCS_NAME}/octave/)

