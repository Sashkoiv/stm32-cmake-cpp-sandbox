cmake_minimum_required(VERSION 3.13.4)

project(stm32f303)

enable_language(C ASM)
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

SET(CPU         "cortex-m4")
set(FPU         "fpv4-sp-d16")
set(FLOAT_ABI   "hard")

set(TOOLCHAIN_PATH  "/opt/gcc-arm-none-eabi-10-2020-q4-major/bin")
set(CMAKE_ASM_COMPILER  ${TOOLCHAIN_PATH}/arm-none-eabi-gcc)
set(CMAKE_C_COMPILER ${TOOLCHAIN_PATH}/arm-none-eabi-gcc)
set(CMAKE_CXX_COMPILER ${TOOLCHAIN_PATH}/arm-none-eabi-g++)


set(STM32CUBEMX_GENERATED_FILES
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_ll_usb.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_i2c.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_i2c_ex.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_rcc.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_rcc_ex.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_gpio.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_dma.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_cortex.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_pwr.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_pwr_ex.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_flash.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_flash_ex.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_exti.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_spi.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_spi_ex.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_tim.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_tim_ex.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_pcd.c
        Drivers/STM32F3xx_HAL_Driver/Src/stm32f3xx_hal_pcd_ex.c

        Core/Src/main.c
        Core/Src/gpio.c
        Core/Src/i2c.c
        Core/Src/spi.c
        Core/Src/usb.c
        Core/Src/stm32f3xx_it.c
        Core/Src/stm32f3xx_hal_msp.c
        Core/Src/system_stm32f3xx.c

        startup_stm32f303xc.s)

set(EXECUTABLE ${PROJECT_NAME}.out)

add_executable(${EXECUTABLE} ${STM32CUBEMX_GENERATED_FILES})

target_compile_definitions(${EXECUTABLE} PRIVATE
        -DUSE_HAL_DRIVER
        -DSTM32F303xC
        )

target_include_directories(${EXECUTABLE} PRIVATE
        Core/Inc
        Drivers/STM32F3xx_HAL_Driver/Inc
        Drivers/CMSIS/Device/ST/STM32F3xx/Include
        Drivers/CMSIS/Include
        )

target_compile_options(${EXECUTABLE} PRIVATE
        -mcpu=${CPU}
        -mthumb
        -mfpu=${FPU}
        -mfloat-abi=${FLOAT_ABI}

        -fdata-sections
        -ffunction-sections

        -Wall

        $<$<CONFIG:Debug>:-Og>
        )

target_link_options(${EXECUTABLE} PRIVATE
        -T${CMAKE_SOURCE_DIR}/STM32F303VCTx_FLASH.ld
        -mcpu=${CPU}
        -mthumb
        -mfpu=${FPU}
        -mfloat-abi=${FLOAT_ABI}
        -specs=nano.specs
        -lc
        -lm
        -lnosys
        -Wl,-Map=${PROJECT_NAME}.map,--cref
        -Wl,--gc-sections
        )

# Print executable size
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND arm-none-eabi-size ${EXECUTABLE})

# Create hex file
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND arm-none-eabi-objcopy -O ihex ${EXECUTABLE} ${PROJECT_NAME}.hex
        COMMAND arm-none-eabi-objcopy -O binary ${EXECUTABLE} ${PROJECT_NAME}.bin)
