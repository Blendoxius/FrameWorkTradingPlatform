cmake_minimum_required(VERSION 3.20)

project(TradingPlatform 
    VERSION 1.0.0
    DESCRIPTION "High-Performance Trading Platform"
    LANGUAGES CXX
)

# C++ Standard
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Output directories
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

# Build type
if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

# Compiler flags
if(MSVC)
    add_compile_options(/W4 /WX /permissive- /Zc:__cplusplus)
    add_compile_options($<$<CONFIG:Release>:/O2 /GL>)
    add_compile_options($<$<CONFIG:Debug>:/Od /Zi>)
else()
    add_compile_options(-Wall -Wextra -Wpedantic -Werror)
    add_compile_options($<$<CONFIG:Release>:-O3 -march=native>)
    add_compile_options($<$<CONFIG:Debug>:-g -O0>)
endif()

# Options
option(BUILD_TESTING "Build tests" ON)
option(BUILD_BENCHMARKS "Build benchmarks" ON)
option(ENABLE_BACKTESTING "Enable backtesting framework" ON)
option(USE_OPENGL "Use OpenGL rendering" ON)

# Find packages
find_package(Threads REQUIRED)
find_package(OpenSSL REQUIRED)
find_package(Boost REQUIRED COMPONENTS system thread filesystem)
find_package(SQLite3 REQUIRED)

if(USE_OPENGL)
    find_package(OpenGL REQUIRED)
    find_package(GLEW REQUIRED)
endif()

# Include vcpkg packages
find_package(RapidJSON CONFIG REQUIRED)
find_package(spdlog CONFIG REQUIRED)
find_package(fmt CONFIG REQUIRED)
find_package(websocketpp CONFIG REQUIRED)
find_package(cpprestsdk CONFIG REQUIRED)

# Include directories
include_directories(
    ${CMAKE_SOURCE_DIR}/src
    ${CMAKE_SOURCE_DIR}/external
    ${Boost_INCLUDE_DIRS}
    ${OPENSSL_INCLUDE_DIR}
)

# ==============================================================================
# CORE LIBRARY
# ==============================================================================
set(CORE_SOURCES
    # Engine
    src/Core/Engine/TradingEngine.cpp
    src/Core/Engine/OrderManager.cpp
    src/Core/Engine/PositionManager.cpp
    src/Core/Engine/RiskManager.cpp
    src/Core/Engine/ExecutionEngine.cpp
    
    # Data
    src/Core/Data/MarketDataManager.cpp
    src/Core/Data/TickProcessor.cpp
    src/Core/Data/HistoricalDataManager.cpp
    src/Core/Data/DataNormalizer.cpp
    
    # Memory
    src/Core/Memory/MemoryPool.cpp
    src/Core/Memory/RingBuffer.cpp
    src/Core/Memory/LockFreeQueue.cpp
    
    # Utils
    src/Core/Utils/Logger.cpp
    src/Core/Utils/Config.cpp
    src/Core/Utils/Timer.cpp
    src/Core/Utils/ThreadPool.cpp
)

add_library(TradingCore STATIC ${CORE_SOURCES})
target_link_libraries(TradingCore 
    PUBLIC
        Threads::Threads
        spdlog::spdlog
        fmt::fmt
)

