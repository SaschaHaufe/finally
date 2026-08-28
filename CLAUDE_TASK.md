# Market Data Backend Implementation Task

This PR is created to trigger Claude (Coding Agent) to build the complete Market Data backend as specified in issue #2.

## Task Requirements

Build the complete Market Data backend in `backend/app/market/` directory with:

1. **The unified market data interface** - Abstract base class and concrete implementations
2. **The market data simulator** - GBM-based price simulation with correlated moves
3. **Full unit tests** - Comprehensive test suite with 70+ tests

## Reference Documentation

Read all documents in the `planning/` directory:
- `planning/PLAN.md` - Full project specification (Section 6: Market Data)
- `planning/MARKET_DATA_SUMMARY.md` - Detailed architecture and implementation summary

## Expected Deliverables

### Core Modules in `backend/app/market/`
- `models.py` - `PriceUpdate` dataclass
- `interface.py` - `MarketDataSource` abstract base class
- `cache.py` - `PriceCache` thread-safe store
- `seed_prices.py` - Realistic seed prices and GBM parameters
- `simulator.py` - GBM simulator and `SimulatorDataSource`
- `massive_client.py` - Polygon.io REST client (`MassiveDataSource`)
- `factory.py` - Factory function for data source selection
- `stream.py` - FastAPI SSE endpoint factory

### Test Suite in `backend/tests/market/`
- `test_models.py` - 11 tests for data models
- `test_cache.py` - 13 tests for price cache
- `test_simulator.py` - 17 tests for GBM simulator
- `test_simulator_source.py` - 10 integration tests
- `test_factory.py` - 7 tests for factory pattern
- `test_massive.py` - 13 tests for Massive client

## Key Design Decisions

- **Strategy Pattern**: Both simulator and Massive implement same `MarketDataSource` ABC
- **PriceCache**: Single point of truth for all price data
- **GBM with Correlations**: Cholesky-decomposed correlation matrix per sector
- **SSE Streaming**: Version-based change detection for efficient client updates
- **Environment Variable**: Select data source via `MASSIVE_API_KEY`

## Testing Goals

- Overall coverage: 84%+
- All 73 tests passing
- Edge cases: unknown tickers, correlation groups, fallback seeds
- Integration: Simulator and Massive sources both work with same interface

## Success Criteria

✓ All 8 modules implemented with 100% of spec from MARKET_DATA_SUMMARY.md
✓ 73 unit tests, all passing
✓ Code review findings from the spec all resolved
✓ Demo script runs without errors: `cd backend && uv run market_data_demo.py`
