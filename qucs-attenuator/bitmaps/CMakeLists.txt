
SET(XPMS
)

# toolbar images
SET(PNGS
att_pi.png
att_tee.png
att_bridge.png
)

# application images
SET(ICONS
)

INSTALL(FILES ${XPMS} DESTINATION ${DEPLOY_FILES_DIR}/qucs/bitmaps)
INSTALL(FILES ${PNGS} DESTINATION ${DEPLOY_FILES_DIR}/qucs/bitmaps)
INSTALL(FILES ${ICONS} DESTINATION ${DEPLOY_FILES_DIR}/qucs/bitmaps)