# ==============================================================================
# DATA PROVIDERS LIBRARY
# ==============================================================================
set(DATA_PROVIDERS_SOURCES
    # Base
    src/DataProviders/Base/BaseDataProvider.cpp
    src/DataProviders/Base/WebSocketClient.cpp
    src/DataProviders/Base/RestClient.cpp
    src/DataProviders/Base/DataFeedSession.cpp
    
    # Binance
    src/DataProviders/Exchanges/Binance/BinanceDataProvider.cpp
    src/DataProviders/Exchanges/Binance/BinanceWebSocket.cpp
    src/DataProviders/Exchanges/Binance/BinanceRest.cpp
    
    # BitMEX
    src/DataProviders/Exchanges/BitMEX/BitMEXDataProvider.cpp
    src/DataProviders/Exchanges/BitMEX/BitMEXWebSocket.cpp
    
    # Bybit
    src/DataProviders/Exchanges/Bybit/BybitDataProvider.cpp
    src/DataProviders/Exchanges/Bybit/BybitWebSocket.cpp
    
    # OKX
    src/DataProviders/Exchanges/OKX/OKXDataProvider.cpp
    src/DataProviders/Exchanges/OKX/OKXWebSocket.cpp
    
    # Interactive Brokers
    src/DataProviders/Exchanges/InteractiveBrokers/IBDataProvider.cpp
    src/DataProviders/Exchanges/InteractiveBrokers/IBClient.cpp
    
    # MT4/MT5
    src/DataProviders/Exchanges/MT4MT5/MTDataProvider.cpp
    src/DataProviders/Exchanges/MT4MT5/MTClient.cpp
    
    # CQG
    src/DataProviders/Exchanges/CQG/CQGDataProvider.cpp
    src/DataProviders/Exchanges/CQG/CQGClient.cpp
    
    # Simulation
    src/DataProviders/Simulation/SimulationProvider.cpp
    src/DataProviders/Simulation/BacktestProvider.cpp
)

add_library(DataProviders STATIC ${DATA_PROVIDERS_SOURCES})
target_link_libraries(DataProviders
    PUBLIC
        TradingCore
        websocketpp::websocketpp
        cpprestsdk::cpprest
        OpenSSL::SSL
        OpenSSL::Crypto
        ${Boost_LIBRARIES}
)

# ==============================================================================
# AGGREGATION LIBRARY
# ==============================================================================
set(AGGREGATION_SOURCES
    src/Aggregation/TickAggregator.cpp
    src/Aggregation/CandleBuilder.cpp
    src/Aggregation/VolumeProfileBuilder.cpp
    src/Aggregation/OrderFlowBuilder.cpp
    src/Aggregation/TimeFrameManager.cpp
    src/Aggregation/CustomIntervalBuilder.cpp
)

add_library(Aggregation STATIC ${AGGREGATION_SOURCES})
target_link_libraries(Aggregation
    PUBLIC
        TradingCore
)

# ==============================================================================
# INDICATORS LIBRARY
# ==============================================================================
set(INDICATORS_SOURCES
    # Base
    src/Indicators/Base/BaseIndicator.cpp
    src/Indicators/Base/IndicatorManager.cpp
    src/Indicators/Base/IndicatorRegistry.cpp
    
    # Moving Averages
    src/Indicators/Technical/MovingAverages/SMA.cpp
    src/Indicators/Technical/MovingAverages/EMA.cpp
    src/Indicators/Technical/MovingAverages/WMA.cpp
    src/Indicators/Technical/MovingAverages/VWMA.cpp
    
    # Oscillators
    src/Indicators/Technical/Oscillators/RSI.cpp
    src/Indicators/Technical/Oscillators/MACD.cpp
    src/Indicators/Technical/Oscillators/Stochastic.cpp
    src/Indicators/Technical/Oscillators/Williams.cpp
    
    # Trend
    src/Indicators/Technical/Trend/BollingerBands.cpp
    src/Indicators/Technical/Trend/Ichimoku.cpp
    src/Indicators/Technical/Trend/ParabolicSAR.cpp
    
    # Volume
    src/Indicators/Technical/Volume/VolumeProfile.cpp
    src/Indicators/Technical/Volume/VWAP.cpp
    src/Indicators/Technical/Volume/OnBalanceVolume.cpp
    
    # Order Flow
    src/Indicators/OrderFlow/MarketProfile.cpp
    src/Indicators/OrderFlow/DeltaIndicator.cpp
    src/Indicators/OrderFlow/CVD.cpp
    src/Indicators/OrderFlow/FootprintChart.cpp
    
    # Custom
    src/Indicators/Custom/SessionHighLow.cpp
    src/Indicators/Custom/LiquidityLevels.cpp
    src/Indicators/Custom/SmartMoney.cpp
)

add_library(Indicators STATIC ${INDICATORS_SOURCES})
target_link_libraries(Indicators
    PUBLIC
        TradingCore
        Aggregation
)

