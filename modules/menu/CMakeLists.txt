project(menu)

set(SRCS menu.c static_menu.c dynamic_menu.c)

if(STATIC)
    add_library(${PROJECT_NAME} STATIC ${SRCS})
    target_link_libraries(${PROJECT_NAME} PRIVATE baresip)
else()
    add_library(${PROJECT_NAME} MODULE ${SRCS})
endif()

set_target_properties(${PROJECT_NAME} PROPERTIES PREFIX "")

list(APPEND MODULES ${PROJECT_NAME})
set(MODULES ${MODULES} PARENT_SCOPE)