# ==============================================================================
# STRATEGIES LIBRARY
# ==============================================================================
set(STRATEGIES_SOURCES
    # Base
    src/Strategies/Base/BaseStrategy.cpp
    src/Strategies/Base/StrategyManager.cpp
    src/Strategies/Base/StrategyEngine.cpp
    
    # Algorithms
    src/Strategies/Algorithms/ScalpingStrategy.cpp
    src/Strategies/Algorithms/BreakoutStrategy.cpp
    src/Strategies/Algorithms/MeanReversionStrategy.cpp
    src/Strategies/Algorithms/TrendFollowingStrategy.cpp
    
    # Order Flow
    src/Strategies/OrderFlow/VolumeProfileStrategy.cpp
    src/Strategies/OrderFlow/DeltaDivergenceStrategy.cpp
    src/Strategies/OrderFlow/FootprintStrategy.cpp
    
    # Portfolio
    src/Strategies/Portfolio/MultiInstrumentStrategy.cpp
    src/Strategies/Portfolio/HedgingStrategy.cpp
    src/Strategies/Portfolio/ArbitrageStrategy.cpp
)

add_library(Strategies STATIC ${STRATEGIES_SOURCES})
target_link_libraries(Strategies
    PUBLIC
        TradingCore
        Indicators
)

# ==============================================================================
# NETWORK LIBRARY
# ==============================================================================
set(NETWORK_SOURCES
    # Base
    src/Network/Base/NetworkManager.cpp
    src/Network/Base/ConnectionPool.cpp
    src/Network/Base/SSLManager.cpp
    src/Network/Base/ProxyManager.cpp
    
    # WebSocket
    src/Network/WebSocket/WebSocketClient.cpp
    src/Network/WebSocket/WebSocketManager.cpp
    src/Network/WebSocket/ReconnectionHandler.cpp
    
    # HTTP
    src/Network/HTTP/HTTPClient.cpp
    src/Network/HTTP/RESTClient.cpp
    src/Network/HTTP/RateLimiter.cpp
    
    # FIX
    src/Network/FIX/FIXClient.cpp
    src/Network/FIX/FIXParser.cpp
    src/Network/FIX/FIXSession.cpp
)

add_library(Network STATIC ${NETWORK_SOURCES})
target_link_libraries(Network
    PUBLIC
        TradingCore
        OpenSSL::SSL
        OpenSSL::Crypto
        websocketpp::websocketpp
        cpprestsdk::cpprest
        ${Boost_LIBRARIES}
)

# ==============================================================================
# DATABASE LIBRARY
# ==============================================================================
set(DATABASE_SOURCES
    # SQLite
    src/Database/SQLite/SQLiteManager.cpp
    src/Database/SQLite/HistoryDatabase.cpp
    src/Database/SQLite/SettingsDatabase.cpp
    
    # TimeSeries
    src/Database/TimeSeries/TSDBManager.cpp
    src/Database/TimeSeries/TickDatabase.cpp
    src/Database/TimeSeries/CandleDatabase.cpp
    
    # Cache
    src/Database/Cache/MemoryCache.cpp
    src/Database/Cache/DiskCache.cpp
    src/Database/Cache/CacheManager.cpp
)

add_library(Database STATIC ${DATABASE_SOURCES})
target_link_libraries(Database
    PUBLIC
        TradingCore
        SQLite::SQLite3
)

# ==============================================================================
# RENDERING LIBRARY
# ==============================================================================
if(USE_OPENGL)
    set(RENDERING_SOURCES
        # OpenGL
        src/Rendering/OpenGL/OpenGLRenderer.cpp
        src/Rendering/OpenGL/ShaderManager.cpp
        src/Rendering/OpenGL/TextureManager.cpp
        src/Rendering/OpenGL/BufferManager.cpp
        src/Rendering/OpenGL/GLContext.cpp
        
        # Primitives
        src/Rendering/Primitives/CandlestickRenderer.cpp
        src/Rendering/Primitives/LineRenderer.cpp
        src/Rendering/Primitives/BarRenderer.cpp
        src/Rendering/Primitives/VolumeRenderer.cpp
        src/Rendering/Primitives/HeatmapRenderer.cpp
        
        # Text
        src/Rendering/Text/FontManager.cpp
        src/Rendering/Text/TextRenderer.cpp
        src/Rendering/Text/LabelRenderer.cpp
        
        # Effects
        src/Rendering/Effects/AnimationManager.cpp
        src/Rendering/Effects/TransitionEffects.cpp
        src/Rendering/Effects/HighlightEffects.cpp
    )
    
    add_library(Rendering STATIC ${RENDERING_SOURCES})
    target_link_libraries(Rendering
        PUBLIC
            TradingCore
            OpenGL::GL
            GLEW::GLEW
    )
endif()

# ==============================================================================
# UI LIBRARY
# ==============================================================================
set(UI_SOURCES
    # Framework
    src/UI/Framework/MainWindow.cpp
    src/UI/Framework/DockingManager.cpp
    src/UI/Framework/ThemeManager.cpp
    src/UI/Framework/UIManager.cpp
    src/UI/Framework/LayoutManager.cpp
    
    # Charts
    src/UI/Charts/ChartWidget.cpp
    src/UI/Charts/ChartRenderer.cpp
    src/UI/Charts/ChartInteraction.cpp
    src/UI/Charts/CrosshairTool.cpp
    src/UI/Charts/DrawingTools.cpp
    src/UI/Charts/ZoomPanHandler.cpp
    src/UI/Charts/MultiTimeframe.cpp
    
    # Order Entry
    src/UI/Controls/OrderEntry/OrderEntryWidget.cpp
    src/UI/Controls/OrderEntry/QuickOrderButtons.cpp
    src/UI/Controls/OrderEntry/BracketOrderWidget.cpp
    
    # Market Depth
    src/UI/Controls/MarketDepth/DOMWidget.cpp
    src/UI/Controls/MarketDepth/Level2Display.cpp
    src/UI/Controls/MarketDepth/OrderBookWidget.cpp
    
    # Portfolio
    src/UI/Controls/Portfolio/PositionsWidget.cpp
    src/UI/Controls/Portfolio/OrdersWidget.cpp
    src/UI/Controls/Portfolio/AccountWidget.cpp
    
    # Watchlist
    src/UI/Controls/Watchlist/WatchlistWidget.cpp
    src/UI/Controls/Watchlist/QuoteBoard.cpp
    src/UI/Controls/Watchlist/SymbolSearch.cpp
    
    # Dialogs
    src/UI/Dialogs/SettingsDialog.cpp
    src/UI/Dialogs/IndicatorDialog.cpp
    src/UI/Dialogs/StrategyDialog.cpp
    src/UI/Dialogs/AboutDialog.cpp
    
    # Themes
    src/UI/Themes/DarkTheme.cpp
    src/UI/Themes/LightTheme.cpp
    src/UI/Themes/CustomTheme.cpp
)

add_library(UI STATIC ${UI_SOURCES})
target_link_libraries(UI
    PUBLIC
        TradingCore
        Rendering
)

# ==============================================================================
# ANALYTICS LIBRARY
# ==============================================================================
set(ANALYTICS_SOURCES
    # Performance
    src/Analytics/Performance/PerformanceAnalyzer.cpp
    src/Analytics/Performance/TradeAnalyzer.cpp
    src/Analytics/Performance/DrawdownAnalyzer.cpp
    
    # Statistics
    src/Analytics/Statistics/StatisticsEngine.cpp
    src/Analytics/Statistics/VolatilityCalculator.cpp
    src/Analytics/Statistics/CorrelationAnalyzer.cpp
    
    # Reporting
    src/Analytics/Reporting/ReportGenerator.cpp
    src/Analytics/Reporting/PDFExporter.cpp
    src/Analytics/Reporting/ExcelExporter.cpp
)

add_library(Analytics STATIC ${ANALYTICS_SOURCES})
target_link_libraries(Analytics
    PUBLIC
        TradingCore
)

# ==============================================================================
# MAIN APPLICATION
# ==============================================================================
add_executable(TradingPlatform
    src/main.cpp
    src/Application.cpp
)

target_link_libraries(TradingPlatform
    PRIVATE
        TradingCore
        DataProviders
        Aggregation
        Indicators
        Strategies
        Network
        Database
        UI
        Analytics
)

if(USE_OPENGL)
    target_link_libraries(TradingPlatform PRIVATE Rendering)
endif()

# ==============================================================================
# TESTING
# ==============================================================================
if(BUILD_TESTING)
    enable_testing()
    
    # Unit Tests
    add_executable(UnitTests
        src/Testing/UnitTests/CoreTests.cpp
        src/Testing/UnitTests/IndicatorTests.cpp
        src/Testing/UnitTests/StrategyTests.cpp
        src/Testing/UnitTests/DataProviderTests.cpp
    )
    
    target_link_libraries(UnitTests
        PRIVATE
            TradingCore
            Indicators
            Strategies
            DataProviders
    )
    
    add_test(NAME UnitTests COMMAND UnitTests)
    
    # Integration Tests
    add_executable(IntegrationTests
        src/Testing/IntegrationTests/EndToEndTests.cpp
        src/Testing/IntegrationTests/NetworkTests.cpp
    )
    
    target_link_libraries(IntegrationTests
        PRIVATE
            TradingCore
            Network
            DataProviders
    )
    
    add_test(NAME IntegrationTests COMMAND IntegrationTests)
endif()

# ==============================================================================
# BACKTESTING
# ==============================================================================
if(ENABLE_BACKTESTING)
    add_executable(BacktestEngine
        src/Testing/BacktestFramework/BacktestEngine.cpp
        src/Testing/BacktestFramework/HistoricalSimulator.cpp
        src/Testing/BacktestFramework/BacktestResults.cpp
    )
    
    target_link_libraries(BacktestEngine
        PRIVATE
            TradingCore
            DataProviders
            Strategies
            Analytics
    )
endif()

# ==============================================================================
# BENCHMARKS
# ==============================================================================
if(BUILD_BENCHMARKS)
    add_executable(Benchmarks
        src/Testing/Benchmarks/PerformanceBenchmark.cpp
        src/Testing/Benchmarks/LatencyBenchmark.cpp
    )
    
    target_link_libraries(Benchmarks
        PRIVATE
            TradingCore
            Indicators
    )
endif()

# ==============================================================================
# RESOURCES
# ==============================================================================
file(COPY ${CMAKE_SOURCE_DIR}/resources DESTINATION ${CMAKE_BINARY_DIR})
file(COPY ${CMAKE_SOURCE_DIR}/config DESTINATION ${CMAKE_BINARY_DIR})

# ==============================================================================
# INSTALLATION
# ==============================================================================
install(TARGETS TradingPlatform
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
)

install(DIRECTORY resources/ DESTINATION share/TradingPlatform/resources)
install(DIRECTORY config/ DESTINATION share/TradingPlatform/config)

# ==============================================================================
# PACKAGE
# ==============================================================================
set(CPACK_PACKAGE_NAME "TradingPlatform")
set(CPACK_PACKAGE_VERSION ${PROJECT_VERSION})
set(CPACK_PACKAGE_VENDOR "Your Company")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY ${PROJECT_DESCRIPTION})

include(CPack)

# ==============================================================================
# STATUS OUTPUT
# ==============================================================================
message(STATUS "")
message(STATUS "TradingPlatform Configuration:")
message(STATUS "  Version: ${PROJECT_VERSION}")
message(STATUS "  Build Type: ${CMAKE_BUILD_TYPE}")
message(STATUS "  C++ Standard: ${CMAKE_CXX_STANDARD}")
message(STATUS "  Testing: ${BUILD_TESTING}")
message(STATUS "  Benchmarks: ${BUILD_BENCHMARKS}")
message(STATUS "  Backtesting: ${ENABLE_BACKTESTING}")
message(STATUS "  OpenGL Rendering: ${USE_OPENGL}")
message(STATUS "")
